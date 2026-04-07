# Backlog Inicial de Implementação — SaaS Editorial Inteligente

Baseado na arquitetura consolidada do SaaS editorial multi-tenant em .NET 8, Angular e PostgreSQL, este backlog organiza a implementação em épicos, features e histórias priorizadas para entregar valor incremental, reduzir retrabalho e manter coerência com a modelagem de tenants, roles e workflow editorial.[web:291][web:293][web:298]

## Estrutura do backlog

- **Épico**: bloco funcional de negócio ou arquitetura.
- **Feature**: agrupamento implementável dentro do épico.
- **História**: item de valor para usuário ou operação.
- **Prioridade**: P0 = essencial para MVP; P1 = importante após MVP; P2 = evolução.
- **Dependências**: o que precisa existir antes.

## Critérios transversais

Todos os itens do backlog devem respeitar:
- isolamento por tenant com `EditoraId`; 
- autenticação e autorização por role/permissão;
- trilha de auditoria para ações críticas;
- LGPD por padrão para dados pessoais e financeiros; 
- observabilidade mínima, logs estruturados e tratamento de erros. [web:136][web:280][web:298]

## Épico E01 — Fundação técnica e multi-tenant

**Objetivo**: estabelecer a base técnica segura do produto, com isolamento entre editoras e ambiente pronto para evolução incremental. [web:136][web:286]

### Features
- E01-F01. Scaffold backend, frontend e infraestrutura local.
- E01-F02. Multi-tenancy com `EditoraId`.
- E01-F03. Pipeline CI/CD e ambientes.
- E01-F04. Observabilidade inicial.

### Histórias
- **E01-01 (P0)** — Como desenvolvedor, quero criar a solução base com `.NET 8`, `Angular`, `PostgreSQL` e `Docker Compose`, para iniciar o desenvolvimento em ambiente reproduzível.
- **E01-02 (P0)** — Como sistema, quero persistir `EditoraId` nas entidades multi-tenant, para garantir segregação lógica de dados.
- **E01-03 (P0)** — Como backend, quero aplicar filtros globais por tenant no EF Core, para evitar vazamento de dados entre editoras.
- **E01-04 (P0)** — Como plataforma, quero propagar o contexto do tenant na autenticação e nas requisições, para controlar autorização por escopo.
- **E01-05 (P1)** — Como operação, quero health checks, logs estruturados e tracing básico, para facilitar suporte e diagnóstico.
- **E01-06 (P1)** — Como time, quero pipeline de build/test/deploy automatizado, para reduzir falhas manuais.

**Dependências**: nenhuma.

## Épico E02 — Identidade, acesso e onboarding

**Objetivo**: permitir o cadastro da editora, do primeiro administrador e a estrutura inicial de identidade do tenant. [web:136][web:277]

### Features
- E02-F01. Registro da editora.
- E02-F02. Login e refresh token.
- E02-F03. Perfil inicial da editora.
- E02-F04. Auditoria de identidade.

### Histórias
- **E02-01 (P0)** — Como responsável pela editora, quero cadastrar minha editora e meu usuário administrador em um único fluxo, para começar a operar a plataforma.
- **E02-02 (P0)** — Como sistema, quero criar roles padrão no onboarding, para o tenant nascer com configuração mínima utilizável.
- **E02-03 (P0)** — Como usuário administrador, quero fazer login com JWT e refresh token, para acessar o painel com segurança.
- **E02-04 (P0)** — Como administrador, quero configurar logo, cor primária e dados iniciais da editora, para personalizar o ambiente.
- **E02-05 (P1)** — Como usuário, quero recuperar meu acesso por e-mail, para retornar ao sistema sem intervenção manual.
- **E02-06 (P1)** — Como compliance, quero auditar login, troca de senha e ações administrativas, para manter rastreabilidade.

**Dependências**: E01.

## Épico E03 — Gestão de usuários, roles e permissões

**Objetivo**: permitir gestão de atores editoriais e autorização coerente com a operação. [web:293][web:302]

### Features
- E03-F01. CRUD de usuários.
- E03-F02. Convites e ativação.
- E03-F03. Roles e permissões.
- E03-F04. Escopos por livro e etapa.

