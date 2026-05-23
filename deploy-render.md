# Dossiê de Configuração para Deploy no Render

Este documento serve como um guia detalhado passo a passo para realizar o deploy com sucesso desta aplicação Kanban no **Render**.

---

## 1. Mudanças Realizadas no Projeto para o Deploy

Para garantir que a aplicação funcione corretamente no ambiente de nuvem do Render, as seguintes alterações foram implementadas no código-fonte:

1. **Porta Dinâmica (`server.js`):** A porta do servidor foi alterada de `3001` fixo para `process.env.PORT || 3001`. O Render gerencia e atribui automaticamente a porta onde o servidor deve escutar por meio da variável de ambiente `PORT`.
2. **Chave Secreta Dinâmica (`server.js`):** O JWT agora consome a chave secreta a partir de `process.env.SECRET_KEY`, aumentando a segurança da produção em comparação ao uso de uma chave em texto puro no código.
3. **Persistência Dinâmica (`server.js`):** Os arquivos de banco de dados JSON (`users.json`, `tasks.json`, `revoked_tokens.json`) foram adaptados para ler de um diretório configurável via variável de ambiente `DATA_DIR` (`process.env.DATA_DIR || __dirname`). Isso permite a anexação de um disco persistente no Render.
4. **URL da API Inteligente (`public/index.html`):** O frontend detecta automaticamente se a aplicação está rodando em ambiente local (usando `http://localhost:3001/api`) ou se está rodando no Render (usando `/api` de forma relativa no mesmo domínio).

---

## 2. Passo a Passo do Deploy no Render

Siga os passos abaixo na plataforma [Render](https://render.com/) para fazer o deploy da aplicação:

### Passo 1: Criar uma Conta e Conectar o GitHub
1. Acesse o painel do [Render](https://dashboard.render.com/) e faça login (recomenda-se usar sua conta do GitHub para facilitar a integração).
2. Clique no botão **New +** no canto superior direito e selecione a opção **Web Service**.

### Passo 2: Selecionar o Repositório GitHub
1. Conecte sua conta do GitHub ao Render (caso ainda não tenha feito).
2. Na lista de repositórios exibidos, localize o repositório `KanbanSb` (ou o nome do seu repositório sincronizado) e clique em **Connect**.

### Passo 3: Configurar os Detalhes do Web Service
Preencha o formulário de criação com as seguintes definições:

* **Name:** Escolha um nome para o seu serviço (ex: `kanban-sistema`).
* **Region:** Selecione a região mais próxima dos seus usuários (ex: `Oregon (US West)` ou `Ohio (US East)`).
* **Branch:** Selecione `main` (ou a branch principal que você deseja implantar).
* **Runtime:** Selecione **Node**.
* **Build Command:** `npm install`
* **Start Command:** `npm start` (ou `node server.js`).
* **Instance Type:** Selecione o plano **Free** (Grátis) para testes ou outro plano de sua preferência.

### Passo 4: Configurar as Variáveis de Ambiente (Environment Variables)
No menu de configuração do serviço (durante a criação ou após a criação na aba **Env Groups / Environment Variables**), adicione as seguintes variáveis de ambiente clicando em **Add Environment Variable**:

1. **`SECRET_KEY`**: Insira uma string longa, aleatória e segura para a criptografia dos tokens JWT (ex: `U0VEQ0EtZ1QwWkhI...`). **Não reutilize a chave padrão de desenvolvimento.**
2. **`NODE_ENV`**: Defina como `production`.
3. **`DATA_DIR`**: Defina como `/data` (necessário se você for usar um disco persistente para salvar os dados, veja a seção abaixo).

### Passo 5: Adicionar um Disco Persistente (Opcional, mas Altamente Recomendado)
Como a aplicação utiliza arquivos JSON locais para salvar os usuários e tarefas, o sistema de arquivos padrão do plano Free do Render é **efêmero** (os dados de novos cadastros e tarefas salvas se perdem sempre que o servidor reinicia ou recebe novas atualizações). 

Para evitar a perda de dados, configure um disco persistente:
1. Após criar o Web Service, vá para a aba **Disks** no menu lateral esquerdo do serviço no Render.
2. Clique em **Add Disk**.
3. Preencha com as seguintes informações:
   * **Name:** `kanban-dados` (ou outro nome desejado).
   * **Mount Path:** `/data` (este caminho deve ser exatamente o mesmo valor definido na variável `DATA_DIR`).
   * **Size:** `1 GiB` (mais do que suficiente para armazenar arquivos JSON de texto).
4. Clique em **Save**. O Render reiniciará o serviço automaticamente com o disco montado no caminho indicado.

---

## 3. Verificando o Status do Deploy

1. Monitore a aba **Events** e **Logs** no painel do Render para acompanhar o progresso de instalação das dependências (`npm install`) e a inicialização do servidor.
2. Quando o log indicar `Server running on http://localhost:3001` (ou similar) e o status do serviço mudar para **Live** (em verde), clique na URL fornecida pelo Render no topo da página (ex: `https://seu-kanban.onrender.com`).
3. Faça o registro de um usuário e teste a criação de tarefas no quadro Kanban para validar o funcionamento integrado do sistema!
