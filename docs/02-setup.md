# Guia de Instalação e Configuração

Este guia fornece os passos detalhados para configurar o ambiente do zero.

## 1. Pré-requisitos

- **Docker Desktop:** Garanta que o Docker e o Docker Compose estejam instalados e em execução.
- **Git:** Para clonar o repositório.
- **Terminal:** Um terminal com `curl` instalado (como Git Bash, WSL ou PowerShell moderno).
- **Container MinIO:** Você precisa ter um container MinIO já em execução. Anote seu nome, porta da API (geralmente `9000`), e credenciais (Access Key e Secret Key).

## 2. Configuração Inicial

### Passo 2.1: Clonar o Repositório
```bash
git clone https://github.com/seu-usuario/consultor-trino.git
cd consultor-trino
```

### Passo 2.2: Criar e Configurar a Rede Docker
O Trino e o MinIO precisam estar na mesma rede para se comunicarem pelo nome do container.

1.  **Crie a rede externa:**
    ```bash
    docker network create src_lakehouse
    ```
2.  **Conecte seu container MinIO:**
    ```bash
    docker network connect src_lakehouse <nome_do_seu_container_minio>
    ```
    Substitua `<nome_do_seu_container_minio>` pelo nome real do seu container. Você pode encontrá-lo com `docker ps`.

## 3. Configuração do Conector Hive
O Trino precisa saber como se conectar ao Metastore e ao MinIO.

1.  Navegue até `trino-service/trino/catalog`.
2.  Edite o arquivo `hive.properties`.
3.  **Verifique/Altere as seguintes linhas:**
    ```properties
    # Garante que o Trino pode encontrar o Hive Metastore pelo nome do serviço no Docker
    hive.metastore.uri=thrift://hive-metastore:9083

    # === Configuração do seu MinIO ===
    # Altere "minio-lake" para o nome do seu container MinIO
    hive.s3.endpoint=http://minio-lake:9000

    # Insira as credenciais do seu MinIO
    hive.s3.aws-access-key=SUA_ACCESS_KEY
    hive.s3.aws-secret-key=SUA_SECRET_KEY

    # Configurações padrão para compatibilidade com MinIO
    hive.s3.path-style-access=true
    hive.s3.ssl.enabled=false
    ```

## 4. Download dos Drivers S3
O Trino precisa de drivers (`.jar`) para falar com o S3.

1.  Na pasta raiz do projeto (`consultor-trino`), crie o diretório para os JARs:
    ```bash
    mkdir -p trino-service/trino_jars
    ```
2.  Baixe os drivers para dentro desta pasta:
    ```bash
    curl -L -o trino-service/trino_jars/hadoop-aws-3.3.4.jar https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.4/hadoop-aws-3.3.4.jar
    curl -L -o trino-service/trino_jars/aws-java-sdk-bundle-1.12.367.jar https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.367/aws-java-sdk-bundle-1.12.367.jar
    ```
O arquivo `docker-compose.yml` já está configurado para usar esses arquivos.

## 5. Iniciar o Ambiente
Com tudo configurado, inicie os serviços Trino e Hive Metastore.

1.  Navegue até a pasta `trino-service`:
    ```bash
    cd trino-service
    ```
2.  Execute o Docker Compose:
    ```bash
    docker-compose up -d
    ```

Após alguns instantes, os serviços estarão prontos para uso. Você pode verificar o status com `docker-compose ps`.
