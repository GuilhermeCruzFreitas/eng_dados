
# Resumo
## Conceitos Fundamentais:

### 1. **Leitura de Baixo para Cima**

- Sempre comece pela fonte de dados (FileScan)
- Suba seguindo o fluxo de transformações
- Termine no resultado final

### 2. **Elementos Críticos**

- **`*(n)`**: WholeStageCodegen - operações compiladas juntas
- **Exchange**: Shuffle de dados (operação cara)
- **FileScan**: Como os dados são lidos
- **Join Types**: Broadcast vs SortMerge vs NestedLoop

### 3. **Red Flags**

- **CartesianProduct**: Join sem condição (N×M)
- **Multiple Exchanges**: Muitos shuffles
- **SinglePartition**: Todos dados numa partição
- **BroadcastNestedLoopJoin**: Join ineficiente

## Exemplo Prático:

python

```python
# Ver o plano
df.explain(True)  # Todos os planos
df.explain("formatted")  # Mais legível
df.explain("cost")  # Com estatísticas
```

### Interpretando um Plano:

```
*(2) HashAggregate(keys=[city#123], functions=[avg(age#124)])
+- Exchange hashpartitioning(city#123, 200)
   +- *(1) HashAggregate(keys=[city#123], functions=[partial_avg(age#124)])
      +- *(1) Filter (age#124 > 25)
         +- FileScan parquet [city#123,age#124]
```

**Leitura**:

1. Lê arquivo Parquet (apenas colunas necessárias)
2. Filtra age > 25
3. Agregação parcial (local)
4. Shuffle por cidade
5. Agregação final

## Otimizações Comuns:

1. **Predicate Pushdown**: Filtros no storage
2. **Column Pruning**: Ler apenas colunas necessárias
3. **Broadcast Joins**: Para tabelas pequenas
4. **Partition Pruning**: Ler apenas partições necessárias


# Completo
## O que é um Physical Plan?

O Physical Plan é a representação final de como o Spark executará sua query. É o resultado de:

1. **Logical Plan**: O que você quer fazer (SQL/DataFrame API)
2. **Optimized Logical Plan**: Otimizações aplicadas pelo Catalyst
3. **Physical Plan**: Como será executado (operadores RDD)

## Como Visualizar Physical Plans

python

```python
# Criar um DataFrame de exemplo
df = spark.read.parquet("path/to/data")
filtered_df = df.filter(col("age") > 25).select("name", "age", "city")
grouped_df = filtered_df.groupBy("city").agg(avg("age").alias("avg_age"))

# Visualizar o Physical Plan
grouped_df.explain(True)  # True = mostra todos os planos
grouped_df.explain("formatted")  # Formato mais legível
grouped_df.explain("cost")  # Inclui estatísticas de custo
```

## Estrutura de um Physical Plan

```
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=true
+- == Final Plan ==
   *(2) HashAggregate(keys=[city#123], functions=[avg(age#124)])
   +- Exchange hashpartitioning(city#123, 200), ENSURE_REQUIREMENTS
      +- *(1) HashAggregate(keys=[city#123], functions=[partial_avg(age#124)])
         +- *(1) Project [city#123, age#124]
            +- *(1) Filter (age#124 > 25)
               +- *(1) ColumnarToRow
                  +- FileScan parquet [name#122,age#124,city#123] 
                     Batched: true, DataFilters: [age#124 > 25], 
                     Format: Parquet, Location: InMemoryFileIndex[path/to/data],
                     PartitionFilters: [], PushedFilters: [GreaterThan(age,25)]
```

## Lendo de Baixo para Cima

Physical Plans devem ser lidos **de baixo para cima** - da fonte de dados até o resultado final.

### 1. **FileScan** (Base)

```
FileScan parquet [name#122,age#124,city#123]
```

- **Operação**: Leitura de arquivos
- **Formato**: Parquet
- **Colunas**: Apenas as necessárias (Column Pruning)
- **PushedFilters**: Filtros empurrados para o storage

### 2. **ColumnarToRow**

```
+- *(1) ColumnarToRow
```

