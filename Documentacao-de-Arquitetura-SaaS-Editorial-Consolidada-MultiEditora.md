<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

## Documentação de Arquitetura: SaaS Editorial Inteligente

### 1. Visão Geral

Sistema multi-tenant white-label para gestão completa da cadeia editorial, conectando autores, editoras, prestadores de serviço e IA. Suporta fluxos paralelos de revisão, captação e produção, com automação via LLMs e integração ERP/pagamentos.[^1][^2][^3]

**Princípios chave**:

- Isolamento lógico via `EditoraId` (tenant ID), com identidade global de usuário e vínculo separado por editora.
- Conformidade LGPD: consentimento granular, DPO virtual, logs de auditoria, anonimização de dados sensíveis.[^4][^5]
- Escalabilidade horizontal para picos de processamento IA.

***

## 2. Principais Entidades

### Domínio **Core (Tenant, Identidade e Configuração)**

```text
Editora (Tenant)
├── Id (Guid)
├── Nome, Slug, LogoUrl, CorPrimaria
├── DominioCustomizado
├── Status (Ativa, Suspensa, Trial, Cancelada)
├── ConfiguracoesGerais (JSON)
├── ConfiguracoesIA (JSON)
├── ConfiguracoesFinanceiras (JSON)
├── ConfiguracoesErp (JSON)
├── VinculosUsuarios (1:N)
├── Livros (1:N)
└── Planos/Assinatura
```

```text
Usuario
├── Id
├── Nome, Email, Telefone
├── DocumentoFiscal (opcional)
├── TipoPessoa (Fisica, Juridica, Sistema)
├── StatusConta (Ativa, Inativa, Bloqueada)
├── AuthProvider (Local, Google, Microsoft, etc.)
├── UltimoAcessoEm
├── VinculosEditora (1:N)
└── PreferenciasGerais
```

```text
Role
├── Id
├── EditoraId
├── Nome (AdminEditora, GestorEditorial, etc.)
├── Descricao
├── Escopo (Tenant, Livro, Etapa, Financeiro, Suporte)
└── Permissoes (N:N)
```

```text
Permissao
├── Id
├── Codigo
├── Nome
├── Descricao
└── Agrupamento (Usuarios, Livros, Workflow, Financeiro, IA, Suporte)
```

```text
VinculoUsuarioEditora
├── Id
├── UsuarioId
├── EditoraId
├── TipoVinculo (Interno, Freelancer, Autor, Parceiro, Sistema)
├── Status (Ativo, Inativo, PendenteConvite, Bloqueado)
├── DataEntrada
├── DataSaida
├── OrigemVinculo (Onboarding, Convite, AutoCadastro, Importacao)
├── Roles (N:N)
├── PermissoesGranulares
└── ParticipacoesEmLivros/Etapas

UsuarioRole
├── VinculoUsuarioEditoraId
├── RoleId
└── DataAtribuicao
```

```text
ConviteUsuario
├── Id
├── EditoraId
├── Email
├── UsuarioId (opcional, se já existir cadastro global)
├── RolesIniciais
├── TokenConvite
├── ExpiraEm
└── Status
```

### Domínio **Catálogo de Serviços Editoriais**

```text
ServicoEditorial
├── Id
├── EditoraId
├── Nome
├── Tipo (Plagio, RevisaoLiteraria, RevisaoOrtografica, Capa, Ilustracao, Diagramacao, Ebook, Impressao, Marketing)
├── ModoExecucaoPermitido (Humano, IA, Hibrido)
├── PrecoBase
├── Ativo
└── RegrasDeEntrega (JSON)
```

```text
PlanoServicoIA
├── Id
├── EditoraId
├── ServicoEditorialId
├── Provider (OpenAI, AzureOpenAI, Outro)
├── Modelo
├── LimitesDeUso
├── Parametros (JSON)
└── Ativo
```

### Domínio **Obra, Livro e Produção Editorial**

```text
Livro
├── Id
├── EditoraId
├── AutorResponsavelId
├── Titulo, Subtitulo, ISBN
├── Status (Rascunho, EmAvaliacao, EmProducao, Publicado, Arquivado)
├── MetadadosComerciaisEFiscais
├── Artefatos (1:N)
├── EtapasWorkflow (1:N)
├── Participantes (1:N)
├── ContratacoesServicos (1:N)
└── Pagamentos (1:N)
```

```text
LivroParticipante
├── Id
├── LivroId
├── VinculoUsuarioEditoraId
├── PapelNoLivro (Autor, Coautor, Revisor, Capista, Ilustrador, Diagramador, Gestor, Atendimento)
├── EscopoDeAcesso
└── Ativo
```

```text
Artefato
├── Id
├── LivroId
├── WorkflowEtapaId
├── Tipo (Manuscrito, Capa, Ilustracao, PDFMiolo, EbookEPUB, MaterialMarketing, Comprovante, Outro)
├── NomeArquivo
├── StorageProvider
├── CaminhoStorage
├── HashArquivo
├── MimeType
└── Versoes (1:N)
```

