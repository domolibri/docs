# Plano de Conclusão — Slice 1: Infra, Identidade e Onboarding

Este documento detalha as pendências técnicas e funcionais necessárias para atingir 100% de conclusão do **Slice 1**, conforme a [Documentação de Arquitetura Consolidada](../Documentacao-de-Arquitetura-SaaS-Editorial-Consolidada.md) e o [Backlog Inicial](../Backlog-Inicial-Implementacao-SaaS-Editorial.md).

## 1. Status Atual
- **Infraestrutura:** 100% (Docker, .NET 8, Angular, Postgres, MinIO, Mailpit).
- **Multi-tenancy:** 100% (Isolamento via `EditoraId`, Filtros Globais EF Core).
- **Onboarding/Auth:** 90% (Registro, Login, Branding, E-mail funcional).
- **Modelo de Identidade:** 50% (Usuário implementado, mas Roles/Permissões são simplificadas).
- **Auditoria e LGPD:** 10% (Estrutura de e-mail e segurança pronta, mas falta Log e Consentimento).

---

## 2. Pendências Críticas (P0)

### 2.1. Refatoração do Modelo de Identidade (Épico E02/E03)
Atualmente, as Roles são um `enum`. Para conformidade com a arquitetura, devem ser entidades vinculadas ao tenant.
- [ ] **Backend:** Criar entidades `Role` e `Permission`.
    - `Role`: Id, EditoraId, Nome, Descricao.
    - `Permission`: Id, Codigo (ex: `usuarios.ler`), Nome, Agrupamento.
    - Tabela de junção `RolePermission`.
- [ ] **Backend:** Atualizar `UsuarioEditora` para suportar múltiplas roles (N:N) ou garantir que a Role vinculada venha do banco.
- [ ] **Backend:** Implementar Seed de Roles padrão (`AdminEditora`, `GestorEditorial`, `Autor`) durante o `Register`.
- [ ] **Backend:** Atualizar as Claims do JWT para incluir a lista de permissões da Role do usuário.

### 2.2. Sistema de Auditoria Administrativa (Épico E12)
Requisito essencial para LGPD e rastreabilidade.
- [ ] **Backend:** Criar entidade `AuditLog`.
    - Campos: Id, EditoraId, UsuarioId, Acao, Recurso, RecursoId, IP, UserAgent, DataHora, DadosOriginais (JSON anonimizado), DadosNovos (JSON anonimizado).
- [ ] **Backend:** Implementar `AuditInterceptor` no DbContext para capturar mudanças em `Editora` e `UsuarioEditora` automaticamente.
- [ ] **Backend:** Registrar logs manuais para ações de Login, Logout e Falhas de Autenticação.

### 2.3. Base de Consentimento LGPD (Épico E12)
- [ ] **Backend:** Criar entidade `ConsentimentoLGPD`.
    - Campos: Id, EditoraId, UsuarioId, TipoConsentimento, VersaoTermo, DataConsentimento.
- [ ] **Frontend:** Adicionar checkbox de "Aceito os Termos de Uso e Política de Privacidade" no formulário de registro.
- [ ] **Backend:** Gravar o consentimento inicial durante o fluxo de `Register`.

---

## 3. Melhorias e Polimento (P1)

### 3.1. Dashboard e Branding
- [ ] **Frontend:** Refinar a aplicação da Cor Primária no CSS Global (usar variáveis de runtime baseadas no branding da editora).
- [ ] **Frontend:** Exibir as informações do usuário logado (Nome, Role, Editora) no AppShell de forma consistente.

### 3.2. Fluxo de Convite (Transição para Slice 2)
Embora o Slice 2 foque em usuários, a base do convite deve ser preparada agora.
- [ ] **Backend:** Criar entidade `ConviteUsuario` para suportar o fluxo de "Adicionar Colega" que surge logo após o onboarding.

---

## 4. Cronograma de Execução Estimado

