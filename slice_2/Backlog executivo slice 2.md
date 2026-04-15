<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# faça isso

Perfeito — abaixo está o **backlog executivo do Slice 2**, já estruturado para implementação e alinhado à arquitetura multi-tenant, incluindo o suporte a **domínio próprio da editora**. O slice cobre o pico E03 do backlog inicial e adiciona a parte de white-label/domínio porque ela é parte da identidade do tenant e depende diretamente do contexto de acesso por editora.[^1][^2]

## Slice 2 — Gestão de usuários, roles e domínio próprio

**Nome do slice:** Gestão de Usuários, Permissões e Domínio Próprio
**Objetivo:** permitir que a editora administre usuários, convites, vínculos, roles e permissões, além de configurar e operar seu próprio domínio com branding consistente.[^2][^1]

### Entrega de negócio

- Cadastro de usuários internos e parceiros.
- Convite e ativação de usuários.
- Atribuição de roles e permissões por vínculo com a editora.
- Inativação sem perda de histórico.
- Configuração e resolução de domínio próprio.
- Carregamento do branding correto por tenant.[^1][^2]


## Escopo funcional

### Itens P0

- E03-01: Cadastrar usuários internos e parceiros.
- E03-02: Convidar usuários por e-mail.
- E03-03: Atribuir roles editoriais.
- E03-04: Restringir acesso por permissões e por `EditoraId`.
- Configurar domínio próprio da editora.
- Resolver tenant a partir do host acessado.[^2][^1]


### Itens P1 recomendados no mesmo slice

- Vincular usuários a múltiplas editoras.
- Inativar vínculo sem apagar histórico.
- Registrar auditoria das ações administrativas.
- Validar domínio e status de verificação.
- Exibir marca da editora corretamente no frontend.[^1]


## Entidades do slice

- `Usuario`.
- `VinculoUsuarioEditora`.
- `Role`.
- `Permissao`.
- `UsuarioRole`.
- `ConviteUsuario`.
- `Editora` com `Slug`, `LogoUrl`, `CorPrimaria` e `DominioCustomizado`.[^1]


### Regras principais

- Usuário é global; autorização é por vínculo.
- Role pertence ao vínculo, não ao usuário.
- Tudo deve respeitar isolamento por `EditoraId`.
- A edição do domínio próprio deve ser auditada.
- O tenant deve ser resolvido pelo host quando houver domínio próprio.[^2][^1]


## APIs previstas

| API | Finalidade | Observação |
| :-- | :-- | :-- |
| `GET /api/usuarios` | Listar usuários do tenant | Filtra por `EditoraId` [^1]. |
| `POST /api/usuarios` | Criar usuário e vínculo | Reaproveita identidade global se já existir [^1]. |
| `POST /api/usuarios/convites` | Gerar convite | Suporta novo e-mail ou usuário já cadastrado [^1]. |
| `POST /api/usuarios/{id}/vinculos` | Criar novo vínculo | Permite múltiplas editoras por usuário [^1]. |
| `PUT /api/vinculos/{id}/roles` | Atribuir roles | Role por vínculo [^1]. |
| `PUT /api/vinculos/{id}/status` | Ativar/inativar vínculo | Mantém histórico [^2][^1]. |
| `PUT /api/editoras/{id}/dominio` | Configurar domínio próprio | Valida unicidade e integridade [^1]. |
| `GET /resolve-tenant` ou middleware equivalente | Resolver tenant por host | Base para white-label [^1]. |

## Histórias de usuário

### Epic A — Usuários e vínculos

1. Como admin da editora, quero cadastrar usuários internos e parceiros, para montar minha operação editorial.
2. Como admin, quero listar os usuários vinculados à minha editora, para administrar acessos.
3. Como sistema, quero criar um usuário global e um vínculo com a editora, para permitir múltiplos tenants por pessoa.[^1]

### Epic B — Convites e ativação

1. Como admin, quero convidar usuários por e-mail, para reduzir cadastro manual.
2. Como usuário convidado, quero ativar meu acesso ao aceitar o convite.
3. Como sistema, quero rastrear status, expiração e origem do convite.[^1]

### Epic C — Roles e permissões