```text
VersaoArtefato
├── Id
├── ArtefatoId
├── NumeroVersao
├── CriadoPorUsuarioId
├── Origem (Humano, IA, Sistema)
├── Observacao
├── DataCriacao
└── Metadados (JSON)
```

### Domínio **Workflow Editorial e Execução**

```text
WorkflowEtapa
├── Id
├── LivroId
├── ServicoEditorialId
├── TipoEtapa
├── Status (Pendente, Disponivel, EmAndamento, EmRevisao, AguardandoAprovacao, Concluida, Cancelada)
├── ModoExecucao (Humano, IA, Hibrido)
├── ResponsavelUsuarioId (opcional)
├── ExecutorIaId (opcional)
├── Ordem
├── DataInicio, DataLimite, DataConclusao
├── ArtefatoEntradaId
├── ArtefatoSaidaId
└── ChatMensagens (1:N)
```

```text
AtribuicaoEtapa
├── Id
├── WorkflowEtapaId
├── VinculoUsuarioEditoraId
├── PapelExecucao
├── DataAtribuicao
└── Status
```

```text
ExecucaoIA
├── Id
├── EditoraId
├── LivroId
├── WorkflowEtapaId
├── Provider
├── Modelo
├── TipoOperacao
├── PromptReferencia
├── TokensEntrada, TokensSaida
├── CustoEstimado
├── Status
└── ResultadoResumo
```

### Domínio **Chat, Atendimento e Colaboração**

```text
ChatMensagem
├── Id
├── EditoraId
├── LivroId
├── WorkflowEtapaId
├── RemetenteUsuarioId (opcional)
├── RemetenteSistema (Humano, IA, Sistema)
├── Conteudo
├── TipoMensagem (Texto, Sistema, Anexo, Notificacao)
├── DataEnvio
└── Anexos (1:N)
```

```text
ChatAnexo
├── Id
├── ChatMensagemId
├── ArtefatoId (opcional)
├── NomeArquivo
└── UrlTemporaria
```

```text
TicketSuporte
├── Id
├── EditoraId
├── LivroId (opcional)
├── SolicitanteUsuarioId
├── ResponsavelUsuarioId
├── Categoria
├── Status
├── Prioridade
└── Historico
```

### Domínio **Contratação, Pagamentos e Repasse**

```text
ContratacaoServico
├── Id
├── EditoraId
├── LivroId
├── ServicoEditorialId
├── ContratanteUsuarioId
├── ModoExecucaoEscolhido (Humano, IA, Hibrido)
├── ValorContratado
├── Status
├── DataContratacao
└── ItensFinanceiros
```

```text
Pagamento
├── Id
├── EditoraId
├── LivroId
├── ContratacaoServicoId
├── Valor
├── Moeda
├── Status (Pendente, Autorizado, Pago, Estornado, Cancelado)
├── Gateway
├── ReferenciaExterna
└── DataPagamento
```

```text
RepasseParceiro
├── Id
├── EditoraId
├── VinculoUsuarioEditoraId
├── ContratacaoServicoId
├── WorkflowEtapaId
├── ValorBruto
├── TaxaPlataforma
├── ValorLiquido
├── ModeloRepasse (DiretoParceiro, SaqueIntegralEditora)
├── Status
└── DataRepasse
```

```text
ExtratoFinanceiro
├── Id
├── EditoraId
├── Periodo
├── Tipo (Editora, Parceiro, Livro, Servico)
└── Detalhamento (JSON)
```

### Domínio **LGPD, Auditoria e Segurança**

```text
ConsentimentoLGPD
├── Id
├── EditoraId
├── UsuarioId
├── VinculoUsuarioEditoraId (opcional)
├── TipoConsentimento
├── BaseLegal
├── VersaoTermo
├── DataConsentimento
└── RevogadoEm
```

```text
SolicitacaoTitularLGPD
├── Id
├── EditoraId
├── UsuarioId
├── VinculoUsuarioEditoraId (opcional)
├── TipoSolicitacao (Acesso, Correcao, Exclusao, Portabilidade, Revogacao)
├── Status
├── DataSolicitacao
└── DataConclusao
```

```text
AuditLog
├── Id
├── EditoraId
├── UsuarioId
├── VinculoUsuarioEditoraId (opcional)
├── Acao
├── Recurso
├── RecursoId
├── IP
├── UserAgent
├── DataHora
└── DadosAnonimizados
```

### **Roles iniciais sugeridas**

- `AdminEditora`: administração completa do tenant, branding, usuários, integrações, permissões e configurações financeiras.
- `GestorEditorial`: gestão operacional da esteira editorial, distribuição de tarefas, validação de entregas e acompanhamento do livro.
- `Autor`: contratação de serviços, envio de artefatos, acompanhamento do fluxo, aprovação de entregas e pagamentos.
- `RevisorLiterario`: atuação em conteúdo, clareza, estilo e estrutura narrativa.
- `RevisorOrtografico`: atuação em gramática, ortografia, padronização e normalização textual.
- `Capista`: criação e revisão de capas e criativos visuais.
- `Ilustrador`: produção de ilustrações internas, externas e peças gráficas complementares.
- `Diagramador`: preparação de miolo, PDF final, ebook e arquivos para impressão.
- `Marketing`: criação e acompanhamento de campanhas e criativos promocionais.
- `Financeiro`: gestão de cobrança, extratos, repasses, saques e conciliação.
- `Atendimento`: comunicação operacional, suporte ao autor e acompanhamento de pendências.
- `FreelancerParceiro`: executor externo com acesso restrito às etapas em que foi alocado.
- `AgenteIA`: executor sistêmico auditável para etapas automatizadas, sem autenticação humana direta.

