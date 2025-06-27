---
tags:
  - objeto
entidade: operacional
mestre: true
objeto: tabela
---
**Definição**: Ocorrência de evento coberto pela apólice

**Atributos Principais:**

- `claim_id` (PK)
- `claim_number` - Número do sinistro
- `policy_id` (FK)
- `data_ocorrencia`
- `data_aviso`
- `data_abertura`
- `data_fechamento`
- `tipo_sinistro` - Colisão/Roubo/Incêndio/Morte
- `causa_sinistro`
- `descricao_ocorrencia`
- `local_ocorrencia` (JSON com lat/long)
- `boletim_ocorrencia`
- `valor_estimado`
- `valor_reserva`
- `valor_pago`
- `valor_franquia_cobrada`
- `status_sinistro` - Aberto/Em análise/Aprovado/Negado/Pago
- `motivo_negativa`
- `indicador_fraude`
- `score_fraude`
- `adjuster_id` (FK) - Regulador responsável

**Relacionamentos:**

- [[Apólices]] (N:1)
- [[Pagamentos de Sinistro]] (1:N)
- [[Documentos de Sinistro]] (1:N)
- [[Prestadores de Serviço]] (N:N)
- [[Terceiros Envolvidos]] (1:N)