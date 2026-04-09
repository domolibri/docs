# Relatório de Avaliação Arquitetural - Backend (Slice 1)

Este relatório apresenta uma análise técnica da arquitetura do backend do projeto **SaaS-Editorial**, focando na aplicação de padrões de projeto e princípios de design de software.

## 1. Metodologia de Análise
A avaliação foi realizada através da inspeção do código-fonte nas camadas de `Api`, `Application`, `Domain` e `Infrastructure`, focando em:
*   **Padrões GoF (Gang of Four):** Estruturais, comportamentais e de criação.
*   **Princípios GRASP (General Responsibility Assignment Software Patterns).**
*   **Princípios SOLID:** Fundamentos de programação orientada a objetos.

---

## 2. Avaliação de Padrões GoF e Corporativos

### 2.1. Strategy (GoF)
*   **Status:** **Conforme.**
*   **Observação:** O uso da interface `IEmailService` com múltiplas implementações (`SmtpEmailService`, `ConsoleEmailService`, `BackgroundEmailService`) permite a troca de comportamento de infraestrutura sem impactar a lógica de negócio.

### 2.2. Interceptor / Proxy (GoF)
*   **Status:** **Conforme.**
*   **Observação:** O `AuditInterceptor.cs` utiliza a capacidade de interceptação do EF Core para tratar a auditoria como uma preocupação transversal (Cross-cutting concern), mantendo o código de domínio limpo.

### 2.3. Repository & Unit of Work (Padrões Corporativos)
*   **Status:** **Uso Implícito / Risco de Acoplamento.**
*   **Observação:** O projeto utiliza o `DbContext` diretamente nos serviços e, em alguns casos, nos controllers. Embora o EF Core implemente esses padrões, a falta de abstrações de repositório dificulta a inversão de dependência e testes unitários puros.

---

## 3. Avaliação de Princípios GRASP

### 3.1. Information Expert
*   **Status:** **Não Conforme (Anemia de Domínio).**
*   **Observação:** As entidades no diretório `DomoLibri.Domain/Entities` são puramente portadoras de dados (POCOs). Lógicas que pertencem ao Especialista na Informação (ex: validação de slug em `Editora`, bloqueio de conta em `Usuario`) estão espalhadas em serviços de infraestrutura.

### 3.2. Low Coupling (Baixo Acoplamento)
*   **Status:** **Risco Identificado.**
*   **Observação:** O `AuthController` possui acoplamento direto com o `DomoLibriDbContext`. Mudanças no esquema do banco impactam diretamente o controlador da API, violando a separação de interesses.

### 3.3. High Cohesion (Alta Coesão)
*   **Status:** **Parcialmente Conforme.**
*   **Observação:** O `AuthService.cs` está sobrecarregado ("God Service"), lidando com identidade global, multi-tenancy, envio de e-mails e geração de tokens. Recomenda-se a decomposição em serviços especializados.

---

## 4. Avaliação de Princípios SOLID

| Princípio | Avaliação | Observação |
| :--- | :--- | :--- |
| **SRP** (Single Responsibility) | 🔴 Crítico | `AuthService` e `AuthController` possuem múltiplas razões para mudar. |
| **OCP** (Open/Closed) | 🟡 Médio | O `AuditInterceptor` requer modificação manual para cada nova entidade a ser auditada. |
| **LSP** (Liskov Substitution) | 🟢 Bom | As implementações de `IEmailService` respeitam o contrato da base. |
| **ISP** (Interface Segregation) | 🟡 Médio | `IEmailService` mistura o método genérico de envio com templates específicos. |
| **DIP** (Dependency Inversion) | 🔴 Crítico | Camadas superiores dependem de implementações concretas de dados (`DbContext`). |

---

## 5. Avaliação de DDD (Domain-Driven Design)

O projeto utiliza uma estrutura de pastas inspirada em DDD, mas a implementação tática apresenta falhas conceituais:

*   **Modelos de Domínio Anêmicos:** As entidades são apenas estruturas de dados com `get; set;` públicos. A lógica de negócio "vaza" para os serviços de infraestrutura.
*   **Ausência de Value Objects:** Tipos primitivos (`string`, `Guid`) são usados para tudo. Não há encapsulamento de conceitos como `Email`, `Slug` ou `Senha`.
*   **Falta de Definição de Agregados:** Não há distinção clara entre Raízes de Agregado (Aggregate Roots) e entidades internas. Isso permite que serviços manipulem entidades de forma inconsistente.
*   **Linguagem Ubíqua:** Os termos estão presentes, mas não estão protegidos pelo código (ex: a regra de que uma Editora deve ter um Slug único é validada externamente e não pela própria entidade `Editora`).

---

## 6. Tabela de Inconformidades e Melhorias Detalhada

Esta seção classifica os desvios identificados, priorizando as correções baseadas no impacto arquitetural.

