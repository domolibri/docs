# Implementação do Onboarding

Este documento descreve a arquitetura e os fluxos do processo de onboarding da editora no SaaS Editorial.

## Diagrama de Classes

O diagrama abaixo ilustra as principais entidades envolvidas no processo de onboarding e o relacionamento entre elas.

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
        +string? TokenConfirmacao (Hashed)
        +DateTime? ExpiracaoToken
        +int AcessosFalhos
        +DateTime? BloqueioAte
        +bool Ativo
    }
    class Role {
        <<enumeration>>
        Admin
        Editor
    }
    Editora "1" -- "*" UsuarioEditora
    UsuarioEditora --> Role
```

## Fluxo de Registro e Confirmação de E-mail

O fluxo a seguir detalha as etapas desde o registro inicial da editora até o primeiro login bem-sucedido após a confirmação do e-mail.

```mermaid
sequenceDiagram
    participant User as Usuário
    participant FE as Frontend (Angular)
    participant API as Backend (ASP.NET Core)
    participant DB as Banco de Dados (PostgreSQL)
    participant Email as IEmailService

    User->>FE: Preenche formulário de registro
    FE->>API: POST /api/auth/register (RegisterRequest)
    API->>DB: Cria Editora e UsuarioEditora (EmailConfirmado=false)
    API->>Email: SendVerificationEmailAsync
    Email-->>User: Envia e-mail com link de confirmação
    API-->>FE: 201 Created (Mensagem de sucesso)
    FE-->>User: Mostra tela de sucesso (verifique e-mail)

    Note over User,Email: Processo de Confirmação

    User->>Email: Clica no link de confirmação
    Email->>FE: Navega para /onboarding/verify-email?email=...&token=...
    FE->>API: POST /api/auth/verify-email (VerifyEmailRequest)
    API->>DB: Valida token e marca EmailConfirmado=true
    API-->>FE: 200 OK
    FE-->>User: Mostra confirmação de sucesso

    Note over User,API: Login

    User->>FE: Realiza Login (email/senha)
    FE->>API: POST /api/auth/login (LoginRequest)
    API->>DB: Verifica credenciais e EmailConfirmado
    API-->>FE: 200 OK + JWT (HttpOnly Cookie)
    FE-->>User: Redireciona para Dashboard
```

## Fluxos Alternativos e Exceções

### E-mail já cadastrado
Se o administrador tentar registrar uma editora com um e-mail que já possui vínculo com outra editora (ou a mesma), o sistema retorna um conflito.

```mermaid
sequenceDiagram
    participant User as Usuário
    participant FE as Frontend (Angular)
    participant API as Backend (ASP.NET Core)
    participant DB as Banco de Dados (PostgreSQL)

    User->>FE: Tenta registrar com e-mail duplicado
    FE->>API: POST /api/auth/register
    API->>DB: Verifica existência do e-mail
    DB-->>API: E-mail encontrado
    API-->>FE: 409 Conflict (E-mail já cadastrado)
    FE-->>User: Exibe mensagem de erro amigável
```

### Token de Confirmação Expirado ou Inválido
O token de confirmação possui validade de 24 horas. Se o usuário clicar em um link expirado, ele deve ser orientado a solicitar um novo e-mail.

```mermaid
sequenceDiagram
    participant User as Usuário
    participant FE as Frontend (Angular)
    participant API as Backend (ASP.NET Core)
    participant DB as Banco de Dados (PostgreSQL)

    User->>FE: Clica em link de confirmação expirado
    FE->>API: POST /api/auth/verify-email
    API->>DB: Valida token e data de expiração
    DB-->>API: Token inválido ou Expirado
    API-->>FE: 400 Bad Request (Token inválido/expirado)
    FE-->>User: Sugere solicitar novo link de confirmação
```

## Endpoints de Autenticação

| Endpoint | Método | Descrição |
| :--- | :--- | :--- |
| `api/auth/register` | POST | Registra uma nova editora e o usuário administrador. |
| `api/auth/verify-email` | POST | Valida o token de confirmação enviado por e-mail. |
| `api/auth/login` | POST | Autentica o usuário e emite o token JWT. |
| `api/auth/logout` | POST | Remove o cookie de autenticação. |