| Atividade | Responsabilidade | Esforço Est. |
|---|---|---|
| Refatoração Identity (Roles/Permissions) | Backend | 2 dias |
| Implementação AuditLog + Interceptor | Backend | 2 dias |
| Fluxo de Consentimento LGPD | Fullstack | 1 dia |
| Polimento AppShell e Branding CSS | Frontend | 1 dia |
| **Total Estimado** | | **6 dias úteis** |

---

## 5. Critérios de Aceite para Finalização do Slice 1
1. [ ] O banco de dados possui as tabelas `Roles`, `Permissions` e `AuditLogs`.
2. [ ] Ao registrar uma nova editora, as roles padrão são criadas e o log de auditoria registra a criação.
3. [ ] O JWT emitido contém as claims de permissão baseadas na role do usuário.
4. [ ] O Frontend impede o acesso ao Dashboard se o usuário não tiver a role `AdminEditora` (ou equivalente).
5. [ ] O arquivo `changelog.md` está atualizado com as entregas do Slice 1.

---

## 6. Guia de Implementação (Prompts para Copilot)

Para agilizar o desenvolvimento do item **2.1 (Refatoração do Modelo de Identidade)**, utilize os prompts abaixo em sequência:

### Prompt 1: Criação das Entidades de Identidade (Domain)
> **Contexto:** Estou refatorando o modelo de identidade do projeto DomoLibri para suportar Roles e Permissions dinâmicas por Editora (tenant). Atualmente, `UsuarioEditora` usa um `enum Role`.
>
> **Tarefa:**
> 1. No projeto `DomoLibri.Domain/Entities`, crie as seguintes entidades:
>    - `Permission`: Propriedades `Id`, `Codigo` (ex: "usuarios.ler"), `Nome`, `Agrupamento` (ex: "Configurações").
>    - `Role`: Propriedades `Id`, `EditoraId`, `Nome`, `Descricao`. Deve possuir uma coleção de `Permission` e uma coleção de `UsuarioEditora`.
> 2. Atualize a entidade `UsuarioEditora`:
>    - Remova a propriedade `enum Role`.
>    - Adicione uma coleção `ICollection<Role> Roles`.
> 3. No `DomoLibriDbContext.cs`:
>    - Adicione os `DbSet` para `Role` e `Permission`.
>    - Configure o relacionamento N:N entre `Role` e `Permission` (tabela de junção `RolePermissions`).
>    - Configure o relacionamento N:N entre `UsuarioEditora` e `Role` (tabela de junção `UsuarioRoles`).
>    - Aplique o Filtro Global de Multi-tenancy (`EditoraId`) na entidade `Role`.
>
> **Padrão:** Siga o estilo de codificação existente, usando `Guid` para IDs e mantendo o isolamento de dados.

### Prompt 2: Refatoração do Fluxo de Registro e Seed de Roles (Application/Infrastructure)
> **Contexto:** As Roles agora são entidades vinculadas à Editora. Preciso que o processo de registro crie as roles padrão automaticamente.
>
> **Tarefa:**
> 1. No `AuthService.cs`, método `RegisterAsync`, após criar a `Editora`, implemente o seed das seguintes Roles padrão:
>    - `AdminEditora`: Com todas as permissões do sistema.
>    - `GestorEditorial`: Permissões de edição e visualização.
>    - `Autor`: Permissões limitadas a submissão de conteúdo.
> 2. Vincule o `adminUser` criado no registro à Role `AdminEditora` recém-criada.
> 3. Crie um método privado ou classe auxiliar para definir a lista mestra de permissões do sistema (ex: `usuarios.ler`, `usuarios.escrever`, `branding.editar`, etc.) para que elas possam ser vinculadas às roles durante o seed.
>
> **Regra de Negócio:** As permissões são globais (fixas no código), mas as Roles e suas associações são por Editora.

