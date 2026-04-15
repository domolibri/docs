Aqui está o detalhamento atualizado para a implementação da história **S2-01 — Cadastro de usuários**, focado no que **realmente falta** ser feito, dado que a infraestrutura básica (entidades e serviço de convite) já está parcialmente implementada.

------

### **Visão Geral da História (S2-01)**

- **História:** Como admin, quero gerenciar (listar e adicionar) usuários internos e parceiros da minha editora.
- **Prioridade:** P0 (Essencial para o Slice 2).
- **Status Atual:** Entidades `Usuario` e `VinculoUsuarioEditora` criadas. `InvitationService` básico e `UsersController` com endpoint de convite já existem.

------

### **O que falta implementar (Backlog Técnico)**

#### **1. Backend (.NET 8)**

**1.1 Listagem de Usuários (Missing)**
- **Endpoint:** `GET /api/users` no `UsersController`.
- **Lógica:** Retornar a lista de usuários vinculados à editora atual (o `Global Query Filter` no `VinculosUsuarioEditora` já garantirá o isolamento).
- **DTO de Saída:** `UserListItemDto` (Id do vínculo, Nome, Email, TipoVinculo, Ativo, Roles).

**1.2 Melhoria no InvitationService (Refinement)**
- **Lógica:** Garantir que se o usuário já existir globalmente, o convite ainda seja necessário para que ele aceite o vínculo com a nova editora (Segurança/LGPD).
- **Status:** O `InvitationService.InviteUserAsync` já faz isso, mas precisamos garantir que o frontend consuma corretamente.

**1.3 Endpoint de Roles (Exists)**
- **Status:** `GET /api/users/roles` já implementado no `UsersController`.

#### **2. Frontend (Angular)**

**2.1 Service de Usuários (Missing)**
- **Arquivo:** `src/app/shared/user.service.ts` (ou similar).
- **Métodos:** `getUsers()`, `inviteUser(email, roleId)`.

**2.2 Tela de Gestão de Usuários (Missing)**
- **Rota:** `/usuarios` (protegida por `authGuard`).
- **Componente de Listagem:** Tabela exibindo os usuários da editora, seus papéis (Roles) e status.
- **Botão "Convidar Usuário":** Abre o modal de convite.

**2.3 Modal de Convite (Missing)**
- **Formulário:** Campo de E-mail e Seleção de Role (carregada via `api/users/roles`).
- **Integração:** Chama `POST /api/users/invite`.

------

### **Prompts Otimizados para GitHub Copilot**

Aqui estão os prompts que você pode usar para gerar rapidamente o código que falta. Copie e cole no Copilot Chat ou diretamente nos arquivos.

#### **1. Backend: Listagem de Usuários (`UsersController.cs`)**

```
// Implement a GET api/users endpoint in UsersController.
// 1. Fetch all VinculosUsuarioEditora for the current tenant (the Global Query Filter handles isolation).
// 2. Include the related Usuario entity and Roles.
// 3. Select into a list of DTOs with:
//    - Id (from Vinculo)
//    - Nome (from Usuario)
//    - Email (from Usuario)
//    - TipoVinculo (enum string)
//    - Ativo (boolean)
//    - Roles (list of names)
// 4. Return the list.
```

#### **2. Frontend: Atualização do `user.service.ts`**

```
// Add a method getUsers() to the UserService class.
// It should make a GET request to environment.apiUrl + '/api/users'.
// Return an Observable of UserListItem[].
// Define the UserListItem interface with properties: id, nome, email, tipoVinculo, ativo, and roles (string[]).
```

#### **3. Frontend: Componente de Listagem de Usuários**

Se você ainda não criou o componente de listagem, use este prompt para gerar a estrutura base:

```
// Create an Angular component UserListComponent (Standalone).
// 1. Use the new signals-based approach (signal, effect, inject).
// 2. Inject UserService and SnackbarService.
// 3. Define a users signal and fetch the data on init from userService.getUsers().
// 4. Add a method openInviteModal() to toggle a visibility signal for <app-invite-modal>.
// 5. Template: Use a modern HTML table with Tailwind-like CSS classes or plain SCSS.
// 6. Display columns: Name, Email, Role, Status, and Actions.
```

#### **4. Integração: Rota e Menu**

```
// In app.routes.ts, add a new route 'usuarios' under the authenticated children array.
// Point it to UserListComponent using loadComponent.
// In app-shell.ts, add a link to '/usuarios' in the navigation menu with the label 'Usuários'.
```

------

### **Critérios de Pronto (DoD) Atualizados**

- [ ] Administrador consegue ver a lista de todos os usuários vinculados à sua editora.
- [ ] Administrador consegue enviar um convite para um novo e-mail associando-o a uma Role.
- [ ] O sistema impede convites duplicados para o mesmo e-mail na mesma editora (já implementado no backend).
- [ ] O `AuditLog` registra quem enviou o convite.