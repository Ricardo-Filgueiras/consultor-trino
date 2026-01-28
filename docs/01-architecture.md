# Arquitetura Detalhada

Este documento detalha os componentes da plataforma e como eles interagem.

## Componentes

### Trino (Motor de Consulta)
- **Imagem Docker:** `trinodb/trino:451`
- **Função:** O cérebro da operação. Trino é responsável por receber, analisar, planejar e executar as consultas SQL. Ele não armazena dados por si só; ele é um motor de "consulta sobre dados".
- **Conectores:** A força do Trino está em seus conectores. Neste projeto, usamos o conector `hive` para permitir que o Trino entenda como interagir com o Hive Metastore e, por extensão, com o S3.
- **Drivers S3:** A imagem base do Trino não inclui os drivers para o sistema de arquivos S3 (`s3a`). Eles são adicionados manualmente ao classpath do conector Hive através de um volume Docker para permitir a comunicação com o MinIO.

### Hive Metastore (Catálogo de Metadados)
- **Imagem Docker:** `apache/hive:4.0.0`
- **Função:** Atua como um "índice" ou "agenda de contatos" para os seus dados. Quando você executa `CREATE TABLE`, ele armazena as informações (metadados) sobre essa tabela:
    - Nome da tabela e suas colunas/tipos de dados.
    - O formato dos dados (ex: `PARQUET`).
    - A localização física dos dados (ex: `s3a://raw/web/titanic`).
- **Interação:** O Trino consulta o Metastore para saber onde buscar os dados de uma tabela antes de ir ao S3 para lê-los.

### MinIO (Armazenamento de Objetos)
- **Status:** Dependência Externa.
- **Função:** O "armazém" onde os arquivos de dados brutos são guardados. O MinIO emula a API do Amazon S3, permitindo que ferramentas do ecossistema Hadoop/Big Data, como o Trino, interajam com ele.
- **Buckets e Pastas:** Dentro do MinIO, os dados são organizados em "buckets" (similares a HDs) e pastas, formando o caminho para a `external_location` das suas tabelas.

## Fluxo de uma Consulta (`SELECT`)

1.  Um cliente envia uma consulta `SELECT * FROM hive.raw.titanic` para o Trino.
2.  O Trino percebe que a consulta é para o catálogo `hive`. Ele pergunta ao Hive Metastore: "Quais são os metadados da tabela `raw.titanic`?".
3.  O Metastore responde, informando: "Os dados estão em formato Parquet no local `s3a://raw/web/titanic`".
4.  Usando os drivers S3 carregados, o Trino estabelece uma conexão com o MinIO.
5.  O Trino lê os arquivos Parquet do caminho especificado, processa os dados (filtros, agregações, etc.) e retorna o resultado para o cliente.