### Prompt 3: Atualização de Claims do JWT e Autorização (Infrastructure/API)
> **Contexto:** O JWT atualmente contém apenas a Role básica. Agora preciso incluir a lista consolidada de permissões baseada em todas as Roles do usuário.
>
> **Tarefa:**
> 1. No `AuthService.cs`, método `GenerateJwt`:
>    - Carregue as `Roles` do usuário incluindo suas `Permissions`.
>    - Adicione claims do tipo `permission` para cada código de permissão único que o usuário possuir.
>    - Mantenha a claim de `role`, mas agora pegando o nome da Role do banco de dados.
> 2. (Opcional) Crie um `AuthorizationHandler` ou uma política no `Program.cs` que valide o claim `permission` para facilitar o uso de `[Authorize(Policy = "usuarios.ler")]` nos controllers.
>
> **Segurança:** Garanta que as claims não tornem o token excessivamente grande. Use códigos curtos para as permissões.

### Prompt 4: Ajustes de Migração e Front-end (Infra/Frontend)
> **Contexto:** Alteramos drasticamente o esquema de banco de dados para Roles.
>
> **Tarefa:**
> 1. Gere uma nova migração do EF Core chamada `AddIdentityRolesAndPermissions`.
> 2. No Frontend (`frontend/src/app/shared/auth.service.ts`), atualize o método que decodifica o token para extrair a nova lista de permissões e disponibilizá-la via um `Observable` ou getter.
> 3. Verifique se o `tenant-interceptor.ts` continua funcionando corretamente com o novo modelo.

### 2.2. Sistema de Auditoria Administrativa (Épico E12)

Utilize os prompts abaixo para implementar a rastreabilidade e auditoria:

### Prompt 1: Criação da Entidade de Auditoria (Domain)
> **Contexto:** Estou implementando o Sistema de Auditoria Administrativa do projeto DomoLibri para rastrear mudanças críticas no banco de dados e eventos de segurança.
>
> **Tarefa:**
> 1. No projeto `DomoLibri.Domain/Entities`, crie a entidade `AuditLog`:
>    - Propriedades: `Id` (Guid), `EditoraId` (Guid?), `UsuarioId` (Guid?), `Acao` (string - ex: "Insert", "Update", "Delete", "Login"), `Recurso` (string - ex: "UsuarioEditora"), `RecursoId` (string), `IP` (string), `UserAgent` (string), `DataHora` (DateTime), `DadosOriginais` (string/JSON), `DadosNovos` (string/JSON).
> 2. No `DomoLibriDbContext.cs`:
>    - Adicione o `DbSet<AuditLog> AuditLogs`.
>    - Configure a entidade: Chave primária, indexação por `EditoraId` e `DataHora`.
>    - Aplique o Filtro Global de Multi-tenancy (`EditoraId`) na entidade `AuditLog`.
>
> **Padrão:** Use `System.Text.Json` para serialização de dados se necessário. Garanta que o campo `DadosOriginais` e `DadosNovos` possam armazenar strings longas (anonimizadas conforme LGPD).

### Prompt 2: Implementação do AuditInterceptor (Infrastructure)
> **Contexto:** Preciso que todas as mudanças de estado nas entidades `Editora` e `UsuarioEditora` sejam registradas automaticamente no `AuditLog` ao salvar alterações no banco de dados.
>
> **Tarefa:**
> 1. No projeto `DomoLibri.Infrastructure/Data`, crie uma classe `AuditInterceptor` que herda de `SaveChangesInterceptor`.
> 2. Sobrescreva o método `SavingChangesAsync`:
>    - Identifique entidades que foram Adicionadas, Modificadas ou Excluídas.
>    - Se a entidade for do tipo `Editora` ou `UsuarioEditora`, capture os valores antigos e novos.
>    - Resolva o `EditoraId` e `UsuarioId` atuais usando o `ITenantProvider` (ou extraia das claims do contexto se disponível).
>    - Crie instâncias de `AuditLog` para cada mudança e adicione-as ao `DbContext` antes de finalizar a operação.
> 3. No `Program.cs` (ou onde o DbContext é configurado), registre o `AuditInterceptor` no pipeline do EF Core.
>
> **Observação:** Garanta que dados sensíveis (como `SenhaHash`) nunca sejam incluídos nos logs de auditoria.

