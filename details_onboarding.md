# Documentação Técnica: Onboarding de Editoras

Este documento detalha a arquitetura, os fluxos de dados e as especificações da API para o processo de onboarding de novas editoras no sistema SaaS Editorial.

---

## 1. Modelo de Dados (Diagrama de Classes)

O diagrama abaixo ilustra as entidades centrais e seus relacionamentos. O sistema utiliza uma arquitetura multi-tenant isolada por `EditoraId`.

```mermaid
classDiagram
    class Editora {
        +Guid Id
        +string Nome
        +string Slug
        +string? LogoUrl
        +string? CorPrimaria
        +DateTime DataCriacao
        +bool Ativo
    }
    class UsuarioEditora {
        +Guid Id
        +Guid EditoraId
        +string Email
        +string SenhaHash
        +string Nome
        +Role Role
        +bool EmailConfirmado
        +string? TokenConfirmacao
        +DateTime? ExpiracaoToken
        +string? TokenRedefinicaoSenha
        +DateTime? ExpiracaoTokenRedefinicaoSenha
        +int AcessosFalhos
        +DateTime? BloqueioAte
        +bool Ativo
    }
    class Role {
        <<enumeration>>
        Admin
        Editor
    }
    Editora "1" -- "*" UsuarioEditora : possui
    UsuarioEditora --> Role : atribuída
```

---

## 2. Fluxos de Sequência

### 2.1. Registro e Verificação de E-mail (Fluxo Principal)
O registro cria a editora e o usuário administrador em estado inativo até a confirmação do e-mail.

```mermaid
sequenceDiagram
    participant User as Usuário
    participant FE as Frontend (Angular)
    participant API as Backend (ASP.NET Core)
    participant DB as Banco de Dados
    participant Email as Mailpit/SMTP

    User->>FE: Preenche formulário de registro
    FE->>API: POST /api/auth/register
    API->>DB: Salva Editora e Usuário (EmailConfirmado=false)
    API->>Email: Envia link com Token de Confirmação
    API-->>FE: 201 Created
    FE-->>User: Exibe tela "Verifique seu e-mail"

    Note over User,Email: Processo de Confirmação

    User->>Email: Clica no link (token)
    Email->>FE: Navega para /verify-email?token=...
    FE->>API: POST /api/auth/verify-email
    API->>DB: Valida Token e marca EmailConfirmado=true
    API-->>FE: 200 OK
    FE-->>User: Redireciona para Login
```

### 2.2. Recuperação de Senha (Fluxo Alternativo)
Permite que o usuário redefina sua senha caso a tenha esquecido.

```mermaid
sequenceDiagram
    participant User as Usuário
    participant FE as Frontend
    participant API as Backend
    participant Email as SMTP

    User->>FE: Solicita recuperação de senha
    FE->>API: POST /api/auth/forgot-password
    API->>Email: Envia link com Token de Redefinição
    API-->>FE: 200 OK (Mensagem Genérica)
    
    User->>Email: Clica no link
    Email->>FE: Navega para /reset-password?token=...
    FE->>API: POST /api/auth/reset-password
    API->>DB: Atualiza SenhaHash e limpa Tokens
    API-->>FE: 200 OK
```

---

## 3. Fluxos de Exceção e Segurança

### 3.1. Bloqueio de Conta (Account Lockout)
Após 5 tentativas falhas consecutivas, a conta é bloqueada por 15 minutos para prevenir ataques de força bruta.

| Condição | Resposta do Sistema |
| :--- | :--- |
| **Tentativa 1-4 falha** | Retorna 401 Unauthorized e incrementa `AcessosFalhos`. |
| **Tentativa 5 falha** | Define `BloqueioAte` para 15 minutos no futuro. |
| **Tentativa durante bloqueio** | Retorna 423 Locked informando o tempo restante. |

### 3.2. Rate Limiting
A API de autenticação possui um limite global de **5 requisições por minuto por IP** para evitar abusos.
*   **Status Code:** `429 Too Many Requests`.

### 3.3. E-mail Duplicado
O sistema não permite o registro de uma editora se o e-mail do administrador já estiver em uso.
*   **Status Code:** `409 Conflict`.

---

## 4. Documentação da API (v1)

### POST `/api/auth/register`
Registra uma nova editora e seu primeiro administrador.

**Request Body:**
```json
{
  "nomeEditora": "Minha Editora",
  "emailAdmin": "admin@editora.com",
  "senha": "SenhaForte123!",
  "nomeAdmin": "João Silva"
}
```

**Responses:**
*   `201 Created`: Sucesso. Retorna `editoraId`.
*   `400 Bad Request`: Erro de validação (ex: senha fraca).
*   `409 Conflict`: E-mail já cadastrado.

---

### POST `/api/auth/login`
Autentica o usuário e define um **HttpOnly Cookie** chamado `access_token`.

**Request Body:**
```json
{
  "email": "admin@editora.com",
  "senha": "SenhaForte123!"
}
```

**Responses:**
*   `200 OK`: Sucesso. Retorna o token JWT e dados básicos do usuário.
*   `401 Unauthorized`: Credenciais inválidas ou e-mail não confirmado.
*   `423 Locked`: Conta temporariamente bloqueada.

---

### POST `/api/auth/verify-email`
Confirma a posse do e-mail através do token.

**Request Body:**
```json
{
  "email": "admin@editora.com",
  "token": "token_gerado_pela_api"
}
```

**Responses:**
*   `200 OK`: E-mail confirmado com sucesso.
*   `400 Bad Request`: Token inválido ou expirado.

---

### POST `/api/auth/reset-password`
Define uma nova senha utilizando o token de redefinição.

**Request Body:**
```json
{
  "email": "admin@editora.com",
  "token": "token_redefinicao",
  "novaSenha": "NovaSenhaForte123!"
}
```

**Responses:**
*   `200 OK`: Senha atualizada com sucesso.
*   `400 Bad Request`: Token inválido ou senha não atende aos requisitos.