### Histórias
- **E03-01 (P0)** — Como admin da editora, quero cadastrar usuários internos e parceiros, para montar minha operação editorial.
- **E03-02 (P0)** — Como admin, quero convidar usuários por e-mail, para reduzir cadastro manual.
- **E03-03 (P0)** — Como admin, quero atribuir roles como `GestorEditorial`, `Autor`, `Revisor`, `Capista` e `Financeiro`, para controlar acesso.
- **E03-04 (P0)** — Como sistema, quero restringir acesso por permissões e por `EditoraId`, para manter segurança multi-tenant.
- **E03-05 (P1)** — Como gestor, quero vincular usuários a livros específicos, para limitar visibilidade operacional.
- **E03-06 (P1)** — Como gestor, quero atribuir usuários a etapas específicas, para controlar execução e responsabilidades.
- **E03-07 (P1)** — Como admin, quero inativar usuários sem apagar histórico, para preservar auditoria.

**Dependências**: E02.

## Épico E04 — Catálogo de serviços editoriais

**Objetivo**: modelar os serviços contratáveis e suas regras operacionais. [web:298][web:303]

### Features
- E04-F01. Cadastro de serviços.
- E04-F02. Configuração de preço e SLA.
- E04-F03. Modo de execução humano/IA/híbrido.
- E04-F04. Ativação por tenant.

### Histórias
- **E04-01 (P0)** — Como admin ou gestor editorial, quero cadastrar serviços como revisão, capa, diagramação e ebook, para montar meu catálogo.
- **E04-02 (P0)** — Como editora, quero definir preço base e SLA de cada serviço, para controlar operação e comercial.
- **E04-03 (P0)** — Como editora, quero definir se um serviço pode ser feito por humano, IA ou híbrido, para refletir minha estratégia operacional.
- **E04-04 (P1)** — Como editora, quero ativar ou desativar serviços do catálogo, para adaptar a oferta por fase do negócio.
- **E04-05 (P1)** — Como gestor, quero definir regras de entrega e aprovação por serviço, para padronizar a execução.

**Dependências**: E02.

## Épico E05 — Livros, manuscritos e artefatos

**Objetivo**: permitir o cadastro de livros e o versionamento dos materiais do projeto editorial.

### Features
- E05-F01. CRUD de livros.
- E05-F02. Participantes do livro.
- E05-F03. Upload e versionamento de artefatos.
- E05-F04. Painel do livro.

### Histórias
- **E05-01 (P0)** — Como autor ou gestor editorial, quero cadastrar um livro com seus metadados básicos, para iniciar seu processamento.
- **E05-02 (P0)** — Como usuário autorizado, quero fazer upload do manuscrito e de outros arquivos, para alimentar a esteira editorial.
- **E05-03 (P0)** — Como sistema, quero versionar artefatos por livro e por etapa, para manter histórico e rastreabilidade.
- **E05-04 (P1)** — Como gestor, quero associar participantes ao livro com papel contextual, para organizar o time da obra.
- **E05-05 (P1)** — Como autor, quero visualizar meus arquivos enviados e entregáveis recebidos, para acompanhar a evolução do projeto.

**Dependências**: E03, E04.

## Épico E06 — Contratação de serviços pelo autor

**Objetivo**: permitir que o autor selecione e contrate serviços disponíveis para sua obra.

### Features
- E06-F01. Carrinho/seleção de serviços.
- E06-F02. Geração de contratação.
- E06-F03. Regras por modo de execução.

### Histórias
- **E06-01 (P0)** — Como autor, quero selecionar serviços editoriais para meu livro, para contratar apenas o que preciso.
- **E06-02 (P0)** — Como autor, quero escolher entre execução humana, IA ou híbrida quando disponível, para equilibrar custo, prazo e qualidade.
- **E06-03 (P0)** — Como sistema, quero gerar uma contratação vinculada ao livro e aos serviços escolhidos, para estruturar a cobrança e a execução.
- **E06-04 (P1)** — Como autor, quero visualizar resumo de valor, prazo e modo de entrega antes de confirmar, para contratar com clareza.

**Dependências**: E04, E05.

## Épico E07 — Workflow editorial

**Objetivo**: transformar contratações em etapas executáveis e controladas. [web:303]

### Features
- E07-F01. Geração de etapas.
- E07-F02. Atribuição de responsáveis.
- E07-F03. Controle de status e SLA.
- E07-F04. Aprovação e retrabalho.