### Prompt 3: Registro de Logs Manuais de Autenticação (Infrastructure/Services)
> **Contexto:** Eventos de autenticação precisam de logs específicos para rastrear tentativas de invasão e auditoria de acesso.
>
> **Tarefa:**
> 1. No `AuthService.cs`:
>    - Injete o `DomoLibriDbContext` (se já não estiver) ou crie um método privado para registrar logs manuais.
>    - No método `LoginAsync`: Registre um `AuditLog` com ação "LoginSucesso" em caso de êxito, e "LoginFalha" em caso de erro de credenciais ou bloqueio.
>    - No método `ResetPasswordAsync`: Registre a ação "RedefinicaoSenha".
> 2. Garanta que o log capture o IP e o UserAgent do usuário (pode ser necessário injetar `IHttpContextAccessor`).
>
> **Regra:** O log deve ser vinculado ao `EditoraId` do usuário que está tentando o acesso.

### Prompt 4: Migração e Validação (Infra)
> **Contexto:** Finalizamos a implementação do sistema de auditoria.
>
> **Tarefa:**
> 1. Gere uma nova migração do EF Core chamada `AddAuditLogging`.
> 2. Verifique se o `AuditInterceptor` está lidando corretamente com transações para evitar que uma falha no log impeça o salvamento dos dados principais (ou vice-versa, dependendo da criticidade).
> 3. Certifique-se de que a serialização JSON para `DadosOriginais` e `DadosNovos` ignore propriedades marcadas como sensíveis.

### 2.3. Base de Consentimento LGPD (Épico E12)

Utilize os prompts abaixo para implementar o fluxo de consentimento:

### Prompt 1: Criação da Entidade de Consentimento LGPD (Domain)
> **Contexto:** Estou implementando a base de consentimento LGPD no projeto DomoLibri para garantir conformidade legal durante o registro de usuários.
>
> **Tarefa:**
> 1. No projeto `DomoLibri.Domain/Entities`, crie a entidade `ConsentimentoLGPD`:
>    - Propriedades: `Id` (Guid), `EditoraId` (Guid), `UsuarioId` (Guid), `TipoConsentimento` (string - ex: "TermosDeUso", "ComunicacaoMarketing"), `VersaoTermo` (string), `DataConsentimento` (DateTime).
> 2. No `DomoLibriDbContext.cs`:
>    - Adicione o `DbSet<ConsentimentoLGPD> ConsentimentosLGPD`.
>    - Configure a entidade: Chave primária, indexação por `UsuarioId` e `EditoraId`.
>    - Aplique o Filtro Global de Multi-tenancy (`EditoraId`) na entidade `ConsentimentoLGPD`.
>
> **Padrão:** Mantenha a consistência com as outras entidades do domínio e use `DateTime.UtcNow` para a data de consentimento.

### Prompt 2: Refatoração do Fluxo de Registro para Capturar Consentimento (Application/DTOs)
> **Contexto:** O formulário de registro agora exige que o usuário aceite os Termos de Uso. Preciso atualizar o DTO e o serviço de autenticação.
>
> **Tarefa:**
> 1. No `DomoLibri.Application/DTOs`, atualize o `RegisterDto` (ou equivalente) para incluir uma propriedade booleana `AceitouTermos`.
> 2. No `AuthService.cs`, método `RegisterAsync`:
>    - Valide se `AceitouTermos` é verdadeiro. Caso contrário, retorne um erro de validação.
>    - Após a criação bem-sucedida do `UsuarioEditora`, crie um registro na tabela `ConsentimentoLGPD` vinculando o `UsuarioId`, `EditoraId`, o tipo "TermosDeUso" e a versão atual dos termos (ex: "1.0").
> 3. Garanta que a operação de criação do usuário e do consentimento ocorra dentro de uma transação (se aplicável) ou de forma atômica.
>
> **Regra:** O registro não deve prosseguir sem o consentimento explícito dos termos de uso.

