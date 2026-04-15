<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

## Backlog Executivo do Slice 2 — Formato Sprint/Kanban

Aqui está o **backlog do Slice 2** formatado como **tabela de sprint/kanban**, com **tarefas técnicas separadas por backend e frontend**, **estimativa de esforço** (em **pontos de história**, assumindo 1 dev fullstack = 8 pontos/dia) e **ordem de prioridade**. O slice é dividido em **2 sprints internos** para facilitar execução.[^1][^2]

### Visão Geral do Sprint

| **Sprint** | **Duração** | **Capacidade** | **Entregas** |
| :-- | :-- | :-- | :-- |
| **Sprint 2A** | 5 dias úteis | 40 pontos | Usuários, convites, roles e permissões |
| **Sprint 2B** | 5 dias úteis | 40 pontos | Domínio próprio, resolução de tenant e branding |

## Tabela Kanban — Histórias e Tarefas

| **ID** | **História** | **Prioridade** | **Esforço Total** | **Backend (Pontos)** | **Frontend (Pontos)** | **Status** |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| **S2-01** | Como admin, cadastrar usuários internos/parceiros | P0 | 8 | 5 | 3 | To Do |
| **S2-02** | Como admin, convidar por e-mail | P0 | 8 | 4 | 4 | To Do |
| **S2-03** | Como admin, atribuir roles por vínculo | P0 | 10 | 6 | 4 | To Do |
| **S2-04** | Como sistema, restringir acesso por tenant/vínculo | P0 | 8 | 8 | 0 | To Do |
| **S2-05** | Como admin, inativar vínculo sem perder histórico | P1 | 5 | 3 | 2 | To Do |
| **S2-06** | Como admin, configurar domínio próprio | P0 | 8 | 4 | 4 | To Do |
| **S2-07** | Como sistema, resolver tenant por host | P0 | 10 | 10 | 0 | To Do |
| **S2-08** | Como usuário, ver branding correto por domínio | P0 | 6 | 1 | 5 | To Do |
| **S2-09** | Como sistema, auditoria de ações administrativas | P1 | 5 | 5 | 0 | To Do |
| **TOTAL** |  |  | **68** | **46** | **22** |  |

## Detalhamento de Tarefas Técnicas

### S2-01 — Cadastro de usuários

