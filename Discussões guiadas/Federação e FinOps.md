**Federação** significa **descentralizar a execução mantendo padrões centralizados**. É como um país federativo: cada estado tem autonomia para governar, mas seguem uma constituição comum. Neste documento encontraremos alguns cenários fictícios para ilustrar alguns conceitos e problemas comuns.

### Exemplos Práticos de Federação:

#### 1. **Governança Federada**

**Modelo Centralizado (Tradicional):**


```yaml
Comitê Central de Dados:
- Define TODOS os padrões
- Aprova TODOS os modelos
- Controla TODOS os acessos
- Audita TODAS as mudanças
```

**Modelo Federado (Data Mesh):**

```yaml
Governança Global (Constituição):
- Define: "CPF deve ser criptografado"
- Define: "APIs devem ter versionamento semântico"
- Define: "Dados PII devem ter retention policy"

Governança Local (Execução):
- Domínio Vendas: Implementa criptografia com AWS KMS
- Domínio Sinistros: Implementa com Azure Key Vault
- Domínio Financeiro: Implementa com HSM on-premise

→ Todos seguem a regra, cada um do seu jeito
```

#### 2. **Federação de Ambientes (Dev/Hom/Prod)**

**Centralizado:**

```
┌─────────────────────────────┐
│   DW CENTRAL PRODUÇÃO       │
├─────────────────────────────┤
│   DW CENTRAL HOMOLOGAÇÃO    │
├─────────────────────────────┤
│   DW CENTRAL DESENVOLVIMENTO│
└─────────────────────────────┘

Problema: Todos esperando na fila do mesmo ambiente
```


**Federado:**

```
┌──────────┬──────────┬──────────┬──────────┐
│ VENDAS   │ SINISTROS│ FINANCE  │ CLIENTES │
├──────────┼──────────┼──────────┼──────────┤
│ Prod     │ Prod     │ Prod     │ Prod     │
│ Hom      │ Hom      │ Hom      │ Hom      │
│ Dev      │ Dev      │ Dev      │ Dev      │
└──────────┴──────────┴──────────┴──────────┘

Cada domínio: Próprios ambientes, próprio release cycle
```

#### 3. **Federação de Tecnologia**

**Centralizado:**

```
"Todos devem usar Oracle + Informatica + Tableau"
```

**Federado:**

```yaml
Padrão Global:
- "Dados devem ser acessíveis via API"
- "Métricas devem estar no formato Prometheus"
- "Logs devem seguir estrutura JSON"

Implementação por Domínio:
- Vendas: PostgreSQL + Airflow + API Gateway
- Sinistros: MongoDB + Spark + GraphQL
- Financeiro: Oracle + SSIS + REST

→ Todos interoperáveis, cada um com sua stack
```

### Outros Exemplos de Federação:

**Federação de Segurança:**

- Global: "Autenticação via SSO corporativo"
- Local: Cada domínio define suas roles e permissões

**Federação de Qualidade:**

- Global: "SLA mínimo de 99.5% uptime"
- Local: Cada domínio decide como alcançar (replica, cache, etc)

**Federação de Metadados:**

- Global: "Catálogo central de descoberta"
- Local: Cada domínio documenta seus próprios dados

## O cuidado com centros de custo

Muitas vezes pensamos em separar a arquitetura em domínios por conta da aparente facilidade em ratear custos de desenvolvimento e manutenção dos "domínios", porém, existem algumas surpresas que podem aparecer mais cedo ou mais tarde caso a arquitetura não seja planejada vendo a questão de custos (tanto o que gasta o que, quanto quem paga o que) com outros olhos. Um exemplo é a questão da incidência de custo no storage account de origem, não de armazenamento, mas de incidência de chamadas de outros domínios. Mesmo que consideremos isto como custo de operação do domínio que paga pelo storage account de origem, deve-se olhar o problema por outro ângulo e entender que a falta de otimização do consumo dos outros domínios terá impacto direto neste "custo operacional".
A seguir, alguns cenários ilustrativos de como a divisão de custos pode ser feita fugindo do tradicional dashboard de Budget dos recursos (é comum domínios diferentes pensarem em soluções completamente originais para este cenário):

### Exemplo 1: **"Commons" (Bem Comum)**

Conceito: Custos de infraestrutura compartilhada são rateados (leve em conta de que não necessariamente o custo será proporcional ao trafego)

- Implementação:
	- Storage Account do Domínio A: R$ 10.000/mês
	- 30% do tráfego vem do Domínio B
	- 20% do tráfego vem do Domínio C
	- 50% uso interno do Domínio A

