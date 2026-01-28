# Conectando ao Trino através do VS Code

Este guia demonstra como se conectar à instância do Trino que está rodando via Docker, usando o Visual Studio Code como cliente.

### Pré-requisitos

- VS Code instalado.
- O ambiente Docker com o Trino deve estar em execução (`docker-compose up -d`).

---

### Método 1: Usando a Extensão Oficial "Trino"

Esta é a abordagem mais recomendada por ser a oficial.

**1. Instale a Extensão**
   - Abra o VS Code.
   - Vá para a aba de Extensões (Ctrl+Shift+X).
   - Procure por `Trino` (publicada pela Trino Software Foundation).
   - Clique em **Instalar**.

**2. Adicione uma Nova Conexão**
   - Após a instalação, um ícone do Trino aparecerá na sua barra de atividades. Clique nele.
   - Na visão "TRINO", clique no ícone `+` para adicionar uma nova conexão.
   - Um formulário de configuração será aberto. Preencha com os seguintes detalhes:
     - **Endpoint:** `http://localhost:8082`
     - **Username:** `admin` (ou qualquer nome de usuário, a configuração padrão não exige autenticação)
     - **Password:** Deixe em branco.
     - **Default Catalog:** `hive`
     - **Default Schema:** `default` (ou pode deixar em branco)
   - Clique em **`Connect`**.

**3. Execute uma Consulta**
   - Após conectar, você verá seus catálogos no painel.
   - Clique com o botão direito na sua conexão e selecione **`New Query`**.
   - Em um novo editor, escreva sua consulta SQL e execute. Por exemplo:
     ```sql
     SHOW TABLES FROM hive.default;
     ```
   - Para executar, use o atalho `Ctrl+Enter` ou clique em **`Run Query`** no topo do editor.

---

### Método 2: Usando a Extensão "SQLTools"

SQLTools é uma ferramenta de banco de dados mais genérica, mas muito poderosa.

**1. Instale as Extensões**
   - Abra o VS Code e vá para a aba de Extensões.
   - Procure por `SQLTools` (publicada por Matheus Teixeira) e instale.
   - Procure por `SQLTools Trino Driver` e instale também.

**2. Adicione uma Nova Conexão**
   - Abra a aba do SQLTools na barra de atividades.
   - Clique em **`Add New Connection`**.
   - Selecione **`Trino`** da lista de drivers.
   - Preencha o formulário de conexão:
     - **Connection Name:** `Trino Local` (ou qualquer nome que preferir).
     - **Server Address:** `localhost`
     - **Port:** `8082`
     - **Username:** `admin`
     - **Password:** Deixe em branco.
     - **Catalog:** `hive`
   - Clique em **`Test Connection`** para verificar se tudo está correto.
   - Clique em **`Save Connection`**.

**3. Execute uma Consulta**
   - No painel do SQLTools, clique na sua nova conexão para ativá-la.
   - Crie um novo arquivo (`.sql`) ou use o `New SQL Query` do SQLTools.
   - Escreva sua consulta. A extensão fornecerá autocomplete para tabelas e colunas.
     ```sql
     SELECT * FROM hive.information_schema.tables;
     ```
   - Clique no ícone **`Run on active connection`** acima do seu script.

Ambos os métodos fornecerão uma excelente experiência para interagir com seus dados no Trino diretamente do seu editor.