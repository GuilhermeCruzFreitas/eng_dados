## Resumo rápido
- **AVRO**: evolução de schema e serialização
- **ORC**: performance columnar e compressão máxima
- **Parquet**: performance em colunar, performance em analytics e alto nível de integração

## Geral

- **Avro** é um formato baseado em esquema que armazena dados em formato binário compacto, sendo excelente para evolução de esquemas e integração com sistemas como Kafka. É amplamente usado em pipelines de streaming e ETL.
	- A plataforma de streaming processa **450 bilhões de eventos únicos diariamente** usando AVRO integrado ao Kafka. O sistema Keystone gerencia pipelines de dados em escala petabyte, utilizando **Confluent Schema Registry** para controle de versão automatizado e **Apache Flink** para processamento de streams.
	- O Uber migrou **mais de 20.000 pipelines críticos** para Spark com AVRO, integrando Kafka para streams em tempo real e Apache Flink para analytics. O framework **Sparkle ETL** utiliza AVRO para processamento de dados de viagens, localização e analytics de motoristas. A evolução de schema sem downtime foi fundamental para suportar mudanças frequentes nos formatos de dados.
- **ORC (Optimized Row Columnar)** foi desenvolvido para otimizar consultas analíticas no Hive, oferecendo compressão eficiente e estrutura colunar que acelera agregações e filtros. Inclui estatísticas integradas e índices para melhorar performance.
	- A Meta utiliza **DWRF (fork interno do ORC)** para gerenciar mais de **300 petabytes** em seu data warehouse. O sistema emprega **Presto ORC reader customizado** com milhões de tabelas Hive armazenadas em formato ORC.
- **Parquet** é o formato colunar mais popular no ecossistema Apache, conhecido pela excelente compressão e compatibilidade ampla com ferramentas como Spark, Presto e Drill. É ideal para workloads analíticos.

## Capacidades ACID para formatos além do parquet

Assim como **Delta Lake** adiciona capacidades ACID ao Parquet. Para ORC e Avro, existem soluções similares:
- **Para ORC:**
	- **Apache Hudi** oferece capacidades ACID completas, incluindo upserts, deletes e time travel
	- **Apache Iceberg** também suporta ORC como formato subjacente com transações ACID
	- O próprio **Hive 3.0+** introduziu transações ACID nativas para tabelas ORC
- **Para Avro:**
	- **Apache Hudi** suporta Avro como formato base com todas as capacidades transacionais
	- **Apache Iceberg** pode usar Avro para armazenar dados com garantias ACID
	- **Confluent Schema Registry** com Kafka Connect oferece algumas garantias transacionais para Avro

## Resumo das Opções ACID
- **Parquet**: Delta Lake, Apache Iceberg, Apache Hudi
- **ORC**: Apache Hudi, Apache Iceberg, Hive ACID
- **Avro**: Apache Hudi, Apache Iceberg

Entre essas opções, **Apache Hudi** se destaca por suportar todos os três formatos e ser especificamente projetado para casos de uso que requerem atualizações frequentes em data lakes.

## Criadores dos Formatos de Table Format

- **Apache Hudi**:
	- Criado pela **Uber** em 2016
	- Desenvolvido inicialmente para resolver problemas de latência em pipelines de dados da Uber
	- Nome original era "Hoodie" (Hadoop Upserts anD Incremental)
	- Tornou-se projeto Apache em 2019

- **Delta Lake**:
	- Criado pela **Databricks** em 2019
	- Desenvolvido pela equipe que também criou o Apache Spark
	- Lançado como open source logo após o desenvolvimento inicial
	- Fortemente integrado com o ecossistema Databricks/Spark

- **Apache Iceberg**:
	- Criado pela **Netflix** em 2018
	- Desenvolvido para resolver limitações de particionamento e evolução de esquemas em grande escala
	- Doado para a Apache Software Foundation em 2018
	- Rapidamente adotado por outras grandes empresas de tecnologia

## Contexto Interessante

Cada formato surgiu de necessidades específicas:
- **Hudi (Uber)**: Foco em upserts eficientes e baixa latência
- **Delta Lake (Databricks)**: Integração profunda com Spark e simplicidade de uso
- **Iceberg (Netflix)**: Evolução de esquemas e particionamento inteligente
Todos os três são agora projetos open source maduros, com **Hudi** e **Iceberg** sendo projetos Apache, enquanto **Delta Lake** é mantido pela Linux Foundation sob o projeto Delta Lake.

## Leitura sugerida
- [Understanding Parquet, Iceberg and Data Lakehouses at Broad](https://davidgomes.com/understanding-parquet-iceberg-and-data-lakehouses-at-broad/)
- [Running Apache Kafka on Kubernetes at Shopify - Shopify](https://shopify.engineering/running-apache-kafka-on-kubernetes-at-shopify) (um pouco velho, porém interessante)
- [GroupBy # 44: Meta | The Data Stack - by Vu Trinh](https://vutr.substack.com/p/groupby-44-meta-the-data-stack)
- [How to choose between Parquet, ORC and AVRO for S3, Redshift and Snowflake? - BryteFlow](https://bryteflow.com/how-to-choose-between-parquet-orc-and-avro/)