**Backend (5 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Criar entidade `VinculoUsuarioEditora` e migration | 1 | - |
| Implementar `POST /api/usuarios` com reaproveitamento de usuário global | 2 | Entidade |
| Service com validação de e-mail único por tenant | 1 | - |
| Controller com filtro `EditoraId` | 1 | - |

**Frontend (3 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Tela de listagem com filtros | 1 | API |
| Modal de cadastro com validação | 1 | - |
| Toast de sucesso/erro | 1 | - |

### S2-02 — Convites por e-mail

**Backend (4 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Entidade `ConviteUsuario` e migration | 1 | - |
| `POST /api/usuarios/convites` com token expiração | 2 | Entidade |
| Endpoint de aceitação do convite | 1 | - |

**Frontend (4 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Campo de convite na tela de usuários | 1 | - |
| Página de aceitação de convite | 2 | API |
| Notificação de convite enviado | 1 | - |

### S2-03 — Atribuição de roles

**Backend (6 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Entidades `Role`, `UsuarioRole` e seed inicial | 2 | - |
| `PUT /api/vinculos/{id}/roles` | 2 | Entidades |
| Policy handler para RBAC por vínculo | 2 | - |

**Frontend (4 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Select de roles múltiplos por usuário | 2 | API |
| Validação de roles obrigatórias | 1 | - |
| Refresh automático de permissões | 1 | - |

### S2-04 — Restrição de acesso

**Backend (8 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Middleware de resolução de tenant por JWT + host | 3 | - |
| Authorization policy por `VinculoUsuarioEditora` | 3 | Middleware |
| Filtro global EF Core por `EditoraId` ativo | 2 | - |

**Frontend (0 pontos):** N/A

### S2-06 — Configuração de domínio

**Backend (4 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Campo `DominioCustomizado` na `Editora` + migration | 1 | - |
| `PUT /api/editoras/{id}/dominio` com validação | 2 | Migration |
| Status de verificação do domínio | 1 | - |

**Frontend (4 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Campo de domínio na tela de editora | 2 | API |
| Validação de formato DNS | 1 | - |
| Indicador de status do domínio | 1 | - |

### S2-07 — Resolução por host

**Backend (10 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Service de resolução tenant por `Host` header | 3 | S2-06 |
| Middleware que popula `EditoraId` no contexto | 3 | Service |
| Fallback para domínio padrão | 1 | - |
| Cache Redis dos domínios ativos | 2 | - |
| Testes de resolução multi-tenant | 1 | - |

**Frontend (0 pontos):** N/A

### S2-08 — Branding por domínio

**Backend (1 ponto):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Endpoint `/api/editora/branding` por tenant | 1 | S2-07 |

**Frontend (5 pontos):**


| Tarefa | Esforço | Dependência |
| :-- | :-- | :-- |
| Service que carrega branding por tenant | 2 | API |
| CSS dinâmico com `CorPrimaria` | 1 | - |
| Logo dinâmico por `LogoUrl` | 1 | - |
| Tema global por editora | 1 | - |

## Planejamento por Sprint

### Sprint 2A (Dias 1-5) — 40 pontos

```
📋 To Do (40) → 🔄 In Progress → ✅ Done
S2-01 (8)    S2-02 (8)    S2-03 (10)   S2-04 (8)    S2-05 (5)
```

**Foco:** Gestão de usuários e autorização base.

### Sprint 2B (Dias 6-10) — 28 pontos

```
📋 To Do (28) → 🔄 In Progress → ✅ Done  
S2-06 (8)    S2-07 (10)   S2-08 (6)    S2-09 (5)
```

**Foco:** White-label e domínio próprio.

## Critérios de Definição de Pronto (DoD)

✅ **Backend:**

- API documentada no Swagger.
- Testes unitários (80% coverage).
- Logs estruturados para ações críticas.
- AuditLog populado.
- LGPD: consentimento e anonimização de PII.

✅ **Frontend:**

- Responsivo e acessível.
- Estados de loading/error.
- Validações no cliente.
- i18n preparado.

✅ **Geral:**

- Deploy em staging testado.
- Documentação atualizada.
- Demo com 2 editoras e domínio próprio.[^2][^1]


## Riscos e Mitigações

| **Risco** | **Probabilidade** | **Impacto** | **Mitigação** |
| :-- | :-- | :-- | :-- |
| Complexidade da resolução por host | Médio | Alto | Implementar primeiro com subdomínio |
| RBAC por vínculo com múltiplos tenants | Alto | Alto | Usar policy handlers granulares |
| Cache de domínios inconsistente | Baixo | Médio | TTL baixo + invalidation por webhook |

## Próximo Passo Sugerido

**Gerar código inicial** das entidades principais (`VinculoUsuarioEditora`, `ConviteUsuario`, `Role`) com migrations, DTOs e controllers skeleton para o **Sprint 2A**.

## Key Takeaways

1. **Slice 2 fecha a base de autorização** para todo o SaaS multi-tenant.[^1]
2. **Domínio próprio é essencial** para white-label desde o início.[^2]
3. **68 pontos totais**, viável em **10 dias úteis** para 1 dev fullstack.
4. **Sprint 2A primeiro** para ter usuários funcionando; **2B** para polish de white-label.
5. **DoD rigoroso** garante qualidade e conformidade LGPD desde o início.[^1][^2]

Hope this helps! Let me know if you have any other questions!

<div align="center">⁂</div>

[^1]: Backlog-Inicial-Implementacao-SaaS-Editorial.md

[^2]: Documentacao-de-Arquitetura-SaaS-Editorial-Consolidada-MultiEditora-2.md