***

## 3. Domínios e Contextos Delimitados (DDD Bounded Contexts)

### **Bounded Context 1: Tenant, Identidade e Acesso**

Responsável pela gestão da editora como tenant, identidade dos usuários, autenticação, autorização, convites, roles e permissões.

**Principais entidades**:
- `Editora`
- `Usuario`
- `VinculoUsuarioEditora`
- `Role`
- `Permissao`
- `UsuarioRole`
- `ConviteUsuario`

**Responsabilidades**:
- Cadastro da editora e do primeiro administrador
- Login, refresh token, recuperação de acesso
- Cadastro global de identidade de usuário
- Vínculo de um mesmo usuário com múltiplas editoras
- Convite e gestão de usuários por tenant
- Matriz de permissões por tenant e por vínculo
- Segregação de acesso por `EditoraId` e contexto de vínculo ativo

**APIs principais**:
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/refresh`
- `GET /api/usuarios`
- `POST /api/usuarios`
- `POST /api/usuarios/convites`
- `POST /api/usuarios/{id}/vinculos`
- `PUT /api/vinculos/{id}/roles`
- `PUT /api/vinculos/{id}/status`

---

### **Bounded Context 2: Catálogo de Serviços e Configuração Operacional**

Responsável por definir quais serviços editoriais a editora oferece, como cada serviço é executado e quais políticas comerciais, operacionais e de IA se aplicam.

**Principais entidades**:
- `ServicoEditorial`
- `PlanoServicoIA`
- `ConfiguracoesIA`
- `ConfiguracoesFinanceiras`
- `ConfiguracoesErp`

**Responsabilidades**:
- Cadastro de serviços disponíveis
- Definição do modo de execução: humano, IA ou híbrido
- Configuração de preço base e regras de entrega
- Habilitação ou bloqueio de serviços por tenant
- Parametrização de providers de IA e ERP

**APIs principais**:
- `GET /api/servicos`
- `POST /api/servicos`
- `PUT /api/servicos/{id}`
- `PUT /api/editoras/me/config/ia`
- `PUT /api/editoras/me/config/financeiro`
- `PUT /api/editoras/me/config/erp`

---

### **Bounded Context 3: Obras, Livros e Participantes**

Responsável pelo ciclo de vida do livro, metadados editoriais, vínculo com autor e associação dos participantes que atuarão no projeto.

**Principais entidades**:
- `Livro`
- `LivroParticipante`
- `Artefato`
- `VersaoArtefato`

**Responsabilidades**:
- Cadastro e edição de livros
- Upload e versionamento de manuscritos e arquivos
- Associação de autor, coautor e colaboradores ao livro
- Organização dos artefatos por livro e por etapa
- Disponibilização da visão consolidada do projeto editorial

**APIs principais**:
- `POST /api/livros`
- `GET /api/livros`
- `GET /api/livros/{id}`
- `PUT /api/livros/{id}`
- `POST /api/livros/{id}/participantes`
- `POST /api/artefatos`
- `GET /api/livros/{id}/artefatos`

---

### **Bounded Context 4: Workflow Editorial e Execução**

Responsável pela esteira editorial, criação e execução das etapas, atribuição de responsáveis, controle de status, SLAs e aceite das entregas.

**Principais entidades**:
- `WorkflowEtapa`
- `AtribuicaoEtapa`
- `ServicoEditorial`
- `ExecucaoIA`

**Responsabilidades**:
- Instanciação da esteira editorial por livro
- Criação e ordenação das etapas
- Atribuição para humano, parceiro ou IA
- Controle de SLA, bloqueios e dependências
- Aprovação, revisão e encerramento das etapas

**APIs principais**:
- `GET /api/livros/{id}/etapas`
- `POST /api/livros/{id}/etapas`
- `PUT /api/etapas/{id}`
- `POST /api/etapas/{id}/atribuir`
- `POST /api/etapas/{id}/iniciar`
- `POST /api/etapas/{id}/concluir`

---

### **Bounded Context 5: Serviços de IA e Automação**

Responsável pela execução automatizada de tarefas editoriais e registro auditável das interações com modelos de IA.

**Principais entidades**:
- `ExecucaoIA`
- `PlanoServicoIA`
- `VersaoArtefato`

**Responsabilidades**:
- Orquestração de chamadas a LLMs e outros modelos
- Registro de prompts, custos, tokens e resultados
- Geração de artefatos automatizados
- Controle de políticas por tenant
- Suporte a execução síncrona e assíncrona

**APIs principais**:
- `POST /api/ia/executar/revisao-literaria`
- `POST /api/ia/executar/revisao-ortografica`
- `POST /api/ia/executar/capa`
- `POST /api/ia/executar/marketing`
- `GET /api/ia/execucoes/{id}`

---

### **Bounded Context 6: Chat, Atendimento e Colaboração**

Responsável pela comunicação entre autor, editora, parceiros e sistema ao longo do processo editorial.

**Principais entidades**:
- `ChatMensagem`
- `ChatAnexo`
- `TicketSuporte`

**Responsabilidades**:
- Chat por livro e por etapa
- Histórico persistido de mensagens
- Envio de anexos e referências
- Mensagens sistêmicas e notificações operacionais
- Atendimento e suporte

**APIs principais**:
- `GET /api/chat/livros/{livroId}`
- `GET /api/chat/etapas/{etapaId}`
- `POST /api/chat/mensagens`
- `POST /api/tickets`
- `PUT /api/tickets/{id}`

**Canal em tempo real**:
- `WebSocket /ws/chat/{livroId}`
- `WebSocket /ws/chat/etapa/{etapaId}`

---

### **Bounded Context 7: Contratação, Pagamentos e Repasse**

Responsável pela contratação de serviços, cobrança do autor, repasse a parceiros e geração de extratos financeiros.

**Principais entidades**:
- `ContratacaoServico`
- `Pagamento`
- `RepasseParceiro`
- `ExtratoFinanceiro`

**Responsabilidades**:
- Contratação de serviços avulsos ou em pacote
- Processamento de pagamentos
- Geração de repasses e comissionamentos
- Regras de saque da editora
- Conciliação com gateway e ERP

**APIs principais**:
- `POST /api/contratacoes`
- `POST /api/pagamentos`
- `POST /api/pagamentos/webhooks`
- `GET /api/extratos/editora`
- `GET /api/extratos/parceiros`
- `POST /api/repasses/processar`

---

### **Bounded Context 8: LGPD, Auditoria e Governança**

Responsável pelo ciclo de compliance, consentimento, retenção, rastreabilidade e atendimento aos direitos do titular.

**Principais entidades**:
- `ConsentimentoLGPD`
- `SolicitacaoTitularLGPD`
- `AuditLog`

**Responsabilidades**:
- Registro de consentimento e base legal
- Auditoria de ações críticas
- Retenção e anonimização de dados
- Exportação e exclusão de dados
- Atendimento de solicitações LGPD

**APIs principais**:
- `POST /api/lgpd/consentimentos`
- `POST /api/lgpd/solicitacoes`
- `GET /api/lgpd/solicitacoes/{id}`
- `GET /api/auditoria/logs`

***

## 4. Fluxos de Dados e Ações de Usuário

### **Fluxo 1: Onboarding da Editora e Criação do Tenant**

```text
1. Usuário responsável acessa a plataforma e preenche o cadastro inicial da editora.
2. POST /api/auth/register cria:
   ├── Editora
   ├── Usuario (identidade global do primeiro administrador)
   ├── VinculoUsuarioEditora do administrador
   ├── Roles iniciais padrão
   └── Configurações iniciais do tenant
