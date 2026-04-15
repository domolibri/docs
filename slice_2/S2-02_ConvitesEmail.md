# Planejamento de Implementação: S2-02 — Convites por E-mail

Este documento detalha o plano de implementação e refinamento da história **S2-02**, focada no fluxo de convite de usuários para a plataforma multi-editora.

------

### **1. Visão Geral da História**

- **História:** Como administrador de uma editora, quero convidar novos colaboradores informando apenas o e-mail e atribuindo uma Role, para que eles possam se cadastrar e acessar o sistema de forma segura.
- **Prioridade:** P0 (Essencial para o Slice 2).
- **Status Atual:** 
    - **Backend:** Estrutura base concluída (`InvitationService`, `UsersController`, Entidade `ConviteUsuario`).
    - **Frontend:** Componentes base criados (`InviteModal`, `RegisterInvite`), mas necessitam de refinamento de UX e tratamento de estados de erro.

------

### **2. Status Técnico Atual**

#### **2.1 Backend (.NET 8)**
- [x] Entidade `ConviteUsuario` e migration aplicada.
- [x] `IInvitationService` implementado com lógica de:
    - Geração de token (128-bit entropy).
    - Validação de expiração (48h).
    - Verificação de e-mail duplicado/já vinculado.
    - Envio de e-mail via `IEmailService`.
- [x] `UsersController` com endpoints:
    - `POST /api/users/invite`: Dispara o convite.
    - `GET /api/users/invite/{token}`: Recupera detalhes (público).
    - `POST /api/users/invite/accept`: Processa a aceitação (público).

#### **2.2 Frontend (Angular)**
- [x] `InviteModal`: Modal para preenchimento de e-mail e role.
- [x] `RegisterInvite`: Página de destino do link de convite (`/cadastro/convite?token=...`).
- [x] `UserService`: Métodos `inviteUser`, `getInviteDetails` e `acceptInvite`.

------

### **3. O que falta implementar (Backlog de Refinamento)**

#### **1. Refinamento de UX para Usuários Existentes**
Atualmente, a tela de aceitação de convite sempre pede Nome e Senha. 
- **Melhoria:** O backend já identifica se o usuário existe globalmente. O frontend deve usar os detalhes retornados por `GET /api/users/invite/{token}` para:
    - Se o usuário **não existe**: Mostrar campos Nome e Senha.
    - Se o usuário **já existe**: Mostrar apenas mensagem "Você já possui uma conta no DomoLibri. Clique abaixo para aceitar o vínculo com a editora [Nome]." e ocultar campos de senha.

#### **2. Tratamento de Erros e Expiração**
- [ ] Implementar visual amigável no frontend para o erro 400 (Token Expirado) na página de convite, oferecendo um botão para "Solicitar novo convite".
- [ ] Garantir que o link no e-mail use a URL correta configurada via `appsettings.json` (`FrontendUrl`).

#### **3. Estilização do E-mail de Convite**
- [ ] Refinar o `BuildInviteEmailBody` no `InvitationService` para incluir um layout HTML mais profissional.

#### **4. Validação e Testes**
- [ ] Criar testes de integração para o fluxo completo de convite.

------

### **4. Prompts Otimizados para GitHub Copilot**

#### **A. Backend: Refinamento do DTO de Detalhes (`IInvitationService.cs`)**
```
// No IInvitationService.cs, adicione a propriedade 'bool UsuarioExiste' ao InviteDetailsDto.
// No InvitationService.cs, implemente a lógica no GetInviteDetailsAsync para verificar se o e-mail do convite
// já existe na tabela de Usuarios (Global Identity). Retorne true se existir, false caso contrário.
```

#### **B. Frontend: Ajuste Dinâmico do Formulário (`register-invite.ts`)**
```
// No componente RegisterInvite (Angular), utilize o novo campo 'usuarioExiste' do InviteDetailsDto.
// Se 'usuarioExiste' for verdadeiro:
// 1. Remova a obrigatoriedade (Validators.required) dos campos 'nome' e 'senha' no ngOnInit.
// 2. No template (register-invite.html), oculte os inputs de Nome e Senha utilizando @if (!details.usuarioExiste).
// 3. Exiba uma mensagem informativa: "Identificamos que você já possui uma conta global no DomoLibri. Basta clicar no botão abaixo para aceitar o vínculo com a editora [NomeEditora]".
```

#### **C. Backend: Template de E-mail Profissional (`InvitationService.cs`)**
```
// Refatore o método BuildInviteEmailBody no InvitationService.cs para gerar um e-mail HTML moderno.
// 1. Utilize uma estrutura de tabela para garantir compatibilidade com Outlook/Gmail.
// 2. Adicione um cabeçalho com fundo azul (#0078D4) e o texto "DomoLibri" em branco.
// 3. Coloque o convite em destaque com um botão (link estilizado) para "Aceitar Convite".
// 4. No rodapé, inclua a informação de expiração (X horas) e um aviso legal: "Se você não esperava este e-mail, pode ignorá-lo com segurança".
```

#### **D. Teste de Integração (C#)**
```
// Crie um teste de integração no projeto DomoLibri.Tests para o fluxo de convite.
// O teste deve:
// 1. Enviar um convite via POST /api/users/invite com um e-mail novo.
// 2. Buscar o convite no banco de dados para recuperar o token gerado.
// 3. Chamar o endpoint POST /api/users/invite/accept com o token e dados de cadastro.
// 4. Verificar se o registro de VinculoUsuarioEditora foi criado corretamente para o novo usuário.
```

------

### **5. Critérios de Pronto (DoD)**

- [ ] Administrador consegue enviar convite e visualizar confirmação (Toast/Snackbar).
- [ ] Usuário recebe e-mail formatado com link funcional.
- [ ] Link de convite redireciona para página de cadastro com token pré-preenchido.
- [ ] Sistema diferencia corretamente entre criação de nova conta e apenas novo vínculo (para usuários já existentes).
- [ ] Após aceitar, o usuário é redirecionado para o Login e consegue acessar a editora convidada.
- [ ] O Log de Auditoria registra a ação `ConviteAceito`.