- Converte dados colunares (Parquet) para formato linha
- Necessário para processamento row-based do Spark

### 3. **Filter**

```
+- *(1) Filter (age#124 > 25)
```

- Aplica filtro nos dados
- *(1) indica WholeStageCodegen

### 4. **Project**

```
+- *(1) Project [city#123, age#124]
```

- Seleciona apenas colunas necessárias
- Reduz dados em memória

### 5. **HashAggregate (Partial)**

```
+- *(1) HashAggregate(keys=[city#123], functions=[partial_avg(age#124)])
```

- Agregação parcial local (map-side)
- Reduz dados antes do shuffle

### 6. **Exchange**

```
+- Exchange hashpartitioning(city#123, 200), ENSURE_REQUIREMENTS
```

- **Shuffle**: Redistribui dados entre executors
- **Particionamento**: Por hash da cidade
- **200**: Número de partições (spark.sql.shuffle.partitions)

### 7. **HashAggregate (Final)**

```
*(2) HashAggregate(keys=[city#123], functions=[avg(age#124)])
```

- Agregação final após shuffle
- Combina resultados parciais

## Elementos Importantes

### 1. **WholeStageCodegen (*)**

```
*(1) Filter...
*(2) HashAggregate...
```

- Número indica diferentes stages de codegen
- Múltiplas operações compiladas juntas
- Melhora performance significativamente

### 2. **Exchange (Shuffle)**

```
Exchange hashpartitioning(column, 200)
Exchange SinglePartition
Exchange rangepartitioning(column ASC, 200)
```

- **hashpartitioning**: Para GROUP BY
- **SinglePartition**: Collect para driver
- **rangepartitioning**: Para ORDER BY

### 3. **Join Types**

```
BroadcastHashJoin [id#1], [id#2], Inner, BuildRight
SortMergeJoin [id#1], [id#2], Inner
BroadcastNestedLoopJoin BuildRight, Inner
```

- **BuildRight/BuildLeft**: Qual lado é construído na memória
- **Broadcast**: Dataset pequeno enviado para todos executors
- **SortMerge**: Datasets grandes, requer sort

### 4. **Adaptive Query Execution (AQE)**

```
AdaptiveSparkPlan isFinalPlan=true
+- == Final Plan ==
   ...
+- == Initial Plan ==
   ...
```

- Otimização runtime baseada em estatísticas reais
- Pode mudar joins, coalescer partições, etc.

## Exemplos Práticos

### Exemplo 1: Join com Broadcast

python

```python
small_df = spark.range(1000).withColumnRenamed("id", "small_id")
large_df = spark.range(1000000).withColumnRenamed("id", "large_id")
result = large_df.join(small_df, large_df.large_id == small_df.small_id)
result.explain(True)
```

```
== Physical Plan ==
*(2) BroadcastHashJoin [large_id#2L], [small_id#0L], Inner, BuildRight, false
:- *(2) Range (0, 1000000, step=1, splits=8)
+- BroadcastExchange HashedRelationBroadcastMode(List(input[0, bigint, false]),false)
   +- *(1) Range (0, 1000, step=1, splits=8)
```

**Interpretação**:

- DataFrame pequeno é broadcast (BuildRight)
- Sem shuffle do DataFrame grande
- Muito eficiente

### Exemplo 2: Join com SortMerge

python

```python
df1 = spark.range(10000000).withColumnRenamed("id", "id1")
df2 = spark.range(10000000).withColumnRenamed("id", "id2")
result = df1.join(df2, df1.id1 == df2.id2)
result.explain(True)
```

```
== Physical Plan ==
*(5) SortMergeJoin [id1#10L], [id2#12L], Inner
:- *(2) Sort [id1#10L ASC NULLS FIRST], false, 0
:  +- Exchange hashpartitioning(id1#10L, 200), ENSURE_REQUIREMENTS
:     +- *(1) Range (0, 10000000, step=1, splits=8)
+- *(4) Sort [id2#12L ASC NULLS FIRST], false, 0
   +- Exchange hashpartitioning(id2#12L, 200), ENSURE_REQUIREMENTS
      +- *(3) Range (0, 10000000, step=1, splits=8)
```