### Histórias
- **E07-01 (P0)** — Como sistema, quero criar etapas editoriais a partir dos serviços contratados, para operacionalizar a produção.
- **E07-02 (P0)** — Como gestor editorial, quero atribuir etapas a usuários internos, parceiros ou IA, para iniciar a execução.
- **E07-03 (P0)** — Como executor, quero ver minha fila de etapas atribuídas, para organizar meu trabalho.
- **E07-04 (P0)** — Como sistema, quero controlar status como `Pendente`, `EmAndamento`, `EmRevisao` e `Concluida`, para dar visibilidade do fluxo.
- **E07-05 (P1)** — Como gestor, quero solicitar retrabalho e aprovar entregas, para garantir qualidade.
- **E07-06 (P1)** — Como sistema, quero desbloquear automaticamente etapas dependentes, para acelerar a esteira.

**Dependências**: E05, E06.

## Épico E08 — Chat, colaboração e atendimento

**Objetivo**: centralizar a comunicação contextual no próprio processo editorial.

### Features
- E08-F01. Chat por livro.
- E08-F02. Chat por etapa.
- E08-F03. Anexos e mensagens sistêmicas.
- E08-F04. Tickets de atendimento.

### Histórias
- **E08-01 (P0)** — Como autor, quero conversar com os envolvidos do meu livro dentro da plataforma, para evitar comunicação dispersa.
- **E08-02 (P0)** — Como executor, quero trocar mensagens no contexto da etapa, para alinhar entregas e ajustes.
- **E08-03 (P0)** — Como sistema, quero registrar mensagens e anexos com contexto de livro/etapa, para manter histórico rastreável.
- **E08-04 (P1)** — Como atendimento, quero abrir e acompanhar tickets operacionais, para resolver pendências do autor.
- **E08-05 (P1)** — Como usuário, quero receber notificações de novas mensagens e mudanças importantes, para responder rapidamente.

**Dependências**: E05, E07.

## Épico E09 — Execução por IA

**Objetivo**: permitir automação de etapas com rastreabilidade, custo e governança. [web:305]

### Features
- E09-F01. Configuração de providers/modelos.
- E09-F02. Execução IA por etapa.
- E09-F03. Registro de prompt, custo e tokens.
- E09-F04. Aprovação humana opcional.

### Histórias
- **E09-01 (P1)** — Como editora, quero configurar provedores e modelos de IA por serviço, para controlar custo e capacidade.
- **E09-02 (P1)** — Como sistema, quero executar etapas automatizadas com base na configuração do tenant, para reduzir trabalho manual.
- **E09-03 (P1)** — Como sistema, quero registrar prompt, modelo, custo, tokens e resultado, para auditoria e gestão financeira.
- **E09-04 (P1)** — Como gestor, quero revisar resultados de IA antes da aprovação final, para manter qualidade editorial.
- **E09-05 (P2)** — Como editora, quero definir limites de uso por serviço ou plano, para controlar consumo.

**Dependências**: E04, E07.

## Épico E10 — Pagamentos, repasse e extrato

**Objetivo**: viabilizar a monetização da plataforma e a operação financeira dos serviços. [web:136][web:298]

### Features
- E10-F01. Cobrança do autor.
- E10-F02. Webhooks e conciliação.
- E10-F03. Repasse a parceiros.
- E10-F04. Extratos por visão.

### Histórias
- **E10-01 (P1)** — Como autor, quero pagar os serviços contratados pela plataforma, para liberar a execução.
- **E10-02 (P1)** — Como sistema, quero processar webhooks do gateway e atualizar o status dos pagamentos, para manter conciliação correta.
- **E10-03 (P1)** — Como editora, quero operar com repasse direto ao parceiro ou saque integral, para adequar meu modelo financeiro.
- **E10-04 (P1)** — Como parceiro, quero visualizar valores a receber e status de repasse, para acompanhar minha remuneração.
- **E10-05 (P1)** — Como financeiro, quero extratos por livro, serviço, parceiro e período, para controle operacional.

**Dependências**: E06, E07.

## Épico E11 — Integração ERP

**Objetivo**: conectar a plataforma aos sistemas administrativos da editora.

### Features
- E11-F01. Interface `IProviderErp`.
- E11-F02. Envio de produtos/serviços.
- E11-F03. Sincronização financeira básica.

