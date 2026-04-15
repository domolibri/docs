# Planejamento de Implementação: S2-03 — Roles e Permissões

Este documento detalha o plano de implementação da história **S2-03**, focada na atribuição de papéis (Roles) aos usuários e no controle de acesso baseado em permissões (RBAC).

------

### **1. Visão Geral da História**

- **História:** Como administrador, quero atribuir um ou mais papéis (Roles) aos usuários vinculados à minha editora, para que cada colaborador tenha acesso apenas às funcionalidades necessárias para sua função.
- **Prioridade:** P0 (Essencial para o Slice 2).
- **Status Atual:** 
    - **Backend:** Infraestrutura de permissões baseada em Claims está funcional. As entidades `Role` e `Permission` e o `PermissionAuthorizationHandler` existem.
    - **Frontend:** A listagem de usuários já exibe as roles, mas não há interface para edição/atribuição.

------

### **2. Status Técnico Detalhado**

#### **2.1 Backend (.NET 8)**
- [x] Entidades `Role` e `Permission` mapeadas (M:N).
- [x] `SystemPermissions.cs` define o "Source of Truth" das permissões fixas no código.
- [x] `AuthService` injeta permissões no JWT durante o login (`SelectContextAsync`).
- [ ] **Falta:** Endpoint para atualizar as roles de um vínculo específico (`PUT /api/users/{vinculoId}/roles`).
- [ ] **Falta:** Endpoint para listar todas as permissões disponíveis (para futura interface de criação de roles).

#### **2.2 Frontend (Angular)**
- [x] `UserService.getRoles()` recupera as roles da editora.
- [ ] **Falta:** Atualizar `UsuarioFormComponent` para incluir um seletor de Roles.
- [ ] **Falta:** Implementar método `updateUserRoles` no `UsuarioService`.

------

### **3. Plano de Implementação**

#### **Etapa 1: Backend — Endpoint de Atribuição**
- Implementar no `UsersController` o endpoint `PUT /api/users/{id}/roles`.
- O ID deve ser o do `VinculoUsuarioEditora`.
- O payload deve ser uma lista de IDs de Roles.
- Validar se as Roles pertencem ao mesmo Tenant (EditoraId).

#### **Etapa 2: Frontend — Interface de Seleção**
- No `UsuarioFormComponent`, adicionar um campo de seleção múltipla para Roles.
- Carregar as roles disponíveis através de `userService.getRoles()`.
- Ao salvar um usuário existente, disparar a atualização de roles.

#### **Etapa 3: Backend — Refinamento de Claims**
- Garantir que ao alterar as roles de um usuário, ele seja forçado a re-autenticar (ou o frontend atualize o token) para que as novas permissões entrem em vigor no JWT.

------

### **4. Prompts Otimizados para GitHub Copilot**

#### **A. Backend: Endpoint de Atualização de Roles (`UsersController.cs`)**
```
// Implemente um endpoint [HttpPut("{id}/roles")] no UsersController.cs.
// 1. O parâmetro 'id' é o Guid do VinculoUsuarioEditora.
// 2. O corpo da requisição deve ser um Guid[] com os Ids das Roles.
// 3. Busque o vínculo no banco, incluindo a coleção de Roles (.Include(v => v.Roles)).
// 4. Verifique se todas as Roles fornecidas existem e pertencem à editora atual (tenant isolation).
// 5. Atualize a coleção v.Roles do vínculo e salve as alterações.
// 6. Retorne NoContent().
```

#### **B. Frontend: Seletor de Roles no Formulário (`usuario-form.component.ts`)**
```
// No componente UsuarioFormComponent, adicione um seletor de múltiplas opções para Roles.
// 1. Injete o UserService e utilize o sinal 'roles' (ou carregue via Observable) para listar as opções.
// 2. No formulário (ReactiveForms), adicione o campo 'roleIds' como um FormControl de array.
// 3. No template (usuario-form.component.html), use um <select multiple> ou uma lista de checkboxes estilizada.
// 4. Ao submeter, se o usuário já possuir um ID (edição), chame o novo método do service para atualizar as roles.
```

#### **C. Frontend: Atualização do Service (`usuario.service.ts`)**
```
// Adicione o método updateRoles(vinculoId: string, roleIds: string[]) ao UsuarioService.
// Deve realizar um PUT para environment.apiUrl + `/api/users/${vinculoId}/roles`.
```

------

### **5. Critérios de Pronto (DoD)**

- [ ] Administrador consegue selecionar uma ou mais roles ao convidar ou editar um usuário.
- [ ] As roles são persistidas corretamente na tabela associativa entre `Vinculo` e `Role`.
- [ ] O JWT gerado após a troca de roles contém as permissões (claims) correspondentes às novas roles.
- [ ] O sistema impede a atribuição de roles que não pertencem à editora do administrador.
- [ ] A interface reflete as roles atuais do usuário ao abrir o formulário de edição.
