# Data Contract - [[Data product - Lifecycle Events]]

## O que é um Data Contract?

Um **Data Contract** é um acordo formal entre produtores e consumidores de dados que define:

- **O QUE** será entregue (schema, formato, campos)
- **COMO** será entregue (APIs, eventos, arquivos)
- **QUANDO** será entregue (SLA, frequência)
- **QUALIDADE** esperada (completude, acurácia)
- **GARANTIAS** oferecidas (versionamento, breaking changes)

É como uma "API para dados" - um compromisso público do que seu data product oferece.

## O que é "Escrito em Pedra" vs Flexível

### Imutável (Definido pela Arquitetura Data Mesh)

```yaml
Princípios Fundamentais:
  - Ownership: Domínio de Apólices é dono dos dados
  - Interoperabilidade: Usar padrões da empresa
  - Self-service: Consumidores devem conseguir usar sem ajuda
  - Governança Federada: Seguir políticas globais

Padrões Técnicos Corporativos:
  - Formato de eventos: JSON com schema registry
  - Protocolo de streaming: Event Hubs
  - API Gateway: Azure API Management
  - Autenticação: Azure AD / Service Principal
  - Classificação de dados: PII, Sensitive, Public
  - Retenção mínima: 5 anos (regulatório)
```

### Flexível (Decisão do Time)

```yaml
Decisões do Domínio:
  - Schema específico dos eventos
  - SLAs de disponibilidade
  - Campos opcionais vs obrigatórios
  - Frequência de atualização
  - Métodos de acesso oferecidos
  - Transformações e agregações
  - Modelos de qualidade específicos
```

## Data Contract - Lifecycle Events

