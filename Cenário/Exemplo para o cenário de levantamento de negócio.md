
Os tópicos discutidos neste documento tem o objetivo de ser um exemplo didático para ilustrar de maneira ilustrativa os levantamentos iniciais de negócio em uma seguradora fictícia. Lembrem-se de que as estruturas de negócio e arquiteturas de dados variam muito de caso a caso em cenários reais. O arquivo [[Exemplo para o cenário de levantamento de negócio]] fornece um pouco mais de aprofundamento nos assuntos que abordaremos a seguir.

# Visão geral de estrutura de dados e domínios de negócio em seguradoras

 **O setor de seguros do exemplo irá operar com nove domínios principais**, onde dados de sinistros alimentam subscrição, informações de clientes permeiam vendas e produtos, criando uma teia complexa de dependências que torna arquiteturas centralizadas tradicionais inadequadas para organizações de grande porte (mais em [[A escalabilidade da arquitetura centralizada]]).

## Principais domínios de negócio encontrados em seguradoras

**Sinistros (Claims Processing)**: representa o coração operacional das seguradoras, gerenciando todo o ciclo desde o primeiro aviso de sinistro até o fechamento. Este domínio processa dados de identificação do sinistro (número único, status, tipo de perda), informações financeiras (valor reservado, pago, franquia), dados do reclamante (informações pessoais, relação com segurado, detalhes de danos) e documentação externa (boletins policiais, laudos médicos, orçamentos). O processo engloba desde a recepção do aviso até investigação, avaliação, negociação e pagamento, incluindo detecção de fraudes e recuperação via sub-rogação.

**Subscrição/Underwriting**: foca na avaliação de riscos e precificação de apólices. Os dados incluem fatores de risco e variáveis de classificação, histórico de sinistros, informações financeiras e de crédito, detalhes de propriedades (localização, construção, idade) e informações pessoais (idade, ocupação, estilo de vida). Fontes externas como registros de veículos, relatórios médicos, informações de bureaus de crédito e dados telemáticos alimentam modelos de scoring automatizados que determinam aceitação, preços e condições de cobertura.

**Vendas e Distribuição**: gerencia leads, cotações e conversões através de múltiplos canais. Dados de prospects incluem informações demográficas, fonte do lead, nível de interesse e preferências de produtos. Informações de cotações cobrem opções de cobertura, preços, dados comparativos e taxas de conversão. Métricas de canais (direto, corretor, online) e performance de campanhas direcionam estratégias de aquisição e retenção de clientes.

**Gestão de Produtos**: desenvolve e mantém o portfólio de seguros, processando especificações de cobertura, estruturas de preços, fatores de classificação, termos e condições, além de requisitos regulatórios. Dados de performance incluem volumes de vendas, índices de sinistralidade, participação de mercado e satisfação do cliente, alimentando análises de lucratividade que orientam modificações e lançamentos de produtos.

**Gestão de Clientes/Segurados**: centraliza relacionamentos e dados pessoais, incluindo informações demográficas completas, relacionamentos familiares, beneficiários, histórico de pagamentos e preferências de comunicação. Este domínio integra interações de atendimento, histórico de reclamações, pesquisas de satisfação e métricas de engajamento digital para criar visões 360° dos clientes.

Os domínios complementares incluem **Atuarial/Gestão de Riscos** (modelos estatísticos, reservas, precificação), **Financeiro/Contabilidade** (demonstrações financeiras, investimentos, comissões), **Regulatório/Compliance** (relatórios, licenças, auditoria) e **Gestão de Corretores/Agentes** (performance, comissões, treinamento).

## Tipos de dados e campos por domínio

### Domínio de sinistros
Os dados estruturados incluem identificadores únicos (claim_id, claim_number, policy_id), datas críticas (data_ocorrencia, data_aviso, data_fechamento), valores financeiros (valor_sinistro, valor_pago, valor_reserva, franquia) e classificações (tipo_sinistro, causa_perda, status_processamento). Dados relacionais conectam segurados, beneficiários, prestadores de serviço e reguladores. Informações não estruturadas incluem fotos, vídeos, documentos médicos, laudos técnicos e depoimentos de testemunhas.

### Domínio de subscrição
Campos de avaliação de risco incluem pontuação de crédito, histórico de sinistros, fatores demográficos (idade, sexo, estado_civil, profissao), dados de propriedade (endereco, tipo_construcao, ano_construcao, valor_segurado) e informações de veículos (make, model, year, vin). Dados de decisão capturam accept_decline_flag, risk_score, premium_amount, coverage_limits e razões de exceções ou recusas.

