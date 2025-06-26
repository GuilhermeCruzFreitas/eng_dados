# Exemplos de estruturação de times dos domínios
### DOMÍNIO: Clientes

```yaml
Product Owner: 
  - Gerente de CRM/Customer Experience
  - Reporta para: VP de Marketing ou CDO

Time Core (6-8 pessoas):
  - 2 Engenheiros de Dados Senior
  - 1 Arquiteto de Soluções
  - 1 Analista de Negócios (CRM)
  - 1 Especialista em Qualidade de Dados
  - 1 DevOps/SRE
  - 1 Analista de Compliance (LGPD)
  
Times Contribuidores (part-time):
  - Legal (privacidade)
  - Segurança (InfoSec)
  - Marketing (requisitos)
  - Atendimento (feedback)

Tecnologias sob gestão:
  - MDM Platform (Informatica/Talend)
  - API Gateway
  - Cache distribuído
  - Event Hub
  - Databricks Workspace #1 (Analytics)
  - Databricks Workspace #2 (Real-time)
```

### DOMÍNIO: Vendas e Distribuição


```yaml
Product Owner:
  - Diretor de Vendas Digitais
  - Reporta para: VP Comercial

Time Core (8-10 pessoas):
  - 2 Engenheiros de Dados
  - 2 Desenvolvedores Full-Stack
  - 1 Cientista de Dados (pricing)
  - 1 Analista de Negócios (vendas)
  - 1 Especialista em Integrações
  - 1 DevOps
  - 1 Scrum Master

Times Contribuidores:
  - Canais de Vendas
  - Corretores
  - Produto (catálogo)
  - Atuarial (pricing)

Tecnologias:
  - CRM (Salesforce)
  - Portal de Cotações
  - APIs de Parceiros
  - Databricks (Analytics de Vendas)
  - Motor de Regras
```

### DOMÍNIO: Sinistros


```yaml
Product Owner:
  - Gerente de Operações de Sinistros
  - Reporta para: VP de Operações

Time Core (10-12 pessoas):
  - 3 Engenheiros de Dados
  - 2 Desenvolvedores Backend
  - 1 Cientista de Dados (fraude)
  - 2 Analistas de Negócios
  - 1 Arquiteto de Integração
  - 1 DevOps
  - 1 Especialista em Processos
  - 1 QA Engineer

Times Contribuidores:
  - Regulação
  - Jurídico
  - Rede Credenciada
  - Antifraude

Tecnologias:
  - Sistema Core de Sinistros
  - Workflow Engine
  - OCR/AI para documentos
  - Databricks ML
  - Databricks
  - APIs de Terceiros
```

# Estrutura de Domínios e Entidades

| **DOMÍNIO**               | **ENTIDADE PRINCIPAL**        | **ENTIDADES RELACIONADAS**                                                                         | **OWNERSHIP**         | **FREQUÊNCIA ATUALIZAÇÃO** | **OVERLAP COM OUTROS DOMÍNIOS**                                                   |
| ------------------------- | ----------------------------- | -------------------------------------------------------------------------------------------------- | --------------------- | -------------------------- | --------------------------------------------------------------------------------- |
| **GESTÃO DE CLIENTES**    | **Cliente**                   | • Endereço<br>• Contato<br>• Documento<br>• Beneficiário<br>• Preferências                         | Marketing / CRM       | Baixa                      | • Vendas (dados cadastrais)<br>• Sinistros (validação)<br>• Financeiro (cobrança) |
| **GESTÃO DE PRODUTOS**    | **Produto**                   | • Cobertura Base<br>• Regras de Aceitação<br>• Fatores de Rating<br>• Requisitos<br>• Documentação | Produto / Atuarial    | Muito Baixa                | • Vendas (catálogo)<br>• Subscrição (regras)<br>• Sinistros (coberturas)          |
| **VENDAS E DISTRIBUIÇÃO** | **Cotação**<br>**Proposta**   | • Lead<br>• Oportunidade<br>• Canal de Venda<br>• Corretor/Agente<br>• Campanha                    | Comercial / Digital   | Alta                       | • Clientes (cadastro)<br>• Produtos (preços)<br>• Subscrição (análise)            |
| **SUBSCRIÇÃO**            | **Análise de Risco**          | • Parecer Técnico<br>• Score de Risco<br>• Exceções<br>• Aprovações<br>• Restrições                | Underwriting          | Média                      | • Vendas (propostas)<br>• Sinistros (histórico)<br>• Externo (bureaus)            |
| **GESTÃO DE APÓLICES**    | **Apólice**                   | • Risco/Bem Segurado<br>• Cobertura Contratada<br>• Endosso<br>• Renovação<br>• Cancelamento       | Emissão / Operações   | Média                      | • Cliente (titular)<br>• Financeiro (prêmios)<br>• Sinistros (vigência)           |
| **GESTÃO DE SINISTROS**   | **Sinistro**                  | • Aviso<br>• Regulação<br>• Reserva<br>• Pagamento Sinistro<br>• Salvado<br>• Sub-rogação          | Sinistros / Regulação | Alta                       | • Apólice (cobertura)<br>• Financeiro (pagamentos)<br>• Jurídico (disputas)       |
| **FINANCEIRO**            | **Pagamento**<br>**Comissão** | • Prêmio<br>• Parcela<br>• Cobrança<br>• Inadimplência<br>• Reconciliação                          | Financeiro / Cobrança | Alta                       | • Apólice (vigência)<br>• Vendas (comissões)<br>• Sinistros (indenizações)        |

