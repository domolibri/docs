<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Considerando a arquitetura e o backlog inicial faça o planejamento do slice 2. Inclua nesse planejamento o suporte ao uso de domínio próprio pela editora

A seguir está um planejamento objetivo do **Slice 2**, considerando a arquitetura consolidada e o backlog inicial, com inclusão explícita de **domínio próprio da editora**. O foco é fechar a base de identidade e autorização para que o resto da plataforma já nasça multi-tenant, segura e preparada para white-label.[^1][^2]

## Objetivo do slice

O Slice 2 deve consolidar a **gestão de usuários, roles, permissões e vínculos por editora**, permitindo que um mesmo usuário participe de múltiplas editoras com contextos separados. Isso é coerente com a arquitetura, que separa identidade global de usuário do vínculo ativo com cada tenant, e com o backlog, que define o pico E03 como “Gesto de usurios, roles e permisses” após o onboarding inicial.[^2][^1]

Também é o slice adequado para preparar a plataforma para personalização por editora, porque o modelo de arquitetura já prevê `Editora` com `Slug`, `LogoUrl`, `CorPrimaria`, `DominioCustomizado`, além de configurações operacionais e de branding por tenant.[^2]

## Escopo funcional

O slice deve entregar os itens P0 do pico E03: cadastro de usuários internos e parceiros, convites por e-mail, roles editoriais iniciais e controle de acesso por `EditoraId` e por vínculo ativo.[^1][^2]

### Entidades envolvidas

- `Usuario`.
- `VinculoUsuarioEditora`.
- `Role`.
- `Permissao`.
- `UsuarioRole`.
- `ConviteUsuario`.
- `Editora` com suporte a `DominioCustomizado`, `Slug`, `LogoUrl` e `CorPrimaria`.[^2]


### APIs principais

- `GET /api/usuarios`.
- `POST /api/usuarios`.
- `POST /api/usuarios/convites`.
- `POST /api/usuarios/{id}/vinculos`.
- `PUT /api/vinculos/{id}/roles`.
- `PUT /api/vinculos/{id}/status`.[^2]


### Fluxos cobertos

- Listar usuários do tenant corrente.
- Cadastrar usuário global e vínculo com a editora.
- Convidar usuário já existente ou novo.
- Atribuir roles por vínculo, não por usuário global.
- Inativar vínculo sem apagar histórico.
- Selecionar o contexto ativo da editora quando o usuário tiver mais de um vínculo.[^2]


## Suporte a domínio próprio

O suporte a domínio próprio deve entrar no Slice 2 porque ele faz parte da identidade da editora e impacta login, branding e acesso à plataforma. A arquitetura já prevê `DominioCustomizado` na entidade `Editora`, então este slice deve transformar isso em comportamento funcional.[^2]

### O que implementar

- Cadastro e edição do domínio próprio da editora.
- Validação de formato e unicidade do domínio.
- Associação do domínio ao tenant correto.
- Resolução do tenant pela URL de acesso.
- Suporte a subdomínio padrão e domínio personalizado.
- Regras de segurança para evitar sequestro de domínio.
- Exibição do branding correto ao entrar por domínio próprio.[^2]


### Fluxo sugerido

1. Admin configura `DominioCustomizado` no painel da editora.
2. O sistema valida o domínio e marca como pendente ou ativo.
3. O DNS aponta para o ambiente da plataforma.
4. Na primeira requisição, a aplicação resolve o tenant pelo host.
5. O frontend carrega logo, cor primária e identidade visual da editora correta.[^2]

### Requisitos técnicos mínimos

- Persistir o domínio no cadastro da editora.
- Criar serviço de resolução por host.
- Criar middleware ou componente equivalente para identificar o tenant pelo domínio.
- Manter fallback para o domínio padrão da plataforma.
- Registrar eventos de alteração em `AuditLog`.[^2]


## Planejamento das histórias

### H1. Cadastro e listagem de usuários

- Como admin da editora, quero cadastrar usuários internos e parceiros, para montar minha operação editorial.
- Como admin, quero listar usuários vinculados ao tenant corrente, para administrar acessos.
- Como sistema, quero criar `Usuario` global e `VinculoUsuarioEditora`, para permitir múltiplas editoras por usuário.[^1][^2]


### H2. Convites e ativação