**Interpretação**:

- Ambos datasets são shuffled
- Ordenados antes do join
- Mais custoso que broadcast

### Exemplo 3: Aggregation com Multiple Stages

python

```python
df = spark.read.parquet("sales_data")
result = df.groupBy("product", "store").agg(
    sum("quantity").alias("total_qty"),
    avg("price").alias("avg_price")
).orderBy("total_qty", ascending=False)
result.explain(True)
```

```
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=true
+- == Final Plan ==
   *(3) Sort [total_qty#123 DESC NULLS LAST], true, 0
   +- Exchange rangepartitioning(total_qty#123 DESC NULLS LAST, 200)
      +- *(2) HashAggregate(keys=[product#10, store#11], 
                          functions=[sum(quantity#12), avg(price#13)])
         +- Exchange hashpartitioning(product#10, store#11, 200)
            +- *(1) HashAggregate(keys=[product#10, store#11], 
                                functions=[partial_sum(quantity#12), partial_avg(price#13)])
               +- *(1) FileScan parquet [product#10,store#11,quantity#12,price#13]
```

**Interpretação**:

- Agregação em 2 fases (parcial + final)
- 2 shuffles: um para GROUP BY, outro para ORDER BY
- AQE pode otimizar número de partições

## Red Flags no Physical Plan

### 1. **CartesianProduct**

```
CartesianProduct Inner
```

- Join sem condição
- Extremamente caro (N×M registros)
- Geralmente indica erro na query

### 2. **BroadcastNestedLoopJoin**

```
BroadcastNestedLoopJoin BuildRight, Inner
```

- Join sem condição de igualdade
- Menos eficiente que hash joins
- Revisar lógica do join

### 3. **Exchange SinglePartition**

```
Exchange SinglePartition
```

- Todos dados vão para uma partição
- Pode causar OOM no driver
- Comum com collect() ou toPandas()

### 4. **Múltiplos Exchanges**

```
Exchange hashpartitioning...
Exchange hashpartitioning...
Exchange hashpartitioning...
```

- Múltiplos shuffles custosos
- Considerar reordenar operações
- Verificar particionamento

## Otimizações Baseadas no Plan

### 1. **Predicate Pushdown**

```
# Verificar se filtros foram empurrados
FileScan parquet ... PushedFilters: [IsNotNull(age), GreaterThan(age,25)]
```

### 2. **Column Pruning**

```
# Verificar se apenas colunas necessárias são lidas
FileScan parquet [col1#1, col2#2]  # Bom
FileScan parquet [*]  # Ruim - lendo todas colunas
```

### 3. **Broadcast Join Threshold**

python

```python
# Ajustar threshold para broadcast
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "100MB")
```

### 4. **Partition Pruning**

```
PartitionFilters: [isnotnull(date#20), (date#20 >= 2024-01-01)]
```

## Análise de Performance

### Checklist de Análise:

1. **Quantos Exchanges (shuffles)?** Menos é melhor
2. **Tipos de Join?** Broadcast > SortMerge > NestedLoop
3. **WholeStageCodegen ativo?** *(n) indica sim
4. **Filtros pushdown?** Verificar PushedFilters
5. **Colunas desnecessárias?** Verificar FileScan
6. **Partições adequadas?** 200 pode não ser ideal

### Métricas no Spark UI:

- **Shuffle Read/Write**: Volume de dados shuffled
- **Spill to Disk**: Indica memória insuficiente
- **Task Duration**: Identifica skew
- **GC Time**: Problema de memória

## Dicas Práticas

1. **Compare Plans**: Antes e depois de otimizações
2. **Use AQE**: Adaptive Query Execution melhora plans
3. **Cache Estratégico**: Evita recomputação
4. **Estatísticas**: ANALYZE TABLE para melhores plans
5. **Monitoramento**: Spark UI complementa o plan

O Physical Plan é sua janela para entender exatamente como o Spark executará seu código. Dominar sua leitura é essencial para otimização de performance!