- Rateio mensal:
	- Domínio A paga: R$ 5.000
	- Domínio B paga: R$ 3.000
	- Domínio C paga: R$ 2.000

- Ferramentas:
	- AWS Cost Explorer com tags
	- Azure Cost Management
	- GCP Billing por labels

### Modelo 2: **"Chargeback" (Cobrança Interna)**

Conceito: Podemos encapsular o ponto de consumo do dado sendo exposto como produto de modo a precificar chamadas, simulando um modelo "pay-as-you-go"

```python
# Pseudo-código de chargeback
class DataProductPricing:
    def __init__(self, domain):
        self.base_cost = calculate_infrastructure_cost()
        self.ops_cost = calculate_team_cost()
        self.margin = 0.2  # 20% para inovação
    
    def price_per_call(self):
        total_cost = self.base_cost + self.ops_cost
        return (total_cost * (1 + self.margin)) / expected_calls

# Exemplo:
customer_360_api.price = R$ 0.05 por chamada
risk_score_api.price = R$ 0.50 por chamada (mais complexo)
```

### Modelo 3: **"Sponsorship" (Patrocínio)**

Conceito: Casos de uso estratégicos são patrocinados:

- Customer 360 View:
	- Custo: R$ 50k/mês
	- Valor estratégico: Crítico para toda empresa
	- Decisão: CEO patrocina do budget  corporativo

- Fraud Detection:
	- Custo: R$ 100k/mês  
	- ROI: Evita R$ 1M/mês em fraudes
	- Decisão: CFO patrocina do budget de riscos


### Modelo 4: **"Value-Based" (Baseado em Valor)**

Conceito: Domínios "clientes" pagam uma porcentagem dos ganhos que o produto de dado gerou

- Domínio A (Clientes) produz: Customer 360
	- Custo real: R$ 30k/mês
- Consumidores e valor gerado:
	- Vendas: +R$ 500k/mês em conversão
	- Sinistros: -R$ 300k/mês em fraudes evitadas  
	- Marketing: +R$ 200k/mês em campanhas efetivas
- Modelo de cobrança:
	- Vendas paga: 5% do valor = R$ 25k
	- Sinistros paga: 3% do valor = R$ 9k
	- Marketing paga: 3% do valor = R$ 6k
	- Total: R$ 40k (cobre custos + inovação)

### Modelos Organizacionais:

#### **Modelo "Netflix"**

- Cada domínio tem budget próprio
- Podem "comprar" de outros domínios
- Incentiva produtos de qualidade (senão ninguém compra)
	- Não existe barreira física que impede a duplicação de um produto, ou seja, se o produto de dado atual não tiver qualidade e a demanda que o envolva for prioridade, chances são de que será iniciado o desenvolvimento de um novo produto com o mesmo propósito, infelizmente

#### **Modelo "Spotify"**

- Squads têm budget compartilhado por tribo
- Custos de plataforma são "subsidiados"
- Foco em valor de negócio, não em custos internos

#### **Modelo "Amazon"**

- Tudo é cobrado (até emails internos!)
- APIs são produtos com preço
- Força eficiência extrema

### Exemplo de implementação prática gradual:


- **Fase 1 (Meses 1-3)**: Visibilidade
	- Implementar tagging/labeling
	- Criar dashboards de uso
	- Sem cobrança ainda
- **Fase 2 (Meses 4-6)**: Showback
	- Mostrar quanto cada um custaria
	- Não cobrar ainda
	- Educar sobre custos
- **Fase 3 (Meses 7-9)**: Chargeback Parcial
	- Cobrar 50% dos custos reais
	- Corporate subsidia 50%
	- Ajustar modelos
- **Fase 4 (Meses 10-12)**: Modelo Completo
	- Implementar modelo escolhido
	- Ajustar budgets dos domínios
	- Monitorar e otimizar

O segredo é que **não existe modelo perfeito** - cada empresa precisa encontrar o equilíbrio entre:

- Transparência de custos
- Incentivo à colaboração
- Simplicidade operacional
- Alinhamento com cultura corporativa

### Perguntas para exercitar o raciocínio:

1. **"Qual modelo de cobrança incentivaria mais inovação vs eficiência?"**
2. **"Como evitar que domínios 'pobres' não consigam consumir dados essenciais?"**
3. **"Deve haver dados 'públicos' que ninguém paga para usar?"**
4. **"Como precificar dados que só têm valor quando combinados?"**