## Hierarquia de Dependências

```
┌─────────────────────────────────────────────────────────────┐
│                      NÍVEL 0 - FUNDAÇÃO                     │
├─────────────────────────────────────────────────────────────┤
│  • CLIENTE (independente)                                   │
│  • PRODUTO (independente)                                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    NÍVEL 1 - TRANSACIONAL                   │
├─────────────────────────────────────────────────────────────┤
│  • COTAÇÃO (depende de: Cliente, Produto)                  │
│  • PROPOSTA (depende de: Cotação, Cliente)                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    NÍVEL 2 - CONTRATUAL                     │
├─────────────────────────────────────────────────────────────┤
│  • APÓLICE (depende de: Proposta, Cliente, Produto)        │
│  • RISCO/BEM (depende de: Apólice)                         │
│  • COBERTURA (depende de: Apólice, Produto)                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    NÍVEL 3 - OPERACIONAL                    │
├─────────────────────────────────────────────────────────────┤
│  • SINISTRO (depende de: Apólice, Cobertura)               │
│  • PAGAMENTO (depende de: Apólice)                         │
│  • COMISSÃO (depende de: Apólice, Pagamento)               │
│  • ENDOSSO (depende de: Apólice)                           │
└─────────────────────────────────────────────────────────────┘
```

## Matriz de Conflitos Entre Domínios

| **DOMÍNIO**    | **CLIENTES** | **PRODUTOS**  | **VENDAS**   | **SUBSCRIÇÃO** | **APÓLICES** | **SINISTROS** | **FINANCEIRO** |
| -------------- | ------------ | ------------- | ------------ | -------------- | ------------ | ------------- | -------------- |
| **Clientes**   | -            | -             | 🟡 Cadastro  | -              | 🟡 Titular   | 🟡 Validação  | 🟡 Cobrança    |
| **Produtos**   | -            | -             | 🟡 Catálogo  | 🔴 Regras      | 🟡 Vigência  | 🔴 Coberturas | -              |
| **Vendas**     | 🟡 Cadastro  | 🟡 Catálogo   | -            | 🔴 Propostas   | 🟡 Conversão | -             | 🔴 Comissões   |
| **Subscrição** | -            | 🔴 Regras     | 🔴 Propostas | -              | 🔴 Aprovação | 🟡 Histórico  | -              |
| **Apólices**   | 🟡 Titular   | 🟡 Vigência   | 🟡 Conversão | 🔴 Aprovação   | -            | 🔴 Vigência   | 🔴 Prêmios     |
| **Sinistros**  | 🟡 Validação | 🔴 Coberturas | -            | 🟡 Histórico   | 🔴 Vigência  | -             | 🔴 Pagamentos  |
| **Financeiro** | 🟡 Cobrança  | -             | 🔴 Comissões | -              | 🔴 Prêmios   | 🔴 Pagamentos | -              |

**Legenda:**

- 🔴 **Conflito Alto**: Mesmos dados com regras diferentes ou ownership disputado
- 🟡 **Conflito Médio**: Necessidade de sincronização ou consulta frequente
- **-** Sem conflito significativo

## Indicadores para Data Mesh

### **Red Flags - Sinais de Necessidade de Domínios Separados**

1. **Múltiplos Owners**:
    - Dados de cliente atualizados por Marketing, Vendas e Sinistros
    - Regras de produto definidas por Produto mas modificadas por Subscrição
2. **Velocidades Diferentes**:
    - Produtos: Atualizações trimestrais/anuais
    - Cotações: Milhares por hora
    - Sinistros: Centenas por dia com SLA crítico
3. **Regras Conflitantes**:
    - Vendas quer flexibilizar dados para converter
    - Subscrição quer rigor para reduzir risco
    - Sinistros quer completude para evitar fraude
4. **Escalas Incompatíveis**:
    - 1 produto → 100.000 cotações → 10.000 apólices → 1.000 sinistros
    - Volume e performance necessária completamente diferentes

### **Métricas de Autonomia por Domínio**

| **Domínio** | **% Dados Próprios** | **% Dados Externos** | **Nível Autonomia** |
| ----------- | -------------------- | -------------------- | ------------------- |
| Clientes    | 90%                  | 10%                  | Alto ⬆️             |
| Produtos    | 95%                  | 5%                   | Muito Alto ⬆️⬆️     |
| Vendas      | 60%                  | 40%                  | Médio ➡️            |
| Subscrição  | 40%                  | 60%                  | Baixo ⬇️            |
| Apólices    | 70%                  | 30%                  | Alto ⬆️             |
| Sinistros   | 65%                  | 35%                  | Médio-Alto ↗️       |
| Financeiro  | 45%                  | 55%                  | Baixo ⬇️            |
### **Priorização para Implementação Data Mesh**

1. **Wave 1 - Domínios Fundacionais**
    - Clientes (alta autonomia, base para outros)
    - Produtos (muito alta autonomia, estável)
2. **Wave 2 - Domínios Transacionais**
    - Vendas (alto volume, regras próprias)
    - Apólices (central ao negócio)
3. **Wave 3 - Domínios Operacionais**
    - Sinistros (crítico, SLA sensível)
    - Financeiro (dependente mas crítico)
4. **Wave 4 - Domínio Analítico**
    - Subscrição (altamente dependente, beneficia de dados refinados dos outros)