3. Usuário realiza login.
4. A autenticação gera JWT com UsuarioId global e lista de vínculos/editoras disponíveis.
5. O usuário seleciona a editora/contexto ativo quando possuir mais de um vínculo.
6. Frontend carrega dashboard inicial, branding e configurações básicas da editora selecionada.
7. Admin conclui setup inicial: logo, cor primária, serviços habilitados, dados financeiros e preferências operacionais.
```

---

### **Fluxo 2: Cadastro e Gestão de Usuários da Editora**

```text
1. Admin acessa Gestão de Usuários da editora corrente.
2. GET /api/usuarios retorna os usuários vinculados ao tenant corrente e seus respectivos vínculos.
3. Admin cadastra ou convida novos usuários.
4. Se o e-mail ainda não existir, POST /api/usuarios cria um Usuario global e um VinculoUsuarioEditora.
5. Se o usuário já existir na plataforma, POST /api/usuarios/{id}/vinculos ou POST /api/usuarios/convites cria apenas um novo vínculo com a editora.
6. Admin atribui roles e permissões no nível do vínculo, não no nível global do usuário.
7. Usuário convidado ativa a conta e passa a acessar os módulos compatíveis com o vínculo e a editora selecionada.
8. Todas as alterações críticas são registradas em AuditLog.
```

---

### **Fluxo 3: Configuração dos Serviços Editoriais**

```text
1. Admin ou Gestor Editorial acessa Catálogo de Serviços.
2. Cadastra ou ativa serviços como revisão, capa, diagramação, ebook, impressão e marketing.
3. Define se cada serviço poderá ser executado por:
   ├── Humano
   ├── IA
   └── Modo híbrido
4. Configura preço, SLA, regras de entrega e políticas de aprovação.
5. Configura integrações de IA e ERP vinculadas ao tenant.
```

---

### **Fluxo 4: Cadastro do Livro e Associação dos Participantes**

```text
1. Autor ou Gestor Editorial cria um novo livro.
2. POST /api/livros registra o livro com metadados básicos e vínculo com a editora.
3. Upload inicial do manuscrito é realizado em /api/artefatos.
4. Participantes do livro são associados por meio do VinculoUsuarioEditora:
   ├── Autor
   ├── Revisor
   ├── Capista
   ├── Diagramador
   ├── Atendimento
   └── Outros envolvidos
