# Plano de Atualização — Arquitetura Multi-Editora (Identidade Global)

Este documento detalha o plano para transicionar do modelo de "Usuário por Tenant" para o modelo de "Identidade Global com Múltiplos Vínculos", conforme a [Documentação Multi-Editora](../Documentacao-de-Arquitetura-SaaS-Editorial-Consolidada-MultiEditora.md).

## 1. Mudanças Estruturais

### 1.1. Modelo de Dados (Domain)
- **Atual:** `UsuarioEditora` contém tanto a identidade (email/senha) quanto o vínculo com a editora.
- **Novo:**
    - `Usuario`: Entidade global com `Id`, `Email`, `SenhaHash`, `Nome`, `StatusConta`.
    - `VinculoUsuarioEditora`: Entidade de junção entre `Usuario` e `Editora` com `TipoVinculo`, `Roles` e `Permissoes`.

### 1.2. Autenticação e Autorização
- O JWT deve conter o `UsuarioId` global.
- Ao logar, se o usuário tiver mais de uma editora, ele deve selecionar o contexto ativo (ou o sistema assume a última/padrão).
- As Claims de permissão e roles devem ser resolvidas com base no `VinculoUsuarioEditora` da editora ativa.

---

## 2. Cronograma de Implementação (Slices)

### Slice A: Refatoração do Core Domain e Migração
- [ ] Criar entidade `Usuario`.
- [ ] Renomear/Refatorar `UsuarioEditora` para `VinculoUsuarioEditora`.
- [ ] Ajustar relacionamentos no `DomoLibriDbContext`.
- [ ] Criar migração de dados para mover credenciais de `UsuarioEditora` para `Usuario`.

### Slice B: Ajuste da Camada de Aplicação (AuthService)
- [ ] Atualizar `RegisterAsync` para verificar se o `Usuario` já existe antes de criar um novo.
- [ ] Atualizar `LoginAsync` para validar credenciais no `Usuario` global.
- [ ] Implementar endpoint `GET /api/auth/contextos` para listar editoras vinculadas ao usuário.
- [ ] Ajustar `GenerateJwt` para incluir o contexto da editora selecionada.

### Slice C: Frontend - Seleção de Contexto
- [ ] Adicionar tela/modal de "Selecionar Editora" após o login (se > 1 vínculo).
- [ ] Atualizar `AuthService.ts` no Angular para gerenciar o `tenantId` ativo.
- [ ] Ajustar interceptors para enviar o `EditoraId` do contexto selecionado.

---

## 3. Guia de Implementação (Prompts para Copilot)

### Prompt 1: Refatoração das Entidades de Identidade (Domain)
> **Contexto:** Estou atualizando o sistema para suportar identidade global. Um usuário (mesmo email/senha) pode estar vinculado a várias editoras.
>
> **Tarefa:**
> 1. No `DomoLibri.Domain/Entities`, crie a entidade `Usuario` movendo as seguintes propriedades de `UsuarioEditora`: `Email`, `SenhaHash`, `Nome`, `EmailConfirmado`, `TokenConfirmacao`, `ExpiracaoToken`, `TokenRedefinicaoSenha`, `ExpiracaoTokenRedefinicaoSenha`, `SenhaAlteradaEm`, `AcessosFalhos`, `BloqueioAte`.
> 2. Renomeie a classe `UsuarioEditora` para `VinculoUsuarioEditora`.
> 3. Na nova `VinculoUsuarioEditora`:
>    - Adicione `Guid UsuarioId` e a propriedade de navegação `Usuario Usuario`.
>    - Mantenha `EditoraId`, `Roles` e `Ativo`.
>    - Adicione `DateTime DataEntrada` e `TipoVinculo` (enum).
> 4. No `DomoLibriDbContext.cs`:
>    - Atualize as configurações para refletir que o `Email` é único na tabela `Usuarios` (global), não mais por `EditoraId`.
>    - Configure o relacionamento 1:N entre `Usuario` e `VinculoUsuarioEditora`.
>
> **Atenção:** Mantenha o Filtro Global de Multi-tenancy na entidade `VinculoUsuarioEditora`, mas a entidade `Usuario` deve ser global (sem filtro de EditoraId).

### Prompt 2: Ajuste do Fluxo de Autenticação (Application)
> **Contexto:** Agora a autenticação é global, mas a autorização depende do vínculo ativo.
>
> **Tarefa:**
> 1. No `AuthService.cs`, método `LoginAsync`:
>    - Busque o usuário pelo e-mail na tabela global `Usuarios`.
>    - Valide a senha.
>    - Retorne a lista de vínculos (`VinculoUsuarioEditora`) que este usuário possui.
> 2. Crie um novo método `SelectContextAsync(Guid usuarioId, Guid editoraId)`:
>    - Valide se o vínculo existe e está ativo.
>    - Gere o JWT contendo as claims da editora selecionada e as roles do respectivo vínculo.
> 3. No método `RegisterAsync`:
>    - Verifique se já existe um `Usuario` com o e-mail informado.
>    - Se existir, apenas crie o `VinculoUsuarioEditora` e a nova `Editora`.
>    - Se não existir, crie ambos.
>
> **Regra:** O `EditoraId` no JWT agora vem do vínculo selecionado.

### Prompt 3: Migração de Dados (Infrastructure)
> **Contexto:** Temos dados na tabela `UsuariosEditora` que precisam ser migrados para a nova estrutura de `Usuarios` + `Vinculos`.
>
> **Tarefa:**
> 1. Crie uma migração do EF Core que:
>    - Cria a tabela `Usuarios`.
>    - Insere registros únicos de e-mail da tabela `UsuariosEditora` na tabela `Usuarios`.
>    - Atualiza a (agora renomeada) tabela `VinculosUsuarioEditora` para apontar para o `UsuarioId` correto.
>    - Remove as colunas de credenciais redundantes da tabela de vínculos.
