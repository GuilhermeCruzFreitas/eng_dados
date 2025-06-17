
### 1. **Arquivos JSON de Commit (00000000000000000000.json)**

Cada operação na tabela Delta cria um novo arquivo JSON numerado sequencialmente. O nome tem sempre 20 dígitos preenchidos com zeros.

#### Estrutura de um arquivo JSON de commit:

json

```json
{
  "commitInfo": {
    "timestamp": 1701234567890,
    "operation": "WRITE",
    "operationParameters": {
      "mode": "Append",
      "partitionBy": "[]"
    },
    "readVersion": 0,
    "isBlindAppend": true,
    "operationMetrics": {
      "numFiles": "1",
      "numOutputBytes": "1234",
      "numOutputRows": "100"
    }
  }
}
{
  "protocol": {
    "minReaderVersion": 1,
    "minWriterVersion": 2
  }
}
{
  "metaData": {
    "id": "abc123-def456-ghi789",
    "format": {
      "provider": "parquet",
      "options": {}
    },
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"id\",\"type\":\"long\",\"nullable\":true,\"metadata\":{}},{\"name\":\"value\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}",
    "partitionColumns": [],
    "configuration": {},
    "createdTime": 1701234567890
  }
}
{
  "add": {
    "path": "part-00000-abc123.snappy.parquet",
    "partitionValues": {},
    "size": 1234,
    "modificationTime": 1701234567890,
    "dataChange": true,
    "stats": "{\"numRecords\":100,\"minValues\":{\"id\":1},\"maxValues\":{\"id\":100},\"nullCount\":{\"id\":0,\"value\":0}}"
  }
}
```

### 2. **Tipos de Ações em Arquivos JSON**

#### **commitInfo**

- Metadados sobre a operação
- Timestamp, tipo de operação, métricas
- Aparece uma vez por transação

#### **protocol**

- Define versões mínimas do leitor/escritor Delta
- Garante compatibilidade

#### **metaData**

- Schema da tabela
- ID único da tabela
- Colunas de partição
- Configurações

#### **add**

- Adiciona arquivo à tabela
- Path, tamanho, estatísticas
- `dataChange`: true para dados novos, false para OPTIMIZE

#### **remove**

- Remove arquivo da tabela (logicamente)
- Arquivo físico permanece até VACUUM
- Contém timestamp de expiração

json

```json
{
  "remove": {
    "path": "part-00000-old.snappy.parquet",
    "deletionTimestamp": 1701234567890,
    "dataChange": true,
    "extendedFileMetadata": true,
    "partitionValues": {},
    "size": 1234
  }
}
```

#### **txn**

- Informações de transação
- Usado para idempotência

#### **cdc**

- Change Data Capture
- Rastreia mudanças linha por linha

### 3. **Arquivos Checkpoint (.checkpoint.parquet)**

Criados a cada 10 commits por padrão para otimizar leitura do log.

#### Estrutura:

- Formato Parquet (não JSON)
- Contém snapshot completo do estado da tabela
- Inclui todas as ações `add` ativas
- Schema, metadados, configurações
### 4. **Arquivo _last_checkpoint**

Arquivo especial que aponta para o último checkpoint criado.

json

```json
{
  "version": 10,
  "size": 23,
  "parts": 1
}
```

- `version`: número da versão do checkpoint
- `size`: número de ações no checkpoint
- `parts`: número de arquivos do checkpoint (para multi-part checkpoints)

## Fluxo de Leitura do Transaction Log

1. **Leitura Inicial**:
    - Verifica se existe `_last_checkpoint`
    - Se sim, lê o checkpoint indicado
    - Depois lê apenas os JSONs após o checkpoint
2. **Sem Checkpoint**:
    - Lê todos os arquivos JSON desde 00000000000000000000.json
3. **Construção do Estado**:
    - Aplica todas as ações em ordem
    - `add` adiciona arquivos
    - `remove` remove arquivos
    - Resultado: lista atual de arquivos válidos

## Exemplos Práticos

### Após CREATE TABLE:

```
_delta_log/
└── 00000000000000000000.json
    - protocol
    - metaData
    - add (arquivos iniciais)
```

### Após INSERT:

```
_delta_log/
├── 00000000000000000000.json
└── 00000000000000000001.json
    - commitInfo (operation: "WRITE")
    - add (novos arquivos)
```

### Após UPDATE:

```
_delta_log/
├── ...
└── 00000000000000000003.json
    - commitInfo (operation: "UPDATE")
    - remove (arquivos antigos)
    - add (arquivos novos com dados atualizados)
```

### Após DELETE:

```
_delta_log/
├── ...
└── 00000000000000000004.json
    - commitInfo (operation: "DELETE")
    - remove (arquivos que continham linhas deletadas)
    - add (novos arquivos sem as linhas deletadas)
```

### Após OPTIMIZE:

```
_delta_log/
├── ...
└── 00000000000000000005.json
    - commitInfo (operation: "OPTIMIZE")
    - remove (arquivos pequenos, dataChange: false)
    - add (arquivos compactados, dataChange: false)
```