1. Como admin, quero atribuir roles como GestorEditorial, Autor, Revisor, Capista e Financeiro, para controlar acesso.
2. Como sistema, quero restringir permissões por tenant e vínculo ativo.
3. Como plataforma, quero começar com roles padrão, mas manter a entidade `Role` customizável no futuro.[^2][^1]

### Epic D — Inativação e auditoria

1. Como admin, quero inativar vínculos sem apagar histórico.
2. Como compliance, quero manter trilha de auditoria das ações administrativas.
3. Como sistema, quero preservar integridade histórica de livros, etapas, convites e acessos.[^2][^1]

### Epic E — Domínio próprio

1. Como admin da editora, quero configurar meu domínio próprio.
2. Como sistema, quero validar e associar o domínio ao tenant correto.
3. Como usuário, quero acessar a plataforma pela URL da editora e ver a identidade visual correspondente.[^1]

## Fluxos do slice

### Fluxo 1 — Gestão de usuários

1. Admin acessa a tela de usuários.
2. Sistema lista os usuários vinculados à editora corrente.
3. Admin cadastra novo usuário ou vincula usuário já existente.
4. Sistema grava `Usuario` global e `VinculoUsuarioEditora`.
5. Permissões são atribuídas por vínculo.[^1]

### Fluxo 2 — Convite

1. Admin informa e-mail e roles iniciais.
2. Sistema cria `ConviteUsuario`.
3. Usuário aceita convite.
4. Sistema ativa o vínculo e libera acesso conforme roles.[^1]

### Fluxo 3 — Multi-editora

1. Mesmo usuário recebe vínculo em outra editora.
2. Sistema mantém identidade única.
3. Ao logar, usuário escolhe o contexto ativo.
4. Permissões passam a valer apenas no tenant selecionado.[^1]

### Fluxo 4 — Domínio próprio

1. Admin cadastra domínio personalizado.
2. Sistema valida formato, unicidade e status.
3. DNS aponta para a plataforma.
4. Middleware resolve tenant pelo host.
5. Frontend carrega branding e configurações da editora.[^1]

## Critérios de aceite

- Usuário pode ser criado, convidado, vinculado e inativado sem perda de histórico.
- Roles são atribuídas por vínculo, não por usuário global.
- Listagem sempre respeita `EditoraId`.
- Convites têm status e expiração.
- Domínio próprio é salvo, validado e resolve o tenant corretamente.
- O frontend exibe branding correto quando acessado pelo domínio personalizado.
- Logs e auditoria registram ações críticas.[^2][^1]


## Dependências

- Slice 1 concluído: tenant, onboarding, login e branding base.
- Estrutura de autenticação JWT e refresh token disponível.
- Filtro global por `EditoraId` já implementado.
- Infra de observabilidade e audit log mínima pronta.[^2][^1]


## Estimativa prática

### Tamanho sugerido

- **Médio-alto**, porque o slice mistura autorização, convites, vínculo multi-tenant e white-label.
- Recomendo dividir internamente em 2 subentregas:
    - **Slice 2A:** usuários, convites, roles e permissões.
    - **Slice 2B:** domínio próprio, resolução por host e branding por tenant.


### Ordem recomendada

1. Modelagem de entidades e migrations.
2. APIs de usuários e vínculos.
3. Convites e ativação.
4. Roles e permissões.
5. Domínio próprio e resolução de tenant.
6. Frontend de administração e ajustes de branding.
7. Testes, auditoria e documentação.[^2][^1]

## Observação de arquitetura

O domínio próprio não deve ser tratado como “extra visual”; ele faz parte da identidade operacional da editora e do white-label. Por isso, ele precisa viver junto do slice de identidade e autorização, não apenas no backoffice futuro.[^1]

Posso seguir agora com o próximo passo e montar isso no formato de **tabela de sprint/kanban**, com **tarefas técnicas por backend e frontend** e uma **estimativa de esforço por item**.

<div align="center">⁂</div>

[^1]: Documentacao-de-Arquitetura-SaaS-Editorial-Consolidada-MultiEditora-2.md

[^2]: Backlog-Inicial-Implementacao-SaaS-Editorial.md

