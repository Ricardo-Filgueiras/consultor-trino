# Guia de Uso

Com o ambiente em execução, você pode começar a criar tabelas e fazer consultas.

## Conectando ao Trino

Você pode usar qualquer cliente SQL que suporte JDBC. As configurações de conexão são:

-   **URL JDBC:** `jdbc:trino://localhost:8082`
-   **Host:** `localhost`
-   **Porta:** `8082`
-   **Usuário:** Qualquer valor. O padrão é usar `admin`. Não há senha por padrão.
-   **Catálogo:** `hive` (para acessar os dados gerenciados pelo Hive Metastore e S3).
-   **Schema:** `raw` ou qualquer outro que você crie com `CREATE SCHEMA`.

### Exemplo com DBeaver
1.  Crie uma nova conexão.
2.  Procure e selecione o driver "Trino".
3.  Na aba "Main", preencha Host, Porta e Usuário.
4.  Na aba "Driver Properties", você pode, opcionalmente, definir `SSL` como `false`.
5.  Teste a conexão.

## Criando um Schema e uma Tabela

1.  **Crie um Schema:**
    Schemas no Trino/Hive são mapeados para diretórios no seu sistema de armazenamento.
    ```sql
    CREATE SCHEMA IF NOT EXISTS hive.raw
    WITH (location = 's3a://raw/');
    ```
    *   **Importante:** O bucket `raw` deve existir no seu MinIO.

2.  **Crie uma Tabela Externa:**
    Uma tabela "externa" diz ao Trino para gerenciar os metadados, mas não os dados em si. Se você deletar a tabela, os arquivos no S3 permanecem.

    Supondo que você tenha os dados do Titanic em formato Parquet na pasta `web/titanic` dentro do bucket `raw`:
    ```sql
    CREATE TABLE hive.raw.titanic (
        passengerid  INTEGER,
        survived     INTEGER,
        pclass       INTEGER,
        name         VARCHAR(255),
        sex          VARCHAR(50),
        age          INTEGER,
        sibsp        INTEGER,
        parch        INTEGER,
        ticket       VARCHAR(50),
        fare         DECIMAL(10,4),
        cabin        VARCHAR(50),
        embarked     VARCHAR(10)
    )
    WITH (
        format = 'PARQUET',
        external_location = 's3a://raw/web/titanic'
    );
    ```

## Executando Consultas

Uma vez que a tabela está criada, você pode consultá-la como qualquer tabela SQL:

**Selecionar os 10 primeiros passageiros:**
```sql
SELECT * FROM hive.raw.titanic LIMIT 10;
```

**Contar o número de sobreviventes por classe:**
```sql
SELECT
    pclass,
    survived,
    COUNT(*) AS total
FROM
    hive.raw.titanic
GROUP BY
    pclass,
    survived
ORDER BY
    pclass,
    survived;
```
