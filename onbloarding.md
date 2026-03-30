<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Atualize a especificação deste slice incluindo a validação do email, listando com clareza as entidades e seus campos, o fluxo do usuário, os fluxos de excessão, quais endpoints REST criar por etapa e quais endpoints correspondem a cada tela.

Aqui está a **especificação atualizada do slice "Onboarding da Editora"**, agora com validação de email obrigatória. Mantive o foco vertical, com isolamento multi-tenant via EditoraId e JWT após confirmação.[^1][^2][^3][^4]

## Entidades Principais

Adicionei campos para confirmação de email no UsuarioEditora. Use Global Query Filter no DbContext para EditoraId.

- **Editora**:
    - Id (Guid, PK)
    - Nome (string, obrigatório, máx. 100)
    - Slug (string único, gerado de Nome)
    - LogoUrl (string, opcional)
    - CorPrimaria (string hex, opcional)
    - DataCriacao (DateTime)
    - Ativo (bool, default true)[^2][^1]
- **UsuarioEditora**:
    - Id (Guid, PK)
    - EditoraId (Guid, FK obrigatória)
    - Email (string único por tenant, obrigatório)
    - SenhaHash (string)
    - Nome (string, obrigatório)
    - Role (enum: Admin=0, Editor=1)
    - EmailConfirmado (bool, default false)
    - EmailConfirmacaoTokenHash (string)
    - EmailConfirmacaoExpiraEm (DateTime?)
    - Ativo (bool, default true)[^3][^4]


## Fluxo Principal do Usuário

1. **Register**: Preenche form (nome editora, email, senha, nome admin).
2. **Aguardar Email**: Recebe email com link de confirmação.
3. **Confirmar Email**: Clica no link (abre tela de sucesso).
4. **Login**: Faz login com email/senha (agora liberado).
5. **Dashboard**: Lista usuários e configura branding.[^5][^2]

## Fluxos de Exceção

- Email já existe: 409 Conflict, mensagem "Email já cadastrado".
- Token inválido/expirado: 400 Bad Request, "Link inválido. Solicite novo email".
- Senha fraca: 400, validação no backend.
- Erro envio email: 500, mas cadastra com EmailConfirmado=false; usuário resend.
- Timeout token (24h): usuário resend novo token.[^6][^3]


## Endpoints REST por Etapa

Controllers em `OnboardingController` e `AuthController`. Use JWT Bearer após confirmação.

**Register**:

- POST /api/onboarding/register-editora → Cria editora/usuário pendente + envia email.

**Confirmação**:

- POST /api/onboarding/confirmar-email → Valida token, marca confirmado.
- POST /api/onboarding/reenviar-email → Gera novo token.

**Login**:

- POST /api/auth/login → Só emite JWT se EmailConfirmado=true.

**Dashboard/Branding** (pós-login):

- GET /api/editora/usuarios
- POST /api/editora/usuarios
- PATCH /api/editora/branding[^2][^3]


## Endpoints por Tela Angular

Telas em `app/onboarding/`.


| Tela Angular | Endpoint REST | Método | Estado |
| :-- | :-- | :-- | :-- |
| register.component.ts | /api/onboarding/register-editora | POST | Inicial |
| verify-email.component.ts | /api/onboarding/reenviar-email | POST | Aguardando |
| confirm-email.component.ts | /api/onboarding/confirmar-email | POST | Link clicado |
| login.component.ts | /api/auth/login | POST | Pós-confirmação |
| dashboard.component.ts | /api/editora/usuarios | GET | Logado |
| branding.component.ts | /api/editora/branding | PATCH | Logado |

Implemente com MediatR para commands (CreateEditoraCommand) e envie email via SMTP ou SendGrid no handler.[^4][^3]

<div align="center">⁂</div>

[^1]: https://www.perplexity.ai/search/de79f0ee-75f6-400d-8cfd-b68d56238e15

[^2]: https://www.perplexity.ai/search/0968e29e-feaf-4cfb-8b72-210333223544

[^3]: https://www.yogihosting.com/aspnet-core-identity-email-confirmation/

[^4]: https://code-maze.com/email-confirmation-aspnet-core-identity/

[^5]: https://auth0.com/blog/enforce-customer-email-verification-b2b-saas-blazor-app/

[^6]: https://www.rfc-editor.org/rfc/rfc9457.html