### Histórias
- **E11-01 (P2)** — Como editora, quero integrar produtos e serviços ao meu ERP, para evitar retrabalho administrativo.
- **E11-02 (P2)** — Como financeiro, quero sincronizar eventos relevantes de cobrança e repasse, para melhorar conciliação.
- **E11-03 (P2)** — Como sistema, quero suportar múltiplos provedores ERP por abstração, para reduzir acoplamento.

**Dependências**: E10.

## Épico E12 — LGPD, auditoria e governança

**Objetivo**: garantir conformidade no tratamento de dados pessoais e financeiros. [web:280][web:288]

### Features
- E12-F01. Consentimento e base legal.
- E12-F02. Audit log.
- E12-F03. Direitos do titular.
- E12-F04. Retenção e anonimização.

### Histórias
- **E12-01 (P0)** — Como sistema, quero registrar consentimentos e versões de termo, para suportar tratamento lícito de dados.
- **E12-02 (P0)** — Como plataforma, quero registrar ações críticas em `AuditLog`, para suportar rastreabilidade e investigação.
- **E12-03 (P1)** — Como titular de dados, quero solicitar acesso, correção ou exclusão dos meus dados, para exercer meus direitos.
- **E12-04 (P1)** — Como compliance, quero definir políticas de retenção e anonimização, para reduzir risco regulatório.
- **E12-05 (P1)** — Como segurança, quero segregar dados financeiros e minimizar exposição de PII, para atender princípios de minimização e necessidade.

**Dependências**: E02, E03, E10.

## Épico E13 — Backoffice, suporte e operação

**Objetivo**: preparar a plataforma para operação contínua e suporte a múltiplos tenants.

### Features
- E13-F01. Painel operacional.
- E13-F02. Monitoramento por tenant.
- E13-F03. Gestão de incidentes e suporte.

### Histórias
- **E13-01 (P2)** — Como operação, quero visualizar tenants, volumes, erros e filas, para agir proativamente.
- **E13-02 (P2)** — Como suporte, quero pesquisar eventos por tenant, livro ou usuário, para acelerar atendimento.
- **E13-03 (P2)** — Como plataforma, quero centralizar métricas e alertas, para melhorar confiabilidade.

**Dependências**: E01, E12.

## Ordem recomendada de implementação

### MVP técnico-operacional
1. E01 — Fundação técnica e multi-tenant.
2. E02 — Identidade e onboarding.
3. E03 — Usuários, roles e permissões.
4. E04 — Catálogo de serviços.
5. E05 — Livros e artefatos.
6. E06 — Contratação de serviços.
7. E07 — Workflow editorial.
8. E08 — Chat básico.
9. E12 — LGPD e auditoria mínima. [web:296][web:302]

### Pós-MVP imediato
10. E09 — Execução por IA.
11. E10 — Pagamentos, repasse e extrato.
12. E11 — Integração ERP.
13. E13 — Backoffice e operação.

## Slices sugeridos para as primeiras entregas

| Slice | Objetivo | Itens principais |
|---|---|---|
| Slice 1 | Tenant e admin inicial | E01-01 a E01-04, E02-01 a E02-04 |
| Slice 2 | Usuários e permissões | E03-01 a E03-04 |
| Slice 3 | Catálogo e livros | E04-01 a E04-03, E05-01 a E05-03 |
| Slice 4 | Contratação e workflow | E06-01 a E06-03, E07-01 a E07-04 |
| Slice 5 | Chat e compliance mínimo | E08-01 a E08-03, E12-01 a E12-02 |
| Slice 6 | IA assistida | E09-01 a E09-04 |
| Slice 7 | Pagamentos e repasses | E10-01 a E10-05 |

## Critérios de pronto por história

Uma história só deve ser considerada pronta quando incluir:
- backend implementado com testes mínimos;
- frontend funcional para o fluxo correspondente;
- validação de permissão por role/tenant;
- logs e tratamento de erros;
- atualização da documentação técnica;
- critérios de aceite validados manualmente. [web:298][web:303]

## Recomendações práticas

- Comece com **roles padrão fixas**, mas modele `Role` como entidade para permitir customização futura. [web:302]
- Não deixe pagamentos antes do workflow mínimo, para evitar monetizar um fluxo ainda instável. [web:296]
- Não deixe permissões “para depois”, porque retrofitar autorização em SaaS multi-tenant aumenta risco e retrabalho. [web:302]
- Mantenha IA como extensão do workflow, não como domínio isolado sem vínculo de etapa, custo e auditoria. [web:305]