### Após VACUUM:

```
_delta_log/
├── ...
└── 00000000000000000006.json
    - commitInfo (operation: "VACUUM")
    - (não adiciona/remove arquivos do log)
```


# Guia Completo de Concorrência no Delta Lake

## Visão Geral da Concorrência

O Delta Lake implementa **Optimistic Concurrency Control (OCC)** - controle otimista de concorrência. Isso significa que:

- Múltiplas operações podem ler simultaneamente
- Escritas assumem que não haverá conflitos
- Conflitos são detectados apenas no momento do commit
- Se houver conflito, uma das operações precisa ser reexecutada

## Matriz de Compatibilidade de Operações

|Operação 1 ↓ / Operação 2 →|INSERT|UPDATE|DELETE|MERGE|OPTIMIZE|ALTER TABLE|
|---|---|---|---|---|---|---|
|**INSERT**|✅|⚠️|⚠️|⚠️|✅|❌|
|**UPDATE**|⚠️|❌|❌|❌|✅|❌|
|**DELETE**|⚠️|❌|❌|❌|✅|❌|
|**MERGE**|⚠️|❌|❌|❌|✅|❌|
|**OPTIMIZE**|✅|✅|✅|✅|✅|❌|
|**ALTER TABLE**|❌|❌|❌|❌|❌|❌|

- ✅ = Sempre compatível
- ⚠️ = Compatível com condições
- ❌ = Conflito (uma operação falhará)

## Tipos de Conflitos

### 1. **Conflitos de Metadados**

Ocorrem quando operações modificam metadados da tabela simultaneamente.

python

```python
# Thread 1
spark.sql("ALTER TABLE delta_table ADD COLUMN new_col STRING")

# Thread 2 (simultânea)
spark.sql("ALTER TABLE delta_table ADD COLUMN other_col INT")
# ❌ Uma falhará com ConcurrentModificationException
```

### 2. **Conflitos de Dados**

Ocorrem quando operações modificam os mesmos arquivos de dados.

python

```python
# Thread 1
spark.sql("UPDATE delta_table SET value = value * 2 WHERE id > 100")

# Thread 2 (simultânea)
spark.sql("DELETE FROM delta_table WHERE id > 100")
# ❌ Uma falhará - ambas tentam modificar os mesmos arquivos
```

### 3. **Conflitos de Append**

INSERTs geralmente não conflitam entre si (exceto com partições).

python

```python
# Thread 1
spark.sql("INSERT INTO delta_table VALUES (1, 'a'), (2, 'b')")

# Thread 2 (simultânea)
spark.sql("INSERT INTO delta_table VALUES (3, 'c'), (4, 'd')")
# ✅ Ambas terão sucesso - append concorrente é permitido
```

### 4. **Conflitos de Partição**

Operações em partições diferentes geralmente não conflitam.

python

```python
# Thread 1
spark.sql("UPDATE delta_table SET value = 'X' WHERE date = '2024-01-01'")

# Thread 2 (simultânea)
spark.sql("UPDATE delta_table SET value = 'Y' WHERE date = '2024-01-02'")
# ✅ Sucesso - partições diferentes
```

## Detecção de Conflitos

### Como funciona:

1. **Leitura do Estado Inicial**
    
    python
    
    ```python
    # Cada operação lê a versão atual
    version_inicial = get_current_version()  # Ex: versão 10
    ```
    
2. **Execução da Operação**
    
    python
    
    ```python
    # Processa dados baseado na versão 10
    arquivos_para_adicionar = process_data()
    arquivos_para_remover = identify_files_to_remove()
    ```
    
3. **Tentativa de Commit**
    
    python
    
    ```python
    try:
        # Tenta criar 00000000000000000011.json
        commit_transaction(version_inicial + 1)
    except ConcurrentModificationException:
        # Outro processo já criou a versão 11
        retry_or_fail()
    ```
    

## Níveis de Isolamento

### 1. **Serializable (Padrão para Escritas)**

- Escritas são completamente isoladas
- Garante consistência total
- Pode causar mais conflitos

### 2. **Snapshot Isolation (Leituras)**

- Leituras veem um snapshot consistente
- Não bloqueiam escritas
- Time travel usa este isolamento

python

```python
# Leitura com snapshot isolation
df = spark.read.format("delta").option("versionAsOf", 5).load(path)
# Sempre verá a versão 5, independente de escritas concorrentes
```

## Exemplos Práticos de Cenários

### Cenário 1: INSERT Concorrente (Sucesso)

python

```python
# Job 1: ETL de vendas
sales_df.write.format("delta").mode("append").save(delta_path)

# Job 2: ETL de devoluções (simultâneo)
returns_df.write.format("delta").mode("append").save(delta_path)

# ✅ Ambos terão sucesso - appends não conflitam
```

### Cenário 2: UPDATE Concorrente (Conflito)

python