- Como admin, quero convidar usuários por e-mail, para reduzir cadastro manual.
- Como usuário convidado, quero ativar meu acesso ao aceitar o convite.
- Como sistema, quero rastrear convite, expiração e status, para manter controle operacional.[^1][^2]


### H3. Roles e permissões por vínculo

- Como admin, quero atribuir roles editoriais por vínculo.
- Como sistema, quero restringir acesso por `EditoraId` e por contexto ativo.
- Como plataforma, quero iniciar com roles padrão, mas manter `Role` como entidade para customização futura.[^1][^2]


### H4. Status e inativação

- Como admin, quero inativar usuários sem excluir histórico.
- Como compliance, quero manter trilha de auditoria das mudanças de acesso.
- Como sistema, quero preservar vínculos e registros passados para rastreabilidade.[^1][^2]


### H5. Domínio próprio

- Como admin da editora, quero configurar meu domínio próprio.
- Como sistema, quero resolver a editora pelo host acessado.
- Como plataforma, quero exibir identidade visual e configurações corretas quando o acesso vier pelo domínio personalizado.[^2]


## APIs e comportamento esperado

| API | Responsabilidade | Observações |
| :-- | :-- | :-- |
| `GET /api/usuarios` | Lista usuários do tenant | Deve filtrar por `EditoraId` ativo [^2]. |
| `POST /api/usuarios` | Cria usuário e vínculo | Pode criar identidade global ou reaproveitar usuário existente [^2]. |
| `POST /api/usuarios/convites` | Gera convite | Deve suportar novo e-mail ou usuário já cadastrado [^2]. |
| `POST /api/usuarios/{id}/vinculos` | Cria novo vínculo | Essencial para múltiplas editoras por usuário [^2]. |
| `PUT /api/vinculos/{id}/roles` | Atribui roles | Roles devem ser ligadas ao vínculo, não ao usuário [^2]. |
| `PUT /api/vinculos/{id}/status` | Ativa/inativa vínculo | Preserva histórico e auditoria [^1][^2]. |
| `PUT /api/editoras/{id}/dominio` | Configura domínio próprio | Deve validar unicidade e status de verificação [^2]. |
| `GET /resolve-tenant` ou equivalente | Resolve tenant por host | Deve alimentar o contexto do tenant no backend e frontend [^2]. |

## Critérios de pronto

Uma história só deve ser considerada pronta quando incluir:

- Backend implementado com testes mínimos.
- Frontend funcional para o fluxo correspondente.
- Validação de permissão por role e tenant.
- Logs e tratamento de erros.
- Atualização da documentação técnica.
- Aceite manual validado.[^1]

Para o item de domínio próprio, eu acrescentaria ainda:

- Resolução correta por host.
- Fallback para domínio padrão.
- Persistência da configuração no tenant.
- Auditoria da alteração.
- Carregamento do branding correto.


## Dependências e ordem interna

A ordem recomendada dentro do slice é:

1. Listagem e cadastro de usuários.
2. Convites e ativação.
3. Roles e permissões por vínculo.
4. Status e inativação.
5. Domínio próprio e resolução por host.
6. Ajustes de branding e contexto ativo da editora.[^1][^2]

Essa ordem reduz retrabalho porque o domínio próprio depende do tenant já existir e o controle de acesso depende do modelo de vínculo estar consolidado. Também mantém alinhamento com a recomendação do backlog de não deixar permissões para depois em um SaaS multi-tenant.[^1]

## Entrega esperada

Ao final do Slice 2, a plataforma deve permitir que uma editora:

- administre seus usuários;
- convide colaboradores e parceiros;
- defina permissões por papel e vínculo;
- opere com múltiplos vínculos por usuário;
- acesse a plataforma por domínio próprio com identidade visual da própria marca.[^2]

Isso fecha a base do tenant para os próximos slices, especialmente catálogo de serviços e livros, sem expor a aplicação a retrabalho de autorização ou de white-label depois.[^1][^2]

Posso transformar esse planejamento em um **backlog executivo do Slice 2** com épicos, histórias, critérios de aceite e esforço estimado.

<div align="center">⁂</div>

[^1]: Backlog-Inicial-Implementacao-SaaS-Editorial.md

[^2]: Documentacao-de-Arquitetura-SaaS-Editorial-Consolidada-MultiEditora-2.md

