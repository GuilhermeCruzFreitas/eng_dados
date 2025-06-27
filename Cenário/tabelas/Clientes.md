---
tags:
  - objeto
entidade: raiz
objeto: tabela
mestre: true
---


**Definição**: Qualquer indivíduo ou organização que tenha relacionamento com a seguradora

**Atributos Principais:**

- `customer_id` (PK) - Identificador único universal
- `customer_type` - PF/PJ/Corretor/Prestador
- `cpf_cnpj` - Documento fiscal
- `nome_completo`
- `data_nascimento` / `data_fundacao`
- `genero`
- `estado_civil`
- `profissao` / `ramo_atividade`
- `renda_mensal` / `faturamento_anual`
- `score_credito`
- `segmento_cliente` - Premium/Standard/Basic
- `data_primeiro_contato`
- `canal_aquisicao`
- `status` - Ativo/Inativo/Bloqueado

**Relacionamentos:**

- [[Endereços]] (1:N)
- [[Contatos]] (1:N)
- [[Documentos]] (1:N)
- [[Beneficiários]] (1:N)
- [[Apólices]] (1:N)

**Domínio**
[[Roteiros/Discussões guiadas/mesh/domínios/Clientes|Clientes]]

**Perguntas de Descoberta:**

- "Como vocês identificam se um mesmo cliente tem múltiplas apólices?"
- "Existe diferença entre segurado e proprietário do bem?"
- "Como tratam pessoas que são tanto clientes quanto corretores?"