### Domínio de vendas
Lead tracking inclui fonte_lead, canal_origem, data_contato, status_qualificacao e probabilidade_conversao. Dados de cotação abrangem quote_number, data_expiracao, opcoes_cobertura, valores_premium e taxas de conversão por canal. Métricas de performance incluem custo_aquisicao_cliente, lifetime_value e NPS por segmento.

### Domínio de produtos
Especificações técnicas incluem product_code, linha_negocio, tipo_cobertura, limites_cobertura, exclusoes e condicoes_especiais. Dados de performance abrangem volumes_vendas, loss_ratio, market_share, customer_satisfaction e análises de lucratividade por segmento.

### Domínio de clientes
Master data inclui customer_id, informações pessoais (nome, cpf, endereco, telefone, email), dados demográficos (data_nascimento, genero, renda, escolaridade) e preferências (canal_comunicacao, frequency_contato, idioma_preferido). Dados relacionais conectam beneficiários, dependentes e procuradores, enquanto dados comportamentais incluem digital_engagement, channel_usage e service_interaction_history.


## Red Flags para Data Mesh

Durante o mapeamento, destacar situações que indicam necessidade de domínios separados como:

- "Essa informação é atualizada por equipes diferentes?"
- "Existem regras de negócio conflitantes para o mesmo dado?"
- "A frequência de atualização é muito diferente?"
- "Quem é o 'dono' dessa informação na organização?"
- 
## Sobreposições entre domínios justificando data mesh

Muitas vezes times diferentes competem pelo ownership do mesmo dado. Diversas necessidades diferentes de níveis de atuações diferentes fazem com que seja necessário o agrupamento destes times em um "grande time conceitual" (domínios).

### Interdependência sinistros-subscrição
A **experiência de sinistros alimenta diretamente modelos de subscrição** através de loops de feedback contínuos. Padrões de frequência e severidade de sinistros por segmento demográfico ou geográfico informam ajustes em fatores de rating e diretrizes de aceitação. Dados de performance de sinistros permitem refinamento de modelos preditivos em tempo real, enquanto informações de subscrição são essenciais para validação de coberturas durante processamento de sinistros. Esta interdependência cria necessidade de acesso bidirecional em tempo real que arquiteturas centralizadas não conseguem suportar eficientemente.

### Visão 360° do cliente entre múltiplos domínios
**Dados de clientes fragmentados** através de administração de apólices, processamento de sinistros, billing e atendimento criam desafios para experiências omnichannel. Informações de interação com call center precisam estar disponíveis durante processamento de sinistros, enquanto histórico de sinistros influencia estratégias de retenção e cross-selling. Preferências de comunicação capturadas em um canal devem refletir imediatamente em todos os pontos de contato, exigindo sincronização em tempo real entre domínios.

### Pricing dinâmico requerendo integração vendas-subscrição
**Cotações em tempo real** para canais digitais demandam acesso instantâneo a regras de subscrição, capacidade de mercado, dados de comportamento do cliente e inteligência competitiva. Feedback de objeções de clientes durante vendas precisa informar ajustes de pricing e critérios de aceitação rapidamente. Esta necessidade de responsividade cria gargalos em arquiteturas centralizadas onde equipes de dados não conseguem atender demandas de múltiplos domínios simultaneamente.

### Detecção de fraudes cross-domain
**Identificação de fraudes** requer análise integrada de padrões de sinistros, comportamento de clientes, dados de vendas e informações de terceiros. Alertas de fraude gerados durante processamento de sinistros devem trigger verificações em todo o portfólio do cliente e histórico de interações. Esta necessidade de análise em tempo real através de múltiplos domínios justifica arquiteturas descentralizadas onde cada domínio pode processar e compartilhar insights de fraude autonomamente.

## Exemplo de estruturas típicas de dados principais

### Estrutura de apólices
```sql
POLICY (
  policy_id PK,
  policy_number UNIQUE,
  client_id FK,
  product_id FK,
  effective_date DATE,
  expiration_date DATE,
  policy_status VARCHAR(20),
  premium_amount DECIMAL(15,2),
  coverage_amount DECIMAL(15,2),
  deductible_amount DECIMAL(10,2),
  agent_id FK,
  broker_id FK,
  created_date TIMESTAMP,
  modified_date TIMESTAMP
)

POLICY_COVERAGE (
  coverage_id PK,
  policy_id FK,
  coverage_type VARCHAR(50),
  coverage_limit DECIMAL(15,2),
  coverage_deductible DECIMAL(10,2),
  coverage_premium DECIMAL(10,2)
)
```

