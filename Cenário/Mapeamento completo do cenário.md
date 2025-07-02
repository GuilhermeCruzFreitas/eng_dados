# Mapa Completo - Hierarquia de Domínios, Entidades e Tabelas


## DOMÍNIO: GESTÃO DE CLIENTES (Master Domain)

### Entidades e Tabelas

```sql
-- ENTIDADE: Cliente (Core)
CREATE TABLE customer_master (
    customer_id         BIGINT PRIMARY KEY,
    customer_type       VARCHAR(20),  -- PF/PJ/ESTRANGEIRO
    tax_id             VARCHAR(20) UNIQUE,  -- CPF/CNPJ
    full_name          VARCHAR(200),
    birth_date         DATE,
    gender             VARCHAR(10),
    marital_status     VARCHAR(20),
    occupation         VARCHAR(100),
    monthly_income     DECIMAL(15,2),
    credit_score       INT,
    customer_segment   VARCHAR(50),
    acquisition_date   TIMESTAMP,
    acquisition_channel VARCHAR(50),
    status             VARCHAR(20),
    created_at         TIMESTAMP,
    updated_at         TIMESTAMP
);

-- ENTIDADE: Endereço
CREATE TABLE customer_address (
    address_id         BIGINT PRIMARY KEY,
    customer_id        BIGINT REFERENCES customer_master,
    address_type       VARCHAR(20),  -- RESIDENTIAL/COMMERCIAL/BILLING
    street             VARCHAR(200),
    number             VARCHAR(20),
    complement         VARCHAR(100),
    neighborhood       VARCHAR(100),
    city               VARCHAR(100),
    state              VARCHAR(2),
    zip_code           VARCHAR(10),
    country            VARCHAR(50),
    latitude           DECIMAL(10,8),
    longitude          DECIMAL(11,8),
    is_primary         BOOLEAN,
    created_at         TIMESTAMP
);

-- ENTIDADE: Contato
CREATE TABLE customer_contact (
    contact_id         BIGINT PRIMARY KEY,
    customer_id        BIGINT REFERENCES customer_master,
    contact_type       VARCHAR(20),  -- PHONE/EMAIL/WHATSAPP
    contact_value      VARCHAR(100),
    is_primary         BOOLEAN,
    is_verified        BOOLEAN,
    opt_in_marketing   BOOLEAN,
    opt_in_whatsapp    BOOLEAN,
    preferred_time     VARCHAR(20),
    created_at         TIMESTAMP
);

-- ENTIDADE: Consentimento LGPD
CREATE TABLE customer_consent (
    consent_id         BIGINT PRIMARY KEY,
    customer_id        BIGINT REFERENCES customer_master,
    consent_type       VARCHAR(50),
    consent_status     BOOLEAN,
    consent_date       TIMESTAMP,
    expiration_date    DATE,
    collection_method  VARCHAR(50),
    ip_address         VARCHAR(45)
);

-- ENTIDADE: Relacionamentos
CREATE TABLE customer_relationship (
    relationship_id    BIGINT PRIMARY KEY,
    customer_id        BIGINT REFERENCES customer_master,
    related_customer_id BIGINT REFERENCES customer_master,
    relationship_type  VARCHAR(50),  -- SPOUSE/CHILD/PARENT/COMPANY
    start_date         DATE,
    end_date           DATE
);
```

---

## DOMÍNIO: GESTÃO DE PRODUTOS (Master Domain)

### Entidades e Tabelas