```yaml
# =============================================================
# DATA CONTRACT - LIFECYCLE EVENTS
# =============================================================
# Domínio: Gestão de Apólices
# Produto: Policy Lifecycle Events
# Versão: 1.0.0
# Data: 2025-01-15
# Owner: policy-domain-team@seguradora.com.br
# =============================================================

contract:
  id: "policy-lifecycle-events-v1"
  version: "1.0.0"
  status: "production"
  type: "event-stream"

# -------------------------------------------------------------
# DESCRIÇÃO DO PRODUTO
# -------------------------------------------------------------
description:
  summary: "Stream de eventos em tempo real do ciclo de vida das apólices"
  purpose: |
    Disponibiliza todos os eventos significativos que ocorrem durante 
    o ciclo de vida de uma apólice, desde emissão até cancelamento,
    permitindo automações e visibilidade em tempo real.
  
business_context:
  domain: "Gestão de Apólices"
  capabilities:
    - "Notificação em tempo real de mudanças em apólices"
    - "Histórico completo de eventos por apólice"
    - "Triggers para automação de processos"
    - "Auditoria e compliance"

# -------------------------------------------------------------
# INTERFACE DE ACESSO
# -------------------------------------------------------------
interfaces:
  streaming:
    protocol: "Azure Event Hubs"
    endpoint: "policydomain-prod.servicebus.windows.net"
    hub_name: "policy-lifecycle-events"
    consumer_groups:
      - name: "default"
        description: "Consumo geral"
      - name: "financial-domain"
        description: "Exclusivo para domínio financeiro"
      - name: "analytics"
        description: "Para processamento analítico"
    
  rest_api:
    protocol: "HTTPS REST"
    base_url: "https://api.seguradora.com.br/policy-domain/v1"
    endpoints:
      - path: "/events"
        method: "GET"
        description: "Consulta eventos históricos"
      - path: "/events/{eventId}"
        method: "GET"
        description: "Detalhes de um evento específico"
      - path: "/events/subscribe"
        method: "POST"
        description: "Webhooks para eventos específicos"
    
  storage:
    protocol: "Azure Data Lake Gen2"
    path: "abfss://datalake@policydomain.dfs.core.windows.net/events/"
    format: "Parquet"
    partitioning: "year/month/day/hour"

# -------------------------------------------------------------
# SCHEMA DOS EVENTOS
# -------------------------------------------------------------
schema:
  format: "JSON"
  registry: "Azure Schema Registry"
  registry_url: "policydomain-prod.servicebus.windows.net"
  
  base_event:
    eventId:
      type: "string"
      format: "uuid"
      required: true
      description: "Identificador único do evento"
      
    eventType:
      type: "string"
      required: true
      enum:
        - "POLICY_QUOTED"
        - "POLICY_ISSUED"
        - "POLICY_ACTIVATED"
        - "POLICY_ENDORSED"
        - "POLICY_RENEWED"
        - "POLICY_SUSPENDED"
        - "POLICY_REINSTATED"
        - "POLICY_CANCELLED"
        - "POLICY_EXPIRED"
        - "PAYMENT_RECEIVED"
        - "PAYMENT_OVERDUE"
        
    eventTimestamp:
      type: "string"
      format: "datetime"
      required: true
      timezone: "UTC"
      
    policyNumber:
      type: "string"
      required: true
      pattern: "^[A-Z]{2,4}-\\d{4}-\\d{6}$"
      
    policyId:
      type: "integer"
      required: true
      
    customerId:
      type: "integer"
      required: true
      classification: "PII"
      
    productId:
      type: "string"
      required: true
      
    actor:
      type: "object"
      required: true
      properties:
        type:
          type: "string"
          enum: ["SYSTEM", "USER", "API", "BATCH"]
        id:
          type: "string"
        name:
          type: "string"
          classification: "PII"
          
    metadata:
      type: "object"
      required: false
      properties:
        correlationId:
          type: "string"
        sourceSystem:
          type: "string"
        version:
          type: "string"

  event_specific_data:
    POLICY_ISSUED:
      effectiveDate: "date"
      expirationDate: "date"
      totalPremium: "decimal"
      paymentPlan: "string"
      
    POLICY_CANCELLED:
      cancellationDate: "date"
      cancellationReason: "string"
      refundAmount: "decimal"
      initiatedBy: "string"
      
    PAYMENT_OVERDUE:
      daysOverdue: "integer"
      overdueAmount: "decimal"
      suspensionDate: "date"

# -------------------------------------------------------------
# QUALIDADE DOS DADOS
# -------------------------------------------------------------
data_quality:
  completeness:
    - field: "eventId"
      threshold: 100%
    - field: "eventType"
      threshold: 100%
    - field: "policyNumber"
      threshold: 100%
      
  timeliness:
    - metric: "event_lag"
      description: "Tempo entre ocorrência e disponibilização"
      threshold: "< 5 segundos"
      measurement: "p95"
      
  accuracy:
    - validation: "schema_compliance"
      threshold: 100%
    - validation: "business_rules"
      threshold: 99.9%
      
  uniqueness:
    - field: "eventId"
      guarantee: "globally unique"

# -------------------------------------------------------------
# SLA - ACORDO DE NÍVEL DE SERVIÇO
# -------------------------------------------------------------
sla:
  availability:
    target: 99.9%
    measurement_period: "monthly"
    exclusions:
      - "Manutenção planejada (máx 4h/mês)"
      - "Força maior"
      
  latency:
    p50: 100ms
    p95: 500ms
    p99: 1000ms
    
  throughput:
    sustained: "1000 eventos/segundo"
    peak: "10000 eventos/segundo"
    
  data_freshness:
    real_time_events: "< 5 segundos"
    historical_api: "< 1 minuto"
    data_lake: "< 1 hora"

# -------------------------------------------------------------
# SEGURANÇA E GOVERNANÇA
# -------------------------------------------------------------
security:
  authentication:
    method: "Azure AD Service Principal"
    token_endpoint: "https://login.microsoftonline.com/{tenant}/oauth2/token"
    
  authorization:
    model: "RBAC"
    roles:
      - name: "lifecycle-events-reader"
        permissions: ["read"]
      - name: "lifecycle-events-subscriber"
        permissions: ["read", "subscribe"]
        
  encryption:
    at_rest: "AES-256"
    in_transit: "TLS 1.2+"
    key_management: "Azure Key Vault"
    
  data_classification:
    - field_pattern: "customer*"
      classification: "PII"
    - field_pattern: "policy*"
      classification: "Sensitive"
    - field_pattern: "event*"
      classification: "Internal"

privacy:
  pii_handling:
    - "Campos PII são mascarados em logs"
    - "Acesso a PII requer justificativa"
    - "PII pode ser anonimizado sob demanda"
    
  retention:
    event_stream: "7 dias"
    api_history: "90 dias"
    data_lake: "7 anos"
    
  compliance:
    - "LGPD compliant"
    - "SUSEP normativas"
    - "SOC 2 Type II"

# -------------------------------------------------------------
# VERSIONAMENTO E MUDANÇAS
# -------------------------------------------------------------
versioning:
  strategy: "Semantic Versioning"
  backward_compatibility: "Garantida por 6 meses"
  deprecation_notice: "3 meses de antecedência"
  
  breaking_changes_policy: |
    - Novos campos: Sempre opcionais
    - Remoção de campos: Deprecação gradual
    - Mudança de tipos: Nova versão major
    - Notificação: Email + Portal de APIs

# -------------------------------------------------------------
# SUPORTE E DOCUMENTAÇÃO
# -------------------------------------------------------------
support:
  documentation: "https://developer.seguradora.com.br/lifecycle-events"
  portal: "https://apiportal.seguradora.com.br"
  
  channels:
    email: "policy-domain-support@seguradora.com.br"
    slack: "#policy-domain-support"
    teams: "Policy Domain Team"
    
  sla_support:
    critical: "2 horas"
    high: "8 horas"
    normal: "2 dias úteis"

monitoring:
  dashboards:
    - name: "Event Stream Health"
      url: "https://monitoring.seguradora.com.br/lifecycle-events"
    - name: "API Metrics"
      url: "https://apim.azure.com/lifecycle-events/metrics"
      
  alerts:
    - metric: "error_rate"
      threshold: "> 1%"
      action: "PagerDuty"
    - metric: "latency_p99"
      threshold: "> 2s"
      action: "Email team"

# -------------------------------------------------------------
# CUSTOS E BILLING
# -------------------------------------------------------------
costs:
  model: "Compartilhado entre consumidores"
  
  breakdown:
    event_hubs: "R$ 5.000/mês"
    storage: "R$ 2.000/mês"
    api_management: "R$ 3.000/mês"
    total: "R$ 10.000/mês"
    
  allocation:
    method: "Por volume consumido"
    billing_cycle: "Mensal"
    reports: "Via Power BI corporativo"

# -------------------------------------------------------------
# EXEMPLOS DE USO
# -------------------------------------------------------------
examples:
  streaming_consumer:
    language: "Python"
    code: |
      from azure.eventhub import EventHubConsumerClient
      
      client = EventHubConsumerClient.from_connection_string(
          conn_str=CONNECTION_STRING,
          consumer_group="$Default",
          eventhub_name="policy-lifecycle-events"
      )
      
      def on_event(partition_context, events):
          for event in events:
              data = json.loads(event.body_as_str())
              if data['eventType'] == 'POLICY_RENEWED':
                  # Processar renovação
                  process_renewal(data)
                  
      with client:
          client.receive(on_event=on_event)
          
  api_query:
    curl: |
      curl -X GET \
        'https://api.seguradora.com.br/policy-domain/v1/events?policyNumber=AUTO-2025-000123' \
        -H 'Authorization: Bearer {token}' \
        -H 'Accept: application/json'

# -------------------------------------------------------------
# ROADMAP
# -------------------------------------------------------------
roadmap:
  2025_Q1:
    - "Eventos básicos de apólice"
    - "API de consulta"
    
  2025_Q2:
    - "Webhooks personalizados"
    - "Agregações em tempo real"
    
  2025_Q3:
    - "Machine Learning para anomalias"
    - "GraphQL endpoint"
```

## Como o Data Contract é Usado

### 1. **Durante o Desenvolvimento**

```yaml
Produtores usam para:
  - Implementar exatamente o schema prometido
  - Configurar os SLAs nos monitores
  - Criar testes que validem o contrato

Consumidores usam para:
  - Entender o que está disponível
  - Planejar suas integrações
  - Saber os limites e garantias
```

### 2. **Em Produção**

```yaml
Monitoramento Automatizado:
  - Azure Monitor valida SLAs
  - Schema Registry valida eventos
  - API Management enforça rate limits

Governança:
  - Auditorias verificam compliance
  - Métricas de qualidade são publicadas
  - Custos são alocados conforme uso
```

### 3. **Para Evolução**

```yaml
Gestão de Mudanças:
  - Pull requests validam breaking changes
  - Consumidores são notificados
  - Versões antigas mantidas por 6 meses
```