### Estrutura de sinistros
```sql
CLAIMS (
  claim_id PK,
  claim_number UNIQUE,
  policy_id FK,
  claim_date DATE,
  loss_date DATE,
  claim_type VARCHAR(50),
  claim_status VARCHAR(20),
  claim_amount DECIMAL(15,2),
  estimated_amount DECIMAL(15,2),
  paid_amount DECIMAL(15,2),
  reserve_amount DECIMAL(15,2),
  adjuster_id FK,
  created_date TIMESTAMP
)

CLAIM_PAYMENTS (
  payment_id PK,
  claim_id FK,
  payment_date DATE,
  payment_amount DECIMAL(15,2),
  payment_type VARCHAR(30),
  payee_information TEXT
)
```

### Estrutura de prêmios
```sql
PREMIUM_CALCULATION (
  calculation_id PK,
  policy_id FK,
  base_premium DECIMAL(12,2),
  risk_factors JSON,
  discounts_applied JSON,
  taxes_fees DECIMAL(8,2),
  total_premium DECIMAL(15,2),
  calculation_date TIMESTAMP
)

BILLING (
  bill_id PK,
  policy_id FK,
  billing_period_start DATE,
  billing_period_end DATE,
  premium_amount DECIMAL(15,2),
  fees_amount DECIMAL(8,2),
  total_amount DECIMAL(15,2),
  due_date DATE,
  payment_status VARCHAR(20)
)
```

### Estrutura de comissões
```sql
COMMISSION (
  commission_id PK,
  policy_id FK,
  agent_id FK,
  broker_id FK,
  commission_type VARCHAR(30),
  commission_rate DECIMAL(5,4),
  commission_amount DECIMAL(12,2),
  earned_date DATE,
  paid_date DATE,
  commission_status VARCHAR(20)
)

COMMISSION_SPLIT (
  split_id PK,
  commission_id FK,
  recipient_id FK,
  recipient_type VARCHAR(20),
  split_percentage DECIMAL(5,2),
  split_amount DECIMAL(12,2)
)
```


### Governança de dados específica para seguros
**Framework de governança** estabelece conselho executivo com sponsors C-level (rank alto, CEO, CFO, etc), stewards de domínio especialistas em subscrição/sinistros/atuarial, equipes técnicas e compliance officers. **Políticas essenciais** incluem classificação de dados por sensibilidade, controles de acesso baseados em papéis, padrões de qualidade de dados e procedimentos de gestão de mudanças.

### Compliance regulatório automatizado
**Controles de privacidade** implementam criptografia AES-256, controles de acesso baseados em papéis, mascaramento dinâmico de dados e técnicas de preservação de privacidade como tokenização e anonimização. **Monitoramento contínuo** inclui alertas de segurança em tempo real, dashboards de compliance regulatório e detecção automatizada de violações de políticas.

### Integração com sistemas core

**Padrões de integração** combinam batch processing para cargas históricas, streaming em tempo real para detecção de fraudes e underwriting, e APIs híbridas para servicing de apólices. **Sistemas fonte** incluem administração de apólices via APIs SOAP/REST, gestão de sinistros através de streaming de eventos, sistemas de billing via extratos programados e CRM através de webhooks.



## Exemplo de entidades do negócio

### 1. Ponto de Partida: Entidades-Raiz (Core Entities)

#### **[[Roteiros/Discussões guiadas/mesh/tabelas/Clientes]]** - A base de tudo

Começaria aqui porque:

- É a entidade mais estável e independente
- Todas as outras entidades se relacionam com ela
- Define o "quem" do negócio

**Perguntas-chave para os alunos terem dúvidas:**

- "O que caracteriza um cliente para a seguradora?"
- "Quais tipos de clientes existem?" (PF, PJ, Corretor, Beneficiário)
- "Um prospect é um cliente? E um beneficiário?"
- "Como identificamos unicamente um cliente?"

#### **[[Produtos]]** - O que vendemos

Segunda prioridade porque:

- Define as regras de negócio
- Independe de transações específicas
- É relativamente estável

**Perguntas-chave para questionar a definição desta entidade:**

- "Quais linhas de produtos a seguradora oferece?"
- "O que diferencia um produto de outro?"
- "Existem produtos compostos? Pacotes?"
- "Como os produtos são categorizados?"

### 2. Entidades Transacionais (Process Entities)

#### **Cotação/Proposta** - O início da jornada

Terceira etapa porque:

- Conecta Cliente e Produto
- Primeira interação transacional
- Gera dados valiosos mesmo sem conversão

**Perguntas-chave:**