### Prompt 3: Interface de Consentimento no Frontend (Frontend)
> **Contexto:** Preciso adicionar a interface de consentimento LGPD no formulário de registro do Angular.
>
> **Tarefa:**
> 1. No `frontend/src/app/onboarding/register/register.html`:
>    - Adicione um checkbox obrigatório com o rótulo "Aceito os Termos de Uso e Política de Privacidade".
>    - Adicione links (placeholders) para os documentos de "Termos de Uso" e "Política de Privacidade".
> 2. No `register.ts`:
>    - Adicione o controle `aceitouTermos` ao `FormGroup` de registro com validação `Validators.requiredTrue`.
>    - Atualize a chamada ao `onboardingService.register` para enviar o novo campo.
> 3. No `onboarding.service.ts`:
>    - Atualize a interface/modelo de dados de registro para incluir o campo `aceitouTermos`.
>
> **UX:** O botão de "Registrar" deve permanecer desabilitado ou exibir erro se o checkbox não estiver marcado.

### Prompt 4: Migração e Finalização (Infra)
> **Contexto:** Finalizamos a implementação do fluxo de consentimento.
>
> **Tarefa:**
> 1. Gere uma nova migração do EF Core chamada `AddLGPDConsent`.
> 2. Verifique se o banco de dados reflete corretamente o relacionamento entre `UsuarioEditora` e seus `ConsentimentosLGPD`.
> 3. Execute um teste ponta-a-ponta no fluxo de registro para garantir que o consentimento está sendo gravado corretamente no banco após o sucesso no Frontend.

---

## 6. Guia de Implementação (Prompts para Copilot)

### 3.1. Dashboard e Branding

Utilize os prompts abaixo para refinar a experiência visual e as informações do usuário:

### Prompt 1: Refinamento de Branding e Variáveis CSS (Frontend)
> **Contexto:** O sistema já suporta a definição de uma cor primária via branding, mas precisamos garantir que ela seja aplicada de forma consistente em toda a aplicação usando variáveis CSS de runtime.
>
> **Tarefa:**
> 1. No arquivo `frontend/src/styles.scss`, refine a declaração da variável `--cor-primaria`.
> 2. Crie variações baseadas na cor primária (ex: `--cor-primaria-hover`, `--cor-primaria-alpha`) usando funções de cor do CSS ou calculando-as no `AuthService.ts` se necessário.
> 3. Varra os componentes principais (`register.scss`, `login.scss`, `dashboard.scss`) e substitua cores hardcoded (como o azul `#4f46e5`) pelo uso de `var(--cor-primaria)`.
> 4. Certifique-se de que o `AuthService.applyBranding` continue injetando a cor correta no `:root`.
>
> **Objetivo:** Garantir que, ao trocar a cor da editora, botões, links e destaques mudem automaticamente sem recarregar a página.

### Prompt 2: Refatoração do AppShell e Exibição de Perfil (Frontend)
> **Contexto:** O `AppShell` atual exibe apenas o nome e o avatar. Preciso que ele exiba de forma elegante o nome da Editora e a Role do usuário logado.
>
> **Tarefa:**
> 1. No `frontend/src/app/shared/app-shell/app-shell.html`:
>    - Adicione o nome da editora (`user.nomeEditora`) próximo ao logo ou nome do sistema.
>    - Abaixo do nome do usuário, exiba a Role (ex: "Administrador", "Autor"). Como o usuário pode ter múltiplas roles, exiba a principal ou uma lista separada por vírgula.
> 2. No `app-shell.scss`, ajuste o layout do cabeçalho para acomodar essas novas informações sem quebrar em telas menores (use `flex-box` e `text-overflow`).
> 3. Garanta que o `AuthService` esteja carregando essas informações corretamente no `currentUser$`.
>
> **UX:** O nome da editora deve ter um peso visual secundário, enquanto a Role deve ser exibida como um pequeno "badge" ou texto discreto abaixo do nome do usuário.

