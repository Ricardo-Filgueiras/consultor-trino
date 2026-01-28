#  консультант Trino

## Visão Geral

Este projeto implanta um ambiente local de análise de dados usando Docker, Trino e Hive Metastore. Ele permite executar consultas SQL em dados armazenados em um serviço de armazenamento de objetos compatível com S3 (como o MinIO), tratando os dados como se estivessem em um banco de dados tradicional.

É um "data lakehouse" em miniatura, ideal para desenvolvimento, testes e estudos de engenharia de dados.

## Arquitetura

O ambiente é composto por três serviços principais orquestrados pelo Docker Compose:

```
+------------------+      +---------------------+      +---------------------+
|                  |      |                     |      |                     |
|   Cliente SQL    +------>      Trino (Motor de Consulta)      +------>  Hive Metastore   |
| (DBeaver, etc.)  |      |  (Porta: 8082)      |      |    (Metadados)      |
|                  |      |                     |      |                     |
+------------------+      +----------+----------+      +----------+----------+
                                     |
                                     | (Lê os dados)
                                     v
                             +------------------+
                             |                  |
                             |   MinIO (S3)     |
                             |  (Armazenamento) |
                             |                  |
                             +------------------+
```

1.  **Trino:** O motor de consulta distribuído. Ele recebe o SQL, planeja a consulta e busca os dados na fonte (MinIO).
2.  **Hive Metastore:** Atua como o catálogo de metadados. Ele informa ao Trino onde os dados da tabela (`s3a://...`) estão localizados e em que formato (`PARQUET`).
3.  **MinIO (Externo):** Onde os arquivos de dados (ex: Parquet, CSV) são de fato armazenados. **Este serviço não está neste `docker-compose.yml` e deve estar rodando separadamente.**

## Pré-requisitos

1.  **Docker e Docker Compose:** Essenciais para executar o ambiente.
2.  **Container MinIO:** Um container MinIO (ou outro S3 compatível) deve estar em execução.
3.  **Rede Docker Externa:** Uma rede Docker para comunicação entre os serviços. O padrão usado é `src_lakehouse`.

## Configuração e Instalação

1.  **Crie a Rede Docker:**
    Se a rede ainda não existir, crie-a manualmente:
    ```bash
    docker network create src_lakehouse
    ```

2.  **Conecte o MinIO à Rede:**
    Certifique-se de que seu container MinIO esteja conectado à mesma rede para que o Trino possa encontrá-lo pelo nome.
    ```bash
    docker network connect src_lakehouse <nome_do_seu_container_minio>
    ```

3.  **Configure o `hive.properties`:**
    Edite o arquivo `trino-service/trino/catalog/hive.properties` para garantir que a configuração do S3 está correta:
    *   `hive.s3.endpoint`: Deve apontar para o endereço e porta da API do seu container MinIO (ex: `http://minio-lake:9000`).
    *   `hive.s3.aws-access-key` e `hive.s3.aws-secret-key`: Devem conter as credenciais de acesso ao seu MinIO.

4.  **Baixe os Drivers S3:**
    Este projeto usa um método de montagem de volume para carregar os drivers S3 no Trino. Crie a pasta e baixe os arquivos:
    ```bash
    # Na pasta raiz do projeto
    mkdir -p trino-service/trino_jars

    # Baixar drivers
    curl -L -o trino-service/trino_jars/hadoop-aws-3.3.4.jar https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.4/hadoop-aws-3.3.4.jar
    curl -L -o trino-service/trino_jars/aws-java-sdk-bundle-1.12.367.jar https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.367/aws-java-sdk-bundle-1.12.367.jar
    ```
    O `docker-compose.yml` já está configurado para usar esses arquivos.

5.  **Inicie os Serviços:**
    Navegue até a pasta `trino-service` e execute:
    ```bash
    docker-compose up -d
    ```

## Uso

-   **Endpoint do Trino:** `http://localhost:8082`
-   **Usuário:** qualquer nome de usuário (ex: `admin`)
-   **Catálogo:** `hive`
-   **Schema:** `raw` (ou o schema que você desejar criar)

Você pode se conectar ao Trino usando qualquer cliente SQL compatível com JDBC, como DBeaver, DataGrip ou o próprio CLI do Trino.

### Exemplo de Consulta

Após criar uma tabela, você pode consultá-la normalmente:

```sql
SELECT * FROM hive.raw.titanic LIMIT 10;
```