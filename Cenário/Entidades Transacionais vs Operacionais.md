## Entidades Transacionais vs Operacionais

### **ENTIDADES TRANSACIONAIS (Process Entities)**

São entidades que **representam o processo de venda/contratação** - a jornada comercial até a formalização do contrato:

- **Cotação**: Simulação, exploração de possibilidades
- **Proposta**: Intenção formal de contratar
- **Apólice**: Contrato efetivado

**Características:**

- Focadas na **jornada de aquisição**
- Representam **etapas de um funil**
- Têm **estados bem definidos** (aberta → convertida → expirada)
- São **imutáveis após conclusão** (uma cotação convertida não muda)
- Ownership geralmente em **Vendas/Comercial**

### **ENTIDADES OPERACIONAIS (Operational Entities)**

São entidades que representam **operações do dia-a-dia após a contratação** - o que acontece durante a vigência do seguro:

- **Sinistro**: Uso efetivo do seguro
- **Pagamento/Prêmio**: Manutenção financeira do contrato
- **Endosso**: Alterações no contrato vigente

**Características:**

- Focadas na **operação do negócio**
- Representam **eventos durante a vigência**
- Têm **ciclos de vida complexos** (sinistro: aberto → em análise → pago)
- São **mutáveis** (status do sinistro muda constantemente)
- Ownership distribuído em **múltiplas áreas operacionais**

## Por que essa distinção importa?

### 1. **Velocidade e Volume**

```
Transacionais:         Operacionais:
Cotação: 100.000/dia   Sinistro: 1.000/dia
Proposta: 10.000/dia   Pagamento: 50.000/dia
Apólice: 5.000/dia     Endosso: 500/dia
```

### 2. **Padrões de Acesso**

- **Transacionais**: Write-once, read-many (histórico)
- **Operacionais**: Read-write intensivo (atualizações constantes)

### 3. **Requisitos de Sistema**

- **Transacionais**: Otimizados para conversão e analytics
- **Operacionais**: Otimizados para SLA e processamento

### 4. **Governança de Dados**

- **Transacionais**: Podem ser arquivadas após conversão/expiração
- **Operacionais**: Precisam estar sempre acessíveis durante vigência

## Exemplo Prático

**Cenário**: João quer um seguro de carro

**Fase Transacional** (pré-contrato):

1. João faz uma **cotação** online
2. Gosta do preço e submete uma **proposta**
3. Após aprovação, é emitida a **apólice**

**Fase Operacional** (pós-contrato):

1. João paga o **prêmio** mensalmente
2. João bate o carro e abre um **sinistro**
3. João muda de endereço e faz um **endosso**

## Overlap e Complexidade

Na prática, há algum overlap:

- **Apólice** é transicional - marca o fim do processo transacional e início do operacional
- **Renovação** pode ser vista como operacional (manutenção) ou transacional (nova venda)
- **Pagamento** da primeira parcela pode ser transacional, mas parcelas seguintes são operacionais