5. O livro passa a ter visão consolidada de participantes, artefatos e serviços contratados.
```

---

### **Fluxo 5: Contratação de Serviços pelo Autor**

```text
1. Autor acessa o catálogo de serviços disponíveis para seu livro.
2. Seleciona um ou mais serviços editoriais.
3. Escolhe, quando permitido, o modo de execução:
   ├── Humano
   ├── IA
   └── Híbrido
4. O sistema gera a contratação e calcula o valor.
5. O pagamento é iniciado via gateway.
6. Após confirmação, o sistema libera a criação ou ativação das etapas correspondentes no workflow.
```

---

### **Fluxo 6: Execução do Workflow Editorial**

```text
1. O sistema instancia a esteira editorial do livro.
2. Cada etapa é criada com tipo, ordem, prazo, executor e status.
3. Uma etapa pode ser:
   ├── atribuída a colaborador interno,
   ├── atribuída a parceiro/freelancer com vínculo ativo naquela editora,
   └── executada por IA.
4. O executor recebe notificação e acessa sua fila de trabalho no contexto da editora correspondente.
5. Durante a execução, são enviados artefatos, comentários, revisões e mensagens.
6. A etapa pode seguir para:
   ├── revisão,
   ├── aprovação,
   ├── retrabalho,
   └── conclusão.
7. A conclusão de uma etapa pode desbloquear a próxima automaticamente.
```

---

### **Fluxo 7: Execução Automatizada por IA**

```text
1. Uma etapa configurada para IA é iniciada.
2. O sistema consulta a configuração do tenant para o serviço correspondente.
3. O provider/modelo é selecionado com base em PlanoServicoIA.
4. O sistema envia prompt, contexto e artefatos de entrada.
5. A execução gera:
   ├── resultado estruturado,
   ├── artefato gerado,
   ├── metadados de custo/tokens,
   └── trilha auditável.
6. O resultado pode seguir direto para aprovação ou para revisão humana complementar.
```

---

### **Fluxo 8: Chat, Atendimento e Colaboração**

```text
1. Cada livro e cada etapa possuem canal contextual de comunicação.
2. Usuários autorizados trocam mensagens, anexos e observações.
3. O sistema registra mensagens humanas, sistêmicas e de IA.
4. Anexos enviados no chat podem ser vinculados a artefatos do livro.
5. Atendimento acompanha dúvidas, pendências e solicitações do autor.
6. Todo histórico permanece acessível conforme escopo de permissão do usuário.
```

---

### **Fluxo 9: Pagamentos, Repasse e Extrato**

```text
1. Autor contrata e paga pelos serviços.
2. Gateway confirma o pagamento via webhook.
3. O sistema marca a contratação como paga e libera ou mantém a execução das etapas.
4. Ao concluir serviços executados por parceiros, gera-se o item financeiro de repasse.
5. O tenant pode operar em dois modos:
   ├── repasse direto ao parceiro/freelancer,
   └── saque integral pela editora, que faz o pagamento externamente.