```sql
-- ENTIDADE: Produto
CREATE TABLE product_master (
    product_id         VARCHAR(50) PRIMARY KEY,
    product_code       VARCHAR(30) UNIQUE,
    product_name       VARCHAR(200),
    product_line       VARCHAR(50),  -- AUTO/HOME/LIFE/HEALTH
    product_type       VARCHAR(50),  -- INDIVIDUAL/BUSINESS
    category           VARCHAR(50),  -- COMPREHENSIVE/BASIC/PREMIUM
    launch_date        DATE,
    discontinue_date   DATE,
    status             VARCHAR(20),
    min_age            INT,
    max_age            INT,
    eligibility_rules  JSONB,
    created_at         TIMESTAMP,
    updated_at         TIMESTAMP
);

-- ENTIDADE: Cobertura (Master)
CREATE TABLE coverage_master (
    coverage_id        VARCHAR(50) PRIMARY KEY,
    coverage_code      VARCHAR(30) UNIQUE,
    coverage_name      VARCHAR(200),
    coverage_type      VARCHAR(30),  -- BASIC/ADDITIONAL/OPTIONAL
    description        TEXT,
    default_limit      DECIMAL(15,2),
    default_deductible DECIMAL(15,2),
    inclusions         JSONB,
    exclusions         JSONB,
    version            VARCHAR(10),
    effective_date     DATE,
    expiration_date    DATE
);

-- ENTIDADE: Produto-Cobertura (Associação)
CREATE TABLE product_coverage (
    product_id         VARCHAR(50) REFERENCES product_master,
    coverage_id        VARCHAR(50) REFERENCES coverage_master,
    is_mandatory       BOOLEAN,
    min_limit          DECIMAL(15,2),
    max_limit          DECIMAL(15,2),
    PRIMARY KEY (product_id, coverage_id)
);

-- ENTIDADE: Fatores de Rating
CREATE TABLE rating_factor (
    factor_id          BIGINT PRIMARY KEY,
    product_id         VARCHAR(50) REFERENCES product_master,
    factor_name        VARCHAR(100),
    factor_type        VARCHAR(50),
    factor_table       JSONB,  -- Tabela de fatores
    effective_date     DATE,
    expiration_date    DATE
);

-- ENTIDADE: Documentos Requeridos
CREATE TABLE product_document (
    document_id        BIGINT PRIMARY KEY,
    product_id         VARCHAR(50) REFERENCES product_master,
    document_type      VARCHAR(100),
    is_mandatory       BOOLEAN,
    validation_rules   JSONB
);
```


---

## DOMÍNIO: VENDAS E DISTRIBUIÇÃO

### Entidades e Tabelas

```sql
-- ENTIDADE: Lead
CREATE TABLE sales_lead (
    lead_id            BIGINT PRIMARY KEY,
    source_channel     VARCHAR(50),
    campaign_id        VARCHAR(50),
    contact_info       JSONB,
    interest_products  JSONB,
    lead_score         DECIMAL(5,2),
    status             VARCHAR(30),
    assigned_to        VARCHAR(100),
    created_at         TIMESTAMP,
    converted_at       TIMESTAMP
);

-- ENTIDADE: Cotação
CREATE TABLE quote (
    quote_id           BIGINT PRIMARY KEY,
    quote_number       VARCHAR(30) UNIQUE,
    customer_id        BIGINT,  -- Ref: customer_master
    lead_id            BIGINT REFERENCES sales_lead,
    product_id         VARCHAR(50),  -- Ref: product_master
    quote_date         TIMESTAMP,
    expiration_date    DATE,
    channel            VARCHAR(50),
    broker_id          BIGINT,  -- Ref: partner_master
    status             VARCHAR(30),
    created_at         TIMESTAMP
);

-- ENTIDADE: Item Cotado
CREATE TABLE quote_item (
    item_id            BIGINT PRIMARY KEY,
    quote_id           BIGINT REFERENCES quote,
    risk_data          JSONB,  -- Dados específicos do risco
    selected_coverages JSONB,  -- Coberturas selecionadas
    base_premium       DECIMAL(15,2),
    discounts          JSONB,
    taxes              DECIMAL(15,2),
    total_premium      DECIMAL(15,2),
    risk_score         DECIMAL(5,2)
);

-- ENTIDADE: Proposta
CREATE TABLE proposal (
    proposal_id        BIGINT PRIMARY KEY,
    proposal_number    VARCHAR(30) UNIQUE,
    quote_id           BIGINT REFERENCES quote,
    customer_id        BIGINT,  -- Ref: customer_master
    submission_date    TIMESTAMP,
    expiration_date    DATE,
    payment_method     VARCHAR(30),
    payment_frequency  VARCHAR(20),
    status             VARCHAR(30),
    underwriter_id     VARCHAR(100),
    decision_date      TIMESTAMP,
    decision_notes     TEXT
);

-- ENTIDADE: Documentos da Proposta
CREATE TABLE proposal_document (
    document_id        BIGINT PRIMARY KEY,
    proposal_id        BIGINT REFERENCES proposal,
    document_type      VARCHAR(100),
    file_path          VARCHAR(500),
    upload_date        TIMESTAMP,
    validation_status  VARCHAR(30),
    validation_notes   TEXT
);
```


