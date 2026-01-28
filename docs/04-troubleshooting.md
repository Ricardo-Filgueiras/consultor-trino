# Guia de Troubleshooting

Este documento lista erros comuns e suas soluções, baseados no histórico de desenvolvimento deste projeto.

### Erro: `Unable to execute HTTP request: minio-lake: Name or service not known`

-   **Causa:** O container do Trino não consegue encontrar o container do MinIO na rede com o nome `minio-lake`.
-   **Solução:**
    1.  Verifique se o nome do seu container MinIO é realmente `minio-lake`. Se for diferente, atualize a linha `hive.s3.endpoint` no arquivo `trino-service/trino/catalog/hive.properties`.
    2.  Garanta que ambos os containers, Trino e MinIO, estão conectados à mesma rede Docker (ex: `src_lakehouse`). Use o comando `docker network inspect src_lakehouse` para ver os containers conectados.
    3.  Se o container MinIO não estiver na rede, adicione-o com `docker network connect src_lakehouse <nome_do_container_minio>`.

### Erro: `ClassNotFoundException: Class org.apache.hadoop.fs.s3a.S3AFileSystem not found`

-   **Causa:** O Trino não possui os drivers (arquivos JAR) necessários para se comunicar com sistemas de arquivo S3.
-   **Solução:**
    1.  Certifique-se de que você criou a pasta `trino-service/trino_jars`.
    2.  Verifique se os arquivos `hadoop-aws-3.3.4.jar` e `aws-java-sdk-bundle-1.12.367.jar` foram baixados corretamente para dentro dessa pasta.
    3.  Confirme que o `docker-compose.yml` contém as linhas que montam esses dois arquivos no diretório de plugins do Trino (`/usr/lib/trino/plugin/hive/`).

### Erro: `OCI runtime create failed ... not a directory: Are you trying to mount a directory onto a file?`

-   **Causa:** Conflito de montagem de volumes no `docker-compose.yml`. Isso ocorre ao tentar montar um arquivo (`.jar`) dentro de um diretório (`/etc/trino/catalog`) que já é o destino de outra montagem de volume de diretório.
-   **Solução:**
    -   Os arquivos JAR não devem ser montados em `/etc/trino/catalog`. O destino correto e mais robusto é o diretório de plugins do conector, como `/usr/lib/trino/plugin/hive/`. O `docker-compose.yml` deste projeto já deve refletir essa configuração correta. Se você encontrar este erro, verifique se os caminhos de destino dos volumes dos JARs estão corretos.

### Problema: O container do Trino não inicia ou para imediatamente

-   **Causa:** Geralmente, um erro no script de inicialização (`command`) dentro do `docker-compose.yml` ou um erro de configuração que impede o Trino de iniciar.
-   **Solução:**
    1.  Verifique os logs do container com `docker logs trino`.
    2.  Se você estiver usando um script de inicialização para baixar arquivos, verifique se os comandos estão corretos, se os URLs são válidos e se o container tem permissão para escrever nos diretórios de destino.
    3.  A solução mais robusta (adotada por este projeto) é evitar scripts de download em tempo de execução e preferir montar os arquivos necessários via volumes Docker.
