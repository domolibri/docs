# Changelog

## [2026-03-30]

### Segurança (Security hardening)
- **Hashed Tokens**: Implementado hashing (SHA-256) para tokens de verificação de e-mail e redefinição de senha, eliminando o armazenamento em texto puro no banco de dados.
- **Account Lockout**: Implementada lógica de bloqueio de conta após 5 tentativas falhas de login (bloqueio por 15 minutos) para mitigar ataques de força bruta.
- **Password Complexity**: Adicionada validação de complexidade de senha via Regex no endpoint de redefinição de senha (igualando à política de registro).
- **XSS/Email Injection Prevention**: Adicionada codificação HTML (HtmlEncode) para nomes de usuários em corpos de e-mail enviados pelo `EmailService`.
- **Secrets Management**: Refatorado `docker-compose.yml` para utilizar variáveis de ambiente para segredos (DB, JWT) e restringir o acesso ao banco de dados e Mailpit apenas ao `localhost`.
- **Session Duration**: Ajustada expiração padrão do JWT para 1 hora (alinhado com as melhores práticas de SaaS).

### Backend (DomoLibri.Infrastructure)
- **Fixed**: Implementado método faltante `SendEmailAsync` na interface `IEmailService` e suas implementações, corrigindo erro de compilação no `AuthService`.
- **Fixed**: Correção na lógica de registro para enviar e-mail de aviso em caso de tentativa com e-mail já cadastrado (prevenção de enumeração).
- **Added**: Migração `AddEmailConfirmation` para suportar fluxo de verificação de e-mail.
- **Fixed**: Ajustes no `AuthService` para lidar com a nova lógica de autenticação e verificação.

### Frontend (Angular)
- **Added**: Novos componentes de onboarding: `register-success`, `verify-email`, e `verify-email-failed`.
- **Added**: Implementao do `AuthService` no frontend com suporte a `checkSession` e interceptores de JWT e Tenant.
- **Fixed**: Erro TypeScript (TS2345) em `frontend/src/app/shared/auth.service.ts`: corrigida a inferncia de tipos no mtodo `checkSession` ao adicionar tipagem explcita ao `HttpClient.get<any>()` e ao operador `map((): boolean => true)` do RxJS.

### Geral
- **Added**: Arquivo `.geminiignore` na raiz do projeto, consolidando as regras de excluso dos diretrios `frontend` e `backend` para otimizar o contexto da IA.
