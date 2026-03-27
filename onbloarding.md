<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# “Estou desenvolvendo um SaaS editorial multi‑tenant com .NET 8 Web API e Angular.

Quero o próximo slice de implementação: onboarding da editora.

Liste, em português, com clareza:

as entidades principais e seus campos (ex.: Editora, UsuarioEditora),

o fluxo de usuário (register → login → dashboard → branding),

qual endpoint REST devo criar por etapa,

qual endpoint corresponde a cada tela.
Dê uma sugestão de nome de pasta e estrutura de projeto (ex.: backend/src/... e frontend/...).”

Aqui está o próximo slice de implementação para o onboarding da editora no seu SaaS multi-tenant com .NET 8 Web API e Angular, focado em um fluxo vertical simples e isolado logicamente por tenant.[^1][^2][^3]

## Entidades Principais

As entidades centrais para multi-tenancy usam EditoraId como chave de isolamento (com Global Query Filter no EF Core).[^3]

- **Editora**: Id (Guid), Nome (string, ex: "DomoLibri"), Slug (string único), LogoUrl (string), CorPrimaria (string hex), DataCriacao (DateTime), Ativo (bool).[^4]
- **UsuarioEditora**: Id (Guid), EditoraId (Guid, FK obrigatória), Email (string único por tenant), SenhaHash (string), Nome (string), Role (enum: Admin, Editor), Ativo (bool).[^4][^3]


## Fluxo de Usuário

O fluxo segue register → login → dashboard → branding, com isolamento automático via EditoraId no JWT.[^5][^4]

1. Register: Nova editora cria conta e cadastra o primeiro admin.
2. Login: Autentica com email/senha, recebe JWT com EditoraId.
3. Dashboard: Lista usuários da editora e inicia setup.
4. Branding: Configura logo, cores e nome da editora.[^4]

## Endpoints REST por Etapa

Crie controllers em `backend/src/DomoLibri.Api/Controllers/Onboarding/`, com middleware para resolver TenantId via header ou subdomínio.[^3]

**Register**:

- POST /api/onboarding/register-editora (cria Editora + primeiro UsuarioEditora admin).

**Login**:

- POST /api/auth/login (retorna JWT com EditoraId e roles).[^5]

**Dashboard**:

- GET /api/editora/usuarios (lista usuários da editora logada).
- POST /api/editora/usuarios (cria novo usuário).[^3]

**Branding**:

- PATCH /api/editora/branding (atualiza logo, cores).[^4]


## Endpoints por Tela

Mapeie rotas Angular para esses endpoints via HttpClient no service `OnboardingService`.


| Tela Angular | Endpoint REST | Método |
| :-- | :-- | :-- |
| Register (register.component.ts) | /api/onboarding/register-editora | POST |
| Login (login.component.ts) | /api/auth/login | POST |
| Dashboard (dashboard.component.ts) | /api/editora/usuarios | GET |
| Branding (branding.component.ts) | /api/editora/branding | PATCH |

## Estrutura de Projeto

Use Clean Architecture para escalabilidade, com workspace VS Code único (como discutido).[^1]

```
domolibri-editorial-saas/
├── backend/
│   ├── src/
│   │   ├── DomoLibri.Domain/          # Entidades: Editora, UsuarioEditora
│   │   ├── DomoLibri.Application/     # Commands/Queries para onboarding (MediatR)
│   │   ├── DomoLibri.Infrastructure/  # DbContext com Global Filter, Repos
│   │   └── DomoLibri.Api/             # Controllers/Onboarding, Program.cs
│   ├── tests/                         # Unit/Integration
│   └── docker-compose.yml             # Postgres + API
└── frontend/
    ├── src/
    │   ├── app/
    │   │   ├── onboarding/             # Components: register, login, dashboard, branding
    │   │   ├── services/               # OnboardingService (HttpClient)
    │   │   └── shared/                 # Guards, interceptors (JWT + TenantId)
    │   └── assets/                     # Logos placeholders
    └── angular.json                    # ng serve proxy para API
```

Rode com `docker-compose up` + `ng serve` no VS Code compound launch.[^2]
<span style="display:none">[^10][^11][^12][^13][^14][^15][^16][^17][^18][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: https://www.perplexity.ai/search/598830e9-621a-464c-9ee6-bd06337ec263

[^2]: https://www.perplexity.ai/search/526f4e0f-9f87-4fe9-b930-7fcf8ed740b5

[^3]: https://www.alertu.io/implementing-multi-tenancy-in-dotnet/

[^4]: https://www.perplexity.ai/search/0968e29e-feaf-4cfb-8b72-210333223544

[^5]: https://dev.to/cristiansifuentes/microsoft-entra-id-multi-tenant-saas-net-8-web-api-a-production-grade-playbook-3-tenants-3-53f5

[^6]: https://www.reddit.com/r/dotnet/comments/1m3suos/anyone_know_a_decent_net_template_with/

[^7]: https://www.devmedia.com.br/transforme-aplicacoes-web-em-servicos-multi-tenant/28881

[^8]: https://aws.amazon.com/pt/blogs/aws-brasil/ma-plataforma-saas-multi-tenant-para-etiquetagem-automatica-de-videos-parte-i-o-front-end/

[^9]: https://www.linkedin.com/posts/angular-material-dev_creating-onboarding-flows-shouldnt-take-activity-7404610878735863810-Sram

[^10]: https://jasontaylor.dev/clean-architecture-getting-started/

[^11]: https://www.dio.me/articles/multi-tenant-no-aspnet-core-controle-de-acesso-baseado-em-tenantid-6927f06925cc

[^12]: https://aws.amazon.com/pt/blogs/aws-brasil/criando-uma-solucao-saas-multi-tenant-usando-servicos-serverless-da-aws/

[^13]: https://www.youtube.com/watch?v=y37_8-M1i-c

[^14]: https://flows.sh/technology/angular-user-onboarding

[^15]: https://www.youtube.com/watch?v=c7hZPxBnDq4

[^16]: https://macoratti.net/23/07/c_multitenant1.htm

[^17]: https://docs.logto.io/pt-BR/use-cases/multi-tenancy/build-multi-tenant-saas-application

[^18]: https://www.reddit.com/r/dotnet/comments/1ex2dcs/how_to_implement_multitenancy_in_aspnet_core_with/