---

## DOMÍNIO: SUBSCRIÇÃO (Cross-Domain)

### Entidades e Tabelas

```sql
-- ENTIDADE: Análise de Risco
CREATE TABLE risk_analysis (
    analysis_id        BIGINT PRIMARY KEY,
    proposal_id        BIGINT,  -- Ref: proposal
    analyst_id         VARCHAR(100),
    analysis_date      TIMESTAMP,
    risk_score         DECIMAL(5,2),
    risk_factors       JSONB,
    external_data      JSONB,  -- Bureau, histórico
    decision           VARCHAR(30),  -- APPROVE/DECLINE/REFER
    conditions         JSONB,
    pricing_adjustment DECIMAL(5,2)
);

-- ENTIDADE: Regras de Subscrição
CREATE TABLE underwriting_rule (
    rule_id            BIGINT PRIMARY KEY,
    product_id         VARCHAR(50),  -- Ref: product_master
    rule_name          VARCHAR(200),
    rule_type          VARCHAR(50),
    rule_expression    TEXT,
    action             VARCHAR(50),
    priority           INT,
    effective_date     DATE,
    expiration_date    DATE
);

-- ENTIDADE: Exceções
CREATE TABLE underwriting_exception (
    exception_id       BIGINT PRIMARY KEY,
    analysis_id        BIGINT REFERENCES risk_analysis,
    rule_id            BIGINT REFERENCES underwriting_rule,
    exception_reason   TEXT,
    approved_by        VARCHAR(100),
    approval_date      TIMESTAMP,
    expiration_date    DATE
);
```

---

## DOMÍNIO: GESTÃO DE APÓLICES

### Entidades e Tabelas