- "Qual a diferença entre cotação e proposta?"
- "Uma cotação pode ter múltiplos produtos?"
- "Por quanto tempo uma cotação é válida?"
- "Quais dados são obrigatórios vs opcionais?"

#### **Apólice** - O contrato efetivo

Quarta etapa porque:

- Evolução natural da proposta
- Centro de várias operações
- Conecta-se com pagamentos e sinistros

**Perguntas-chave:**

- "O que transforma uma proposta em apólice?"
- "Uma apólice pode cobrir múltiplos riscos/bens?"
- "Como tratamos renovações? São novas apólices?"
- "Existe hierarquia entre apólices?"

## 3. Entidades Operacionais (Operational Entities)

#### **Sinistro** - A materialização do risco

- Depende da existência de uma apólice ativa
- Gera o maior volume de interações
- Conecta-se com pagamentos e terceiros

#### **Pagamento/Prêmio** - O fluxo financeiro

- Relaciona-se com apólices e sinistros
- Tem regras complexas de parcelamento
- Gera comissionamento

## 4. Entidades de Suporte (Supporting Entities)

#### **Risco/Bem Segurado** - O objeto da proteção

- Pode ser um carro, casa, vida, etc.
- Tem características específicas por tipo
- Influencia pricing e aceitação

#### **Cobertura** - O que está protegido

- Define limites e exclusões
- Varia por produto e risco
- Essencial para sinistros



### 5. Indicadores de separação de domínios

**Domínio de Clientes:**

- Entidades: Cliente, Endereço, Contato, Beneficiários
- Atualização: Marketing, Atendimento, Vendas
- Frequência: Baixa (exceto interações)

**Domínio de Produtos:**

- Entidades: Produto, Coberturas Padrão, Regras
- Atualização: Produto, Atuarial, Compliance
- Frequência: Muito baixa

**Domínio de Vendas:**

- Entidades: Cotação, Proposta, Comissões
- Atualização: Vendas, Corretores, Digital
- Frequência: Alta

**Domínio de Subscrição:**

- Entidades: Análise de Risco, Parecer, Regras
- Atualização: Underwriting, Atuarial
- Frequência: Média

**Domínio de Apólices:**

- Entidades: Apólice, Riscos, Coberturas, Endossos
- Atualização: Emissão, Renovação
- Frequência: Média

**Domínio de Sinistros:**

- Entidades: Sinistro, Pagamentos, Regulação
- Atualização: Sinistros, Reguladores, Jurídico
- Frequência: Alta

**Domínio Financeiro:**

- Entidades: Pagamentos, Comissões, Cobrança
- Atualização: Financeiro, Cobrança
- Frequência: Alta


# Perguntas e exercícios para exercitar o raciocínio

### Estratégia de Descoberta 

**Exercício 1 - Mapeamento Visual:** "Desenhem as principais entidades que vocês acreditam existir em uma seguradora e suas relações"

**Exercício 2 - Validação por Cenários:**

- "João quer fazer um seguro de carro" → Quais entidades são criadas/consultadas?
- "Maria bateu o carro e precisa acionar o seguro" → Qual o fluxo de entidades?
- "A empresa ABC quer segurar sua frota" → O que muda no modelo?

**Exercício 3 - Identificação de Atributos:** Para cada entidade identificada, perguntar:

- Quais são os identificadores únicos?
- Quais dados são obrigatórios?
- Quais se relacionam com outras entidades?
- Quais mudam com frequência vs. são estáveis?

### Perguntas reveladoras para os Alunos

Estes exemplos de perguntas naturalmente levarão a identificar os pontos de conflito e sobreposição que justificam uma arquitetura Data Mesh:

1. **Sobre Ownership:**
    - "Quando um cliente muda de endereço, quantas áreas precisam ser notificadas?"
    - "Quem decide se um sinistro deve ser pago - sinistros ou financeiro?"
2. **Sobre Conflitos:**
    - "O valor do prêmio calculado em vendas é sempre o mesmo aprovado em subscrição?"
    - "Como resolvem quando o cliente informa dados diferentes para vendas e sinistros?" 
3. **Sobre Temporalidade:**
    - "Com que frequência mudam as regras de aceitação de um produto?"
    - "Quanto tempo leva entre uma cotação e a emissão da apólice?"
4. **Sobre Volume:**
    - "Quantas cotações são feitas vs. quantas viram apólices?"
    - "Qual a proporção de sinistros por apólice ativa?"
5. **Sobre Integração:**
    - "Como o sistema sabe que pode renovar uma apólice?"
    - "Como identificam tentativas de fraude entre múltiplas apólices?"