6. Extratos são consolidados por editora, parceiro, livro, serviço e período.
7. Eventos financeiros relevantes podem ser integrados ao ERP.
```

---

### **Fluxo 10: LGPD, Auditoria e Governança**

```text
1. O sistema registra consentimentos e bases legais por tipo de dado tratado.
2. Ações críticas de usuários são enviadas para AuditLog.
3. Solicitações do titular podem ser abertas para acesso, correção, exclusão ou portabilidade.
4. Dados pessoais e financeiros seguem políticas de retenção, minimização e anonimização.
5. Logs e evidências de tratamento suportam auditoria e governança da plataforma.
```

***

## 5. Casos de Uso e Histórias de Usuário em Visão Macro

### **Administrador da Editora**

```text
Como Administrador da Editora, quero:
- cadastrar e configurar minha editora como tenant da plataforma;
- personalizar branding, domínios, identidade visual e configurações operacionais;
- convidar usuários e atribuir roles e permissões por vínculo com a editora;
- habilitar ou desabilitar serviços editoriais e serviços de IA;
- configurar integrações com ERP, gateway de pagamento e provedores externos;
- acompanhar indicadores operacionais, financeiros e de uso da plataforma;
- manter trilha de auditoria e governança sobre o tenant.
```

### **Gestor Editorial**

```text
Como Gestor Editorial, quero:
- cadastrar livros e estruturar a esteira editorial de cada projeto;
- selecionar os serviços necessários para cada obra;
- distribuir tarefas entre colaboradores internos, parceiros e IA;
- acompanhar status, prazos, gargalos e dependências entre etapas;
- revisar entregas, solicitar ajustes e aprovar resultados;
- manter a operação editorial organizada e previsível.
```

### **Autor**

```text
Como Autor, quero:
- cadastrar meu livro ou manuscrito na plataforma;
- contratar serviços editoriais avulsos ou em pacote;
- escolher, quando disponível, entre execução humana, por IA ou híbrida;
- acompanhar o andamento de cada etapa da produção do meu livro;
- enviar arquivos, responder dúvidas e interagir com os responsáveis via chat;
- visualizar valores cobrados, histórico de pagamentos e entregas recebidas;
- aprovar resultados e baixar os artefatos finais do projeto.
```

### **Atendimento / Suporte**

```text
Como membro do Atendimento, quero:
- acompanhar solicitações dos autores ao longo do processo editorial;
- intermediar dúvidas, pendências e alinhamentos entre autor, editora e parceiros;
- visualizar o contexto do livro, das etapas e das mensagens trocadas;
- abrir, tratar e encerrar tickets operacionais;
- garantir que o autor tenha visibilidade e suporte durante toda a jornada.
```

### **Revisor Literário**

```text
Como Revisor Literário, quero:
- receber etapas atribuídas a mim com prazo, contexto do livro e da editora em que estou atuando;
- acessar o manuscrito e os artefatos relacionados;
- enviar versões revisadas, comentários e orientações ao autor ou gestor;
- registrar minha evolução dentro da etapa;
- concluir a entrega com histórico rastreável.
```

### **Revisor Ortográfico**

```text
Como Revisor Ortográfico, quero:
- acessar textos enviados para correção gramatical e ortográfica;
- devolver versões corrigidas com observações e apontamentos;
- interagir com autor e gestor editorial quando houver dúvidas;
- registrar entregas de maneira organizada e auditável.
```

### **Capista**

```text
Como Capista, quero:
- receber briefing, referências e materiais do livro;
- enviar propostas, versões e arquivos finais de capa;
- discutir ajustes com autor e gestor pelo chat da etapa;
- concluir a entrega com versionamento e histórico de aprovação.
```

### **Ilustrador**

```text
Como Ilustrador, quero:
- visualizar as demandas visuais atribuídas ao projeto;
- acessar briefing, referências e requisitos técnicos;
- publicar versões intermediárias e finais das ilustrações;
- colaborar com a editora e o autor dentro do contexto do livro.
```

### **Diagramador**

```text
Como Diagramador, quero:
- receber os textos finais e os elementos visuais aprovados;
- produzir os arquivos diagramados para impressão e ebook;
- versionar PDFs, EPUBs e demais saídas técnicas;
- registrar entregas e ajustes até a aprovação final.
```

### **Marketing / Tráfego**

```text
Como responsável por Marketing, quero:
- receber informações do livro e seus diferenciais;
- gerar campanhas, copys, criativos e peças promocionais;
- trabalhar com recursos humanos e/ou IA para acelerar produção;
- acompanhar entregas vinculadas ao lançamento e à divulgação da obra.
```

### **Financeiro da Editora**

```text
Como membro do Financeiro, quero:
- visualizar serviços contratados, valores cobrados e pagamentos recebidos;
- acompanhar repasses devidos a parceiros e freelancers;
- operar o modelo financeiro da editora, seja com repasse direto ou saque integral;
- gerar extratos por livro, serviço, parceiro e período;
- integrar eventos financeiros ao ERP e manter conciliação adequada.
```

### **Parceiro / Freelancer**

```text
Como Parceiro ou Freelancer, quero:
- acessar apenas as etapas e livros em que fui alocado, mesmo estando vinculado a mais de uma editora;
- receber tarefas, prazos, briefing e materiais necessários;
- trocar mensagens com os envolvidos na etapa;
- publicar artefatos e registrar entregas;
- acompanhar meus valores a receber e o status dos repasses.
```

### **Agente de IA / Automação Sistêmica**

```text
Como Agente de IA, o sistema deve:
- executar tarefas editoriais automatizadas conforme configuração da editora;
- registrar provider, modelo, prompt, custo, tokens e resultado;
- gerar artefatos auditáveis e vinculados à etapa correspondente;
- respeitar limites de uso, políticas de aprovação e regras do tenant;
- permitir revisão humana posterior quando necessário.
```

### **Equipe de Compliance / Governança**

```text
Como responsável por Compliance ou Governança, quero:
- garantir o registro das bases legais e consentimentos;
- rastrear ações críticas de usuários e processos automatizados;
- atender solicitações do titular relativas à LGPD;
- revisar retenção, anonimização e exposição de dados sensíveis;
- produzir evidências para auditoria e governança do tratamento de dados.
```

## **Casos de Uso Macro da Plataforma**

### **1. Implantação e Configuração do Tenant**
- Cadastro da editora.
- Criação do primeiro administrador.
- Definição de branding, serviços, integrações e parâmetros iniciais.

### **2. Gestão de Usuários e Permissões**
- Convite e cadastro de usuários.
- Associação de um mesmo usuário a múltiplas editoras por meio de vínculos distintos.
- Associação de roles e permissões por vínculo.
- Restrição de acesso por tenant, livro, etapa e função.

### **3. Gestão de Catálogo Editorial**
- Cadastro de serviços editoriais ofertados.
- Definição de preços, SLA, forma de execução e regras de entrega.
- Configuração de serviços com suporte humano, IA ou híbrido.

### **4. Gestão de Livros e Artefatos**
- Cadastro do livro e seus metadados.
- Upload, organização e versionamento de manuscritos e entregáveis.
- Associação de participantes ao projeto editorial.

### **5. Contratação e Execução de Serviços**
- Seleção e contratação de serviços pelo autor.
- Geração das etapas do workflow.
- Execução por humano, parceiro ou IA.
- Aprovação, retrabalho e conclusão.

### **6. Comunicação e Atendimento**
- Chat contextual por livro e por etapa.
- Atendimento operacional e suporte ao autor.
- Registro histórico de decisões, pedidos e arquivos.

### **7. Automação com IA**
- Execução de tarefas automatizadas.
- Registro auditável das operações realizadas por IA.
- Aprovação humana opcional ou obrigatória conforme regra da editora.

### **8. Pagamentos, Repasse e Conciliação**
- Cobrança do autor.
- Repasses a parceiros.
- Extratos financeiros e integração com ERP.

### **9. Segurança, LGPD e Auditoria**
- Controle de acesso.
- Registro de consentimentos.
- Atendimento a direitos do titular.
- Trilha auditável de ações humanas e sistêmicas.

***

## 6. Etapas de Desenvolvimento

### **Fase 1: Infra, Identidade e Onboarding (2–4 semanas)**

1. Configurar repositórios Git, convenções de branching, CI/CD, ambientes local/dev/staging e scaffold inicial .NET 8 + Angular + Docker Compose com PostgreSQL.
2. Implementar a base multi-tenant com `EditoraId`, filtros globais, autenticação JWT, refresh token e estrutura inicial de autorização.
3. Criar o onboarding da editora: cadastro da `Editora`, criação automática do primeiro usuário administrador, login, seleção de contexto de editora, dashboard inicial e branding básico.
4. Estruturar o modelo de identidade com `Usuario`, `VinculoUsuarioEditora`, `Role`, permissões granulares, trilha de auditoria e logs administrativos.

### **Fase 2: Gestão de Usuários, Roles e Atores Editoriais (2–4 semanas)**

5. Implementar CRUD de usuários e vínculos por editora: cadastro manual, convite, ativação, inativação, redefinição de acesso e criação de novos vínculos para usuários já existentes na plataforma.
6. Implementar os perfis editoriais iniciais: `AdminEditora`, `GestorEditorial`, `Financeiro`, `Atendimento`, `Autor`, `RevisorLiterario`, `RevisorOrtografico`, `Capista`, `Ilustrador`, `Diagramador`, `Marketing`, `FreelancerParceiro`.
7. Implementar matriz de autorização por vínculo, role e escopo, garantindo que cada ator visualize apenas os livros, etapas, chats, artefatos e rotinas financeiras compatíveis com a editora/contexto ativo.
8. Criar telas de gestão de usuários, perfis, permissões, vínculos multi-editora e associação dos usuários aos fluxos editoriais.

### **Fase 3: Core Editorial e Cadastro de Livros (4–6 semanas)**

9. Implementar entidades e regras centrais: `Livro`, `WorkflowEtapa`, `Artefato`, `VersaoArtefato`, relacionamentos com autor, editora e executores.
10. Criar CRUD de livros, cadastro de metadados editoriais/comerciais e upload inicial de manuscritos e demais artefatos.
11. Permitir associar atores editoriais ao livro e às etapas da esteira por meio do vínculo correto com a editora, incluindo colaboradores internos e parceiros externos que atuem em múltiplos tenants.
12. Implementar a primeira versão da workflow engine com estados, responsáveis, transições e acompanhamento por dashboard.

### **Fase 4: Colaboração, Atendimento e Chat (2–4 semanas)**

13. Implementar chat por livro e por etapa, com mensagens persistidas, anexos, histórico e identificação do remetente humano ou IA.
14. Criar notificações de atribuição, atualização, menção, pendência e conclusão de etapa.
15. Implementar visão de atendimento/suporte para acompanhamento operacional do autor e interação entre os participantes do processo editorial.

### **Fase 5: Serviços Editoriais Humanos e Automatizados (4–6 semanas)**

16. Modelar o catálogo de serviços editoriais contratáveis: avaliação de plágio, revisão literária, revisão ortográfica, criação de capa, ilustração, diagramação, geração de ebook, preparação para impressão e campanhas de marketing.
17. Permitir que cada serviço seja configurado para execução humana, automatizada por IA ou híbrida, conforme política da editora.
18. Implementar alocação de executor por etapa, SLA, aceite, entrega e versionamento dos resultados produzidos.

### **Fase 6: IA e Automação (3–5 semanas)**

19. Integrar OpenAI/Azure OpenAI para revisão textual, apoio editorial, geração de capa, criativos e automações assistidas.
20. Implementar execução paralela de etapas humanas e automatizadas, com rastreabilidade de prompts, custos, resultados e versões geradas.
21. Permitir que a editora configure quais serviços de IA estarão disponíveis aos autores, com limites, aprovações e políticas de uso.

### **Fase 7: Pagamentos, Repasse e Integração ERP (4–6 semanas)**

22. Implementar contratação e pagamento dos serviços pelo autor, com conciliação por evento/webhook.
23. Criar extrato detalhado por livro, serviço, parceiro, etapa e editora.
24. Implementar regras de repasse: pagamento direto ao parceiro/freelancer ou saque integral pela editora com repasse externo sob sua responsabilidade.
25. Integrar ERP via `IProviderErp` para cadastro de produtos/serviços, pedidos, faturamento e rotinas operacionais.

### **Fase 8: LGPD, Segurança e Governança (2–4 semanas)**

26. Implementar consentimento granular, base legal, retenção e anonimização de dados pessoais, editoriais e financeiros.
27. Implementar audit logs, trilha de acesso por usuário e tenant, revisão periódica de permissões e segregação de funções.
28. Criar rotinas para direitos do titular: exportação, correção, exclusão, revogação de consentimento e registro das solicitações LGPD.

### **Fase 9: Operação, Backoffice e Escala (2–4 semanas)**

29. Criar backoffice central para administração da plataforma, gestão de tenants, suporte operacional e monitoramento.
30. Implementar métricas operacionais, observabilidade, alertas, health checks e rotinas de suporte.
31. Preparar a evolução para escala: filas, processamento assíncrono, políticas de custo de IA e governança multi-tenant.

***

## 7. Conformidade LGPD e Segurança

### **Princípios LGPD implementados**

- **Finalidade específica**: dados só processados para serviços contratados.[^4]
- **Consentimento granular**: autor aprova cada tipo de dado (pessoal, financeiro, conteúdo).
- **Direitos do titular**: download, exclusão, portabilidade via API.
- **DPO virtual**: logs automáticos + relatórios ANPD.

### **Medidas de segurança**

```text
🔐 Autenticação: JWT + Refresh Token + 2FA opcional
🔒 Autorização: RBAC por EditoraId + vínculo ativo + Role
🛡️ LGPD: Consentimento armazenado, AuditLog 1 ano, Anonimização PII
📊 Banco: Row-Level Security Postgres + EditoraId filter
☁️ Arquivos: Azure Blob + SAS Tokens temporários
💳 Pagamentos: PCI-DSS via Stripe, sem armazenar CVV
```

**Auditoria**: todos acessos logados com `UsuarioId`, `VinculoUsuarioEditoraId` (quando aplicável), `EditoraId`, `IP`, `Timestamp`, `DadosAnonimizados`.

***

## 8. Uso de Perplexity + Gemini + Copilot

### **Workflow de 30 minutos por slice**

**Min 0–5 | Perplexity** (navegador):

```text
"Monte o próximo slice: [nome]. Liste entidades, APIs, fluxos."
```

**Min 5–15 | Gemini Pro** (navegador):

```text
"Gere: entidades C#, DTOs, docker-compose, controller skeleton."
```

**Min 15–25 | Copilot + Gemini Code Assist** (VS Code):

```text
Comentários em inglês → // create controller for [feature]
TAB → aceitar sugestão → ajustar.
```

**Min 25–30 | Perplexity** (revisão):

```text
"Revise este código [colar] e sugira melhorias LGPD/performance."
```

### **Prompts reutilizáveis**

**Perplexity (planejamento)**:

```text
"SaaS editorial multi-tenant .NET 8 + Angular + Postgres.
Próximo slice: [NOME]. Entidades, APIs, fluxos, LGPD."
```

**Gemini (geração)**:

```text
"Gere código .NET 8 para [entidade/controller]:
- Multi-tenant EditoraId
- LGPD compliant
- Domain Events
```

**Copilot (implementação)**:

```text
// Create [controller/service] with EditoraId filter
// Use IPasswordHasher, SaveChangesAsync transaction
// Return Created with Id
```

Essa combinação te dá **arquitetura sólida + desenvolvimento 3x mais rápido** mantendo qualidade e conformidade.[^8][^9][^10]

<div align="center">⁂</div>

[^1]: https://books.google.com.br/books?id=bhsEEQAAQBAJ
[^2]: https://aws.amazon.com/pt/blogs/aws-brasil/criando-uma-solucao-saas-multi-tenant-usando-o-amazon-eks/
[^3]: https://frontegg.com/resources/the-complete-guide-to-saas-multi-tenant-architecture
[^4]: https://complydog.com/blog/brazil-lgpd-complete-data-protection-compliance-guide-saas
[^5]: https://affise.com/lgpd/
[^6]: https://desenvolvedor.io/blog/desvendando-os-bounded-contexts-aplicando-ddd-em-projetos-reais
[^7]: https://oneuptime.com/blog/post/2026-02-17-how-to-implement-domain-driven-design-bounded-contexts-as-microservices-on-gcp/view
[^8]: https://www.linkedin.com/posts/adevofc_copilot-vs-gemini-code-assist-adev-blog-activity-7428717092025278464-dC9M
[^9]: https://www.secondtalent.com/resources/ai-code-assistants-for-visual-studio-code/
[^10]: https://angular.dev/ai/develop-with-ai