```sql
-- ENTIDADE: Apólice
CREATE TABLE policy (
    policy_id          BIGINT PRIMARY KEY,
    policy_number      VARCHAR(30) UNIQUE,
    proposal_id        BIGINT,  -- Ref: proposal
    customer_id        BIGINT,  -- Ref: customer_master
    product_id         VARCHAR(50),  -- Ref: product_master
    issue_date         DATE,
    effective_date     DATE,
    expiration_date    DATE,
    renewal_date       DATE,
    total_premium      DECIMAL(15,2),
    payment_frequency  VARCHAR(20),
    status             VARCHAR(30),
    cancellation_date  DATE,
    cancellation_reason VARCHAR(200),
    broker_id          BIGINT,  -- Ref: partner_master
    commission_rate    DECIMAL(5,2)
);

-- ENTIDADE: Risco Segurado
CREATE TABLE insured_risk (
    risk_id            BIGINT PRIMARY KEY,
    policy_id          BIGINT REFERENCES policy,
    risk_type          VARCHAR(50),  -- VEHICLE/PROPERTY/LIFE
    description        TEXT,
    insured_value      DECIMAL(15,2),
    location           JSONB,
    specific_data      JSONB  -- Dados específicos por tipo
);

-- ENTIDADE: Risco Veículo (Específico)
CREATE TABLE risk_vehicle (
    risk_id            BIGINT PRIMARY KEY REFERENCES insured_risk,
    plate              VARCHAR(20),
    chassis            VARCHAR(50),
    make               VARCHAR(50),
    model              VARCHAR(100),
    year_manufacture   INT,
    year_model         INT,
    fuel_type          VARCHAR(20),
    usage_type         VARCHAR(30),
    overnight_zipcode  VARCHAR(10),
    has_tracker        BOOLEAN,
    garage_type        VARCHAR(30)
);

-- ENTIDADE: Coberturas Contratadas
CREATE TABLE policy_coverage (
    policy_coverage_id BIGINT PRIMARY KEY,
    policy_id          BIGINT REFERENCES policy,
    coverage_id        VARCHAR(50),  -- Ref: coverage_master
    coverage_limit     DECIMAL(15,2),
    deductible         DECIMAL(15,2),
    premium            DECIMAL(15,2),
    waiting_period     INT,
    special_conditions TEXT
);

-- ENTIDADE: Endosso
CREATE TABLE endorsement (
    endorsement_id     BIGINT PRIMARY KEY,
    policy_id          BIGINT REFERENCES policy,
    endorsement_number VARCHAR(30),
    endorsement_type   VARCHAR(50),
    request_date       TIMESTAMP,
    effective_date     DATE,
    description        TEXT,
    premium_adjustment DECIMAL(15,2),
    status             VARCHAR(30),
    approved_by        VARCHAR(100)
);

-- ENTIDADE: Beneficiários
CREATE TABLE policy_beneficiary (
    beneficiary_id     BIGINT PRIMARY KEY,
    policy_id          BIGINT REFERENCES policy,
    customer_id        BIGINT,  -- Ref: customer_master
    beneficiary_type   VARCHAR(30),
    percentage         DECIMAL(5,2),
    is_primary         BOOLEAN
);
```


---

## DOMÍNIO: GESTÃO DE SINISTROS

### Entidades e Tabelas

```sql
-- ENTIDADE: Sinistro
CREATE TABLE claim (
    claim_id           BIGINT PRIMARY KEY,
    claim_number       VARCHAR(30) UNIQUE,
    policy_id          BIGINT,  -- Ref: policy
    loss_date          TIMESTAMP,
    notification_date  TIMESTAMP,
    claim_type         VARCHAR(50),
    loss_type          VARCHAR(100),
    description        TEXT,
    location           JSONB,
    police_report      VARCHAR(50),
    estimated_amount   DECIMAL(15,2),
    status             VARCHAR(30),
    adjuster_id        VARCHAR(100),
    fraud_score        DECIMAL(5,2),
    created_at         TIMESTAMP,
    closed_at          TIMESTAMP
);

-- ENTIDADE: Documentos do Sinistro
CREATE TABLE claim_document (
    document_id        BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    document_type      VARCHAR(100),
    document_source    VARCHAR(50),  -- CUSTOMER/POLICE/MEDICAL
    file_path          VARCHAR(500),
    upload_date        TIMESTAMP,
    verified_by        VARCHAR(100),
    verification_date  TIMESTAMP
);

-- ENTIDADE: Avaliação
CREATE TABLE claim_assessment (
    assessment_id      BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    assessor_id        VARCHAR(100),
    assessment_date    TIMESTAMP,
    damage_description TEXT,
    repair_estimate    DECIMAL(15,2),
    total_loss         BOOLEAN,
    photos             JSONB,
    report_path        VARCHAR(500)
);

-- ENTIDADE: Cobertura do Sinistro
CREATE TABLE claim_coverage (
    claim_coverage_id  BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    policy_coverage_id BIGINT,  -- Ref: policy_coverage
    claimed_amount     DECIMAL(15,2),
    approved_amount    DECIMAL(15,2),
    deductible_applied DECIMAL(15,2),
    coverage_status    VARCHAR(30),  -- COVERED/EXCLUDED/PARTIAL
    exclusion_reason   VARCHAR(200)
);

-- ENTIDADE: Pagamento de Sinistro
CREATE TABLE claim_payment (
    payment_id         BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    payment_type       VARCHAR(30),  -- INDEMNITY/REPAIR/MEDICAL
    payee_type         VARCHAR(30),  -- INSURED/PROVIDER/THIRD_PARTY
    payee_id           BIGINT,
    payment_date       DATE,
    amount             DECIMAL(15,2),
    payment_method     VARCHAR(30),
    bank_details       JSONB,
    status             VARCHAR(30)
);

-- ENTIDADE: Terceiros Envolvidos
CREATE TABLE claim_third_party (
    third_party_id     BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    party_type         VARCHAR(30),  -- VICTIM/WITNESS/OTHER_DRIVER
    name               VARCHAR(200),
    document_number    VARCHAR(20),
    contact_info       JSONB,
    insurance_info     JSONB,
    fault_percentage   DECIMAL(5,2)
);

-- ENTIDADE: Prestadores de Serviço
CREATE TABLE claim_service_provider (
    service_id         BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    provider_id        BIGINT,  -- Ref: partner_master
    service_type       VARCHAR(50),  -- TOWING/REPAIR/MEDICAL
    service_date       DATE,
    authorized_amount  DECIMAL(15,2),
    invoiced_amount    DECIMAL(15,2),
    status             VARCHAR(30)
);

-- ENTIDADE: Reserva
CREATE TABLE claim_reserve (
    reserve_id         BIGINT PRIMARY KEY,
    claim_id           BIGINT REFERENCES claim,
    reserve_type       VARCHAR(30),  -- INDEMNITY/EXPENSE/LEGAL
    amount             DECIMAL(15,2),
    currency           VARCHAR(3),
    set_date           DATE,
    set_by             VARCHAR(100),
    adjustment_reason  TEXT
);
```