### 3.2. Fluxo de Convite (Transição para Slice 2)

Utilize os prompts abaixo para preparar a infraestrutura de convites:

### Prompt 1: Criação da Entidade de Convite (Domain)
> **Contexto:** Estou preparando a base para o Slice 2 (Gestão de Usuários) e preciso de uma entidade para gerenciar convites de novos colegas para a editora.
>
> **Tarefa:**
> 1. No projeto `DomoLibri.Domain/Entities`, crie a entidade `ConviteUsuario`:
>    - Propriedades: `Id` (Guid), `EditoraId` (Guid), `Email` (string), `RoleId` (Guid), `Token` (string), `DataCriacao` (DateTime), `DataExpiracao` (DateTime), `Status` (Enum: `Pendente`, `Aceito`, `Expirado`, `Cancelado`), `ConvidadoPorUsuarioId` (Guid).
> 2. No `DomoLibriDbContext.cs`:
>    - Adicione o `DbSet<ConviteUsuario> Convites`.
>    - Configure a entidade: Chave primária, indexação por `Token` (único) e `Email` (por editora).
>    - Aplique o Filtro Global de Multi-tenancy (`EditoraId`).
>    - Configure os relacionamentos com `Editora`, `Role` e `UsuarioEditora` (quem convidou).
>
> **Padrão:** Siga as convenções de isolamento de dados e auditoria já estabelecidas no projeto.

### Prompt 2: Serviço de Convite e Envio de E-mail (Application/Infrastructure)
> **Contexto:** Preciso de um serviço que gere o convite e envie o e-mail para o novo colega.
>
> **Tarefa:**
> 1. Crie um `InvitationService` no projeto `DomoLibri.Application/Services`.
> 2. Implemente o método `InviteUserAsync(string email, Guid roleId)`:
>    - Valide se o e-mail já possui um convite pendente ou se já é um usuário da editora.
>    - Gere um token seguro e único.
>    - Persista o `ConviteUsuario` no banco de dados.
>    - Envie um e-mail usando o `IEmailService` (já existente) contendo um link para o registro/aceite do convite (ex: `/register?token=XYZ`).
> 3. Certifique-se de que a operação seja registrada no `AuditLog`.
>
> **Regra:** O convite deve expirar em 48 horas por padrão.

### Prompt 3: Componente "Adicionar Colega" (Frontend)
> **Contexto:** Logo após o onboarding (ou no dashboard), o usuário deve ver um botão rápido para convidar colegas.
>
> **Tarefa:**
> 1. No Dashboard (`frontend/src/app/onboarding/dashboard`), adicione um botão ou pequeno card "Convidar Colega".
> 2. Ao clicar, abra um modal simples com um campo de e-mail e um select de Roles (carregando as roles da editora via API).
> 3. Integre com o `OnboardingService` (ou crie um novo `UserService`) para chamar o endpoint de convite.
> 4. Adicione feedback visual (toast/snackbar) de sucesso ou erro.
>
> **Interface:** O formulário deve ser limpo e focado, seguindo a estética do restante do onboarding.

### Prompt 4: Migração e Teste (Infra)
> **Contexto:** Finalizamos a estrutura base de convites.
>
> **Tarefa:**
> 1. Gere uma nova migração do EF Core chamada `AddUserInvitations`.
> 2. Verifique se o e-mail enviado pelo Mailpit contém o link correto e o token gerado.
> 3. Valide se o filtro de multi-tenancy impede que uma editora veja convites de outra.