```python
# Job 1: Atualizar preços
spark.sql("""
    UPDATE products 
    SET price = price * 1.1 
    WHERE category = 'electronics'
""")

# Job 2: Atualizar estoque (simultâneo)
spark.sql("""
    UPDATE products 
    SET stock = stock - 10 
    WHERE category = 'electronics'
""")

# ❌ Um falhará - ambos modificam os mesmos arquivos
```

### Cenário 3: OPTIMIZE Durante Escrita (Sucesso)

python

```python
# Job 1: INSERT contínuo
while True:
    new_data.write.format("delta").mode("append").save(delta_path)
    
# Job 2: OPTIMIZE periódico
spark.sql("OPTIMIZE delta_table")

# ✅ Ambos funcionam - OPTIMIZE não conflita com INSERT
```

## Estratégias para Lidar com Concorrência

### 1. **Retry Automático**

python

```python
from delta.tables import DeltaTable
from pyspark.sql.utils import AnalysisException
import time

def write_with_retry(df, path, max_retries=3):
    for attempt in range(max_retries):
        try:
            df.write.format("delta").mode("append").save(path)
            return True
        except AnalysisException as e:
            if "ConcurrentModificationException" in str(e):
                time.sleep(2 ** attempt)  # Backoff exponencial
                continue
            raise
    return False
```

### 2. **Particionamento Inteligente**

python

```python
# Particionar por data reduz conflitos
df.write \
  .format("delta") \
  .partitionBy("date", "hour") \
  .mode("append") \
  .save(delta_path)
```

### 3. **Merge ao Invés de Update/Delete**

python

```python
# MERGE é mais eficiente para upserts
DeltaTable.forPath(spark, delta_path).alias("target") \
  .merge(updates_df.alias("source"), "target.id = source.id") \
  .whenMatchedUpdateAll() \
  .whenNotMatchedInsertAll() \
  .execute()
```

### 4. **Configurações de Concorrência**

python

```python
# Aumentar tentativas de retry
spark.conf.set("spark.databricks.delta.retryWriteConflict.enabled", "true")
spark.conf.set("spark.databricks.delta.retryWriteConflict.limit", "5")

# Configurar intervalo entre tentativas
spark.conf.set("spark.databricks.delta.retryWriteConflict.delay", "100ms")
```

## Padrões Recomendados

### 1. **Streaming + Batch**

python

```python
# Streaming para ingestão
stream_df.writeStream \
  .format("delta") \
  .outputMode("append") \
  .option("checkpointLocation", checkpoint_path) \
  .start(delta_path)

# Batch para manutenção
spark.sql("OPTIMIZE delta_table")
spark.sql("VACUUM delta_table")
```

### 2. **Isolamento por Horário**

python

```python
# ETL de INSERT: 00:00 - 06:00
# OPTIMIZE: 06:00 - 07:00
# UPDATE/MERGE: 07:00 - 23:00
# VACUUM: Domingos 02:00
```

### 3. **Uso de Idempotência**

python

```python
# Usar IDs de transação para evitar duplicatas
df.write \
  .format("delta") \
  .mode("append") \
  .option("txnAppId", "etl_job_001") \
  .option("txnVersion", "20240110_001") \
  .save(delta_path)
```

## Cenários Problemáticos

### 1. **Hot Partitions**

python

```python
# ❌ Problema: Todos escrevem na mesma partição
df.write.partitionBy("date").save(delta_path)  # date = today para todos

# ✅ Solução: Particionar mais granularmente
df.write.partitionBy("date", "hour", "bucket").save(delta_path)
```

### 2. **Long Running Transactions**

python

```python
# ❌ Problema: Transação muito longa
big_df = spark.read.parquet(huge_path)  # 1 hora
processed = complex_transformations(big_df)  # 2 horas
processed.write.format("delta").save(delta_path)  # Conflito provável

# ✅ Solução: Quebrar em batches menores
for batch in batches:
    process_and_write(batch)
```

### 3. **Schema Evolution Concorrente**

python

```python
# ❌ Problema: Múltiplas mudanças de schema
# Job 1: Adiciona coluna A
# Job 2: Adiciona coluna B
# Resultado: Uma falhará

# ✅ Solução: Centralizar mudanças de schema
# Use um job dedicado para DDL
```

## Melhores Práticas

1. **Prefira Appends a Updates** quando possível
2. **Use Particionamento** para reduzir contenção
3. **Implemente Retry Logic** com backoff exponencial
4. **Monitore Conflitos** através de logs e métricas
5. **Separe Workloads** por tipo e horário
6. **Use MERGE** para operações complexas de upsert
7. **Configure Timeouts** apropriados para suas operações
8. **Teste Cenários de Concorrência** em desenvolvimento

## Monitoramento de Conflitos

python

```python
# Query para identificar conflitos frequentes
spark.sql("""
    SELECT 
        operation,
        COUNT(*) as conflict_count,
        AVG(retryCount) as avg_retries
    FROM delta.`path/to/table`
    WHERE error LIKE '%ConcurrentModificationException%'
    GROUP BY operation
    ORDER BY conflict_count DESC
""")
```