---

## DOMÍNIO: FINANCEIRO

### Entidades e Tabelas

```sql
-- ENTIDADE: Faturamento
CREATE TABLE billing (
    billing_id         BIGINT PRIMARY KEY,
    policy_id          BIGINT,  -- Ref: policy
    billing_period     VARCHAR(20),
    due_date           DATE,
    premium_amount     DECIMAL(15,2),
    tax_amount         DECIMAL(15,2),
    fee_amount         DECIMAL(15,2),
    total_amount       DECIMAL(15,2),
    status             VARCHAR(30)
);

-- ENTIDADE: Pagamento de Prêmio
CREATE TABLE premium_payment (
    payment_id         BIGINT PRIMARY KEY,
    billing_id         BIGINT REFERENCES billing,
    payment_date       TIMESTAMP,
    amount_paid        DECIMAL(15,2),
    payment_method     VARCHAR(30),
    transaction_id     VARCHAR(100),
    bank_reference     VARCHAR(100),
    status             VARCHAR(30)
);

-- ENTIDADE: Comissão
CREATE TABLE commission (
    commission_id      BIGINT PRIMARY KEY,
    policy_id          BIGINT,  -- Ref: policy
    broker_id          BIGINT,  -- Ref: partner_master
    commission_type    VARCHAR(30),  -- NEW/RENEWAL/ENDORSEMENT
    base_amount        DECIMAL(15,2),
    commission_rate    DECIMAL(5,2),
    commission_amount  DECIMAL(15,2),
    earned_date        DATE,
    payment_date       DATE,
    status             VARCHAR(30)
);

-- ENTIDADE: Inadimplência
CREATE TABLE delinquency (
    delinquency_id     BIGINT PRIMARY KEY,
    policy_id          BIGINT,  -- Ref: policy
    billing_id         BIGINT REFERENCES billing,
    days_overdue       INT,
    overdue_amount     DECIMAL(15,2),
    collection_stage   VARCHAR(30),
    last_contact_date  DATE,
    next_action        VARCHAR(100),
    status             VARCHAR(30)
);

-- ENTIDADE: Reconciliação
CREATE TABLE reconciliation (
    reconciliation_id  BIGINT PRIMARY KEY,
    reconciliation_date DATE,
    account_type       VARCHAR(30),
    expected_amount    DECIMAL(15,2),
    actual_amount      DECIMAL(15,2),
    difference         DECIMAL(15,2),
    status             VARCHAR(30),
    notes              TEXT
);
```

