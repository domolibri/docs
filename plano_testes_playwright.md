# Plano de Testes — Playwright (Slices 1 e 2)

Este documento detalha os cenários de teste de aceitação (E2E) que devem ser validados utilizando o Playwright, conforme os requisitos dos Slices 1 e 2.

| Func/Slice | Funcionalidade | Descrição do Teste | Resultado Esperado |
| :--- | :--- | :--- | :--- |
| **S1-01** | Cadastro de Editora (Onboarding) | Preencher o formulário de cadastro com dados válidos e aceitar os Termos de Uso. | Redirecionamento para `/cadastro/sucesso` e exibição da mensagem de verificação de e-mail. |
| **S1-02** | Login e Multi-tenant | Realizar login com um usuário que possui acesso a múltiplas editoras. | Exibição da tela de "Selecionar Editora" e, após a seleção, redirecionamento para o Dashboard com o branding da editora escolhida. |
| **S1-03** | Verificação de E-mail | Tentar logar com uma conta que ainda não confirmou o e-mail. | Exibição de alerta informando que o e-mail não foi verificado e presença do botão "Reenviar e-mail". |
| **S1-04** | Proteção de Rotas | Acessar a URL da Home (`/`) sem estar autenticado. | Redirecionamento automático para a página de login (`/login`). |
| **S1-05** | Branding Dinâmico | Logar em uma editora com cor primária customizada. | Verificação de que os elementos principais (botões, links) estão utilizando a cor definida no branding. |
| **S2-01** | Gestão de Usuários | Acessar a lista de usuários da editora como Admin. | Visualização apenas dos usuários vinculados à editora atual (isolamento via `EditoraId`). |
| **S2-02** | Convite por E-mail | Enviar um convite para um novo usuário (E-mail e Role). | Criação do convite no banco, envio de e-mail (simulado/Mailpit) e exibição do convite como "Pendente" na lista. |
| **S2-03** | Aceite de Convite | Acessar o link de convite recebido por e-mail e completar o cadastro. | Criação do vínculo `UsuarioEditora` com a role correta e acesso imediato à editora que o convidou. |
| **S2-04** | Atribuição de Roles | Alterar a Role de um usuário existente na editora. | Atualização imediata das permissões do usuário no próximo login (refletido no token JWT). |
| **S2-05** | Inativação de Vínculo | Inativar o acesso de um usuário a uma editora específica. | Impedimento de login/acesso àquele tenant, mas preservação do usuário na base global e histórico de ações. |
| **S2-06** | Domínio Próprio | Acessar a aplicação através de um domínio customizado (ex: `editora.com.br`). | O sistema resolve o tenant automaticamente pelo host e carrega o branding sem exigir seleção manual. |
