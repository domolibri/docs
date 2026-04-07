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