---

## DOMÍNIO: PARCEIROS (Master Domain)

### Entidades e Tabelas

```sql
-- ENTIDADE: Parceiro
CREATE TABLE partner_master (
    partner_id         BIGINT PRIMARY KEY,
    partner_type       VARCHAR(50),  -- BROKER/PROVIDER/REINSURER
    tax_id             VARCHAR(20) UNIQUE,
    company_name       VARCHAR(200),
    trade_name         VARCHAR(200),
    registration_number VARCHAR(50),
    status             VARCHAR(30),
    rating             DECIMAL(3,1),
    created_at         TIMESTAMP
);

-- ENTIDADE: Corretor
CREATE TABLE broker (
    broker_id          BIGINT PRIMARY KEY REFERENCES partner_master,
    license_number     VARCHAR(50),
    license_expiry     DATE,
    commission_tier    VARCHAR(20),
    sales_target       DECIMAL(15,2),
    ytd_sales          DECIMAL(15,2)
);

-- ENTIDADE: Prestador de Serviços
CREATE TABLE service_provider (
    provider_id        BIGINT PRIMARY KEY REFERENCES partner_master,
    service_types      JSONB,  -- ["REPAIR", "TOWING", "MEDICAL"]
    coverage_areas     JSONB,  -- ZIP codes ou cidades
    capacity           INT,
    avg_response_time  INT,  -- minutos
    certifications     JSONB
);

-- ENTIDADE: Contrato de Parceiro
CREATE TABLE partner_contract (
    contract_id        BIGINT PRIMARY KEY,
    partner_id         BIGINT REFERENCES partner_master,
    contract_type      VARCHAR(50),
    start_date         DATE,
    end_date           DATE,
    terms              JSONB,
    sla_metrics        JSONB,
    status             VARCHAR(30)
);
```

---

## 🔗 Mapa de Integrações Entre Domínios

```yaml
FLUXOS PRINCIPAIS DE DADOS:

1. Jornada de Venda:
   Lead (Vendas) → Cliente (Clientes) → Cotação (Vendas) → 
   → Análise (Subscrição) → Proposta (Vendas) → Apólice (Apólices)

2. Jornada de Sinistro:
   Aviso (Sinistros) → Apólice (Apólices) → Cobertura (Produtos) →
   → Avaliação (Sinistros) → Pagamento (Financeiro)

3. Jornada Financeira:
   Apólice (Apólices) → Faturamento (Financeiro) → 
   → Pagamento (Financeiro) → Comissão (Financeiro)

4. Jornada de Renovação:
   Apólice (Apólices) → Sinistralidade (Sinistros) →
   → Novo Preço (Subscrição) → Renovação (Vendas)
```

## Produtos de Dados por Domínio (Resumo)

```yaml
CLIENTES:
- Customer 360 API
- Segmentation Dataset
- Consent Stream

PRODUTOS:
- Product Catalog API
- Coverage Rules Service
- Rating Factors API

VENDAS:
- Quote Stream
- Conversion Funnel
- Broker Performance

SUBSCRIÇÃO:
- Risk Score Service
- Decision Log
- Rules Engine

APÓLICES:
- Policy Registry
- Lifecycle Events
- Renewal Pipeline

SINISTROS:
- Claims Feed
- Fraud Signals
- Provider Network

FINANCEIRO:
- Payment Status
- Commission Calculator
- Delinquency Report

PARCEIROS:
- Partner Directory
- Performance Metrics
- Contract Management
```
```