| Tipo | Nome | Nível de Criticidade | Descrição da Inconformidade | Sugestão de Correção |
| :--- | :--- | :--- | :--- | :--- |
| **SOLID** | **SRP** (Responsabilidade Única) | 🔴 Crítico | `AuthService` acumula lógica de identidade, inquilinos (tenants) e templates de e-mail. | Decompor em `IdentityService`, `TenantService` e `NotificationService`. |
| **SOLID** | **DIP** (Inversão de Dependência) | 🔴 Crítico | Application e API dependem diretamente da implementação concreta `DomoLibriDbContext`. | Introduzir interfaces de **Repository** no Domain para abstrair a infraestrutura de dados. |
| **DDD** | **Anemic Model** | 🟠 Alto | Entidades sem comportamento. A lógica de negócio está nos serviços (`Transaction Script`). | Migrar para **Rich Domain Models**, movendo validações e regras para métodos dentro das entidades. |
| **DDD** | **Aggregates** | 🟠 Alto | Entidades como `VinculoUsuarioEditora` são manipuladas sem passar por uma Raiz de Agregado clara. | Definir `Editora` e `Usuario` como Raízes de Agregado e restringir o acesso a entidades dependentes. |
| **DDD** | **Value Objects** | 🟡 Médio | Uso excessivo de tipos primitivos para conceitos complexos. | Criar **Value Objects** para `Email`, `Cnpj`, `CorHex`, etc., garantindo auto-validação na criação. |
| **GRASP** | **Low Coupling** | 🟠 Alto | Acoplamento forte entre controladores da API e o EF Core através do uso direto do `DbContext`. | Garantir que controllers dependam apenas de interfaces da camada de Application. |
| **GoF** | **Factory** | 🟢 Baixo | Criação complexa de grafos de objetos feita manualmente no serviço. | Implementar uma **Domain Factory** para garantir a criação atômica e válida de agregados. |

---

## 7. Prompts Otimizados para Copilot / AI

Para acelerar a implementação das melhorias sugeridas, utilize os prompts abaixo no GitHub Copilot ou similar:

### 7.1. Refatoração para SRP (Serviços)
> "Analise o `AuthService.cs` e identifique as responsabilidades de (1) Gestão de Identidade/Auth, (2) Gestão de Tenants/Editoras e (3) Notificações. Refatore o código extraindo-os para `IdentityService`, `TenantService` e `NotificationService`, garantindo que cada um tenha uma única responsabilidade e se comuniquem via interfaces."

### 7.2. Implementação de Repositórios (DIP)
> "Com base no `DomoLibriDbContext.cs`, gere interfaces de repositório no projeto `Domain/Interfaces` para `Editora` e `Usuario`. Em seguida, crie as implementações em `Infrastructure/Data/Repositories` e atualize os serviços para dependerem dessas interfaces em vez do `DbContext` diretamente."

### 7.3. Migração para Rich Domain Model (DDD)
> "Refatore as entidades `Usuario.cs` e `Editora.cs` para seguirem o padrão de Rich Domain Model. Torne os setters das propriedades privados, crie construtores que validem as invariantes e mova lógicas de negócio (como geração de slug, bloqueio de conta e atualização de branding) dos serviços para métodos expressivos dentro das próprias entidades."

### 7.4. Criação de Value Objects (DDD)
> "Crie uma classe base abstrata `ValueObject` que implemente a igualdade por valor. Em seguida, implemente um Value Object `Email` que valide o formato do endereço no construtor. Substitua o uso de `string Email` na entidade `Usuario` por este novo Value Object."

### 7.5. Desacoplamento de Controllers (Low Coupling)
> "Examine o `AuthController.cs`. Identifique todos os pontos onde o `_db` (DbContext) é acessado diretamente. Mova essa lógica para a camada de `Application` (ou para um repositório) para que o controlador dependa apenas de abstrações e não conheça detalhes da persistência."

---

## 8. Conclusões e Recomendações Técnicas

O backend apresenta uma base sólida e organizada, mas demonstra sinais de "vazamento de responsabilidades" entre as camadas de API e Infraestrutura, além de uma aplicação superficial do DDD.

### Plano de Correção DDD:
1.  **Encapsulamento:** Fechar os `setters` das entidades e criar métodos expressivos (ex: `editora.AtualizarBranding(...)`).
2.  **Validação de Invariantes:** Mover validações (ex: "não pode aceitar termos versão 0") para o construtor ou métodos da entidade.
3.  **Repositórios:** Definir `IEditoraRepository` e `IUsuarioRepository` na camada de Domínio.
4.  **Value Objects:** Implementar o padrão Value Object para tipos que possuem lógica própria (ex: `Email` com validação de formato).

---
**Elaborado por:** Gemini CLI (Arquiteto de Sistemas)
**Data:** 8 de Abril de 2026
