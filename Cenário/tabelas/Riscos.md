---
tags:
  - objeto
entidade: suporte
mestre: true
objeto: tabela
---
**Definição**: Objeto específico coberto pelo seguro. O bem segurado.

**Atributos Base:**

- `risk_id` (PK)
- `policy_id` (FK)
- `risk_type` - Veículo/Imóvel/Vida/Equipamento
- `descricao_risco`
- `valor_segurado`
- `localizacao_risco`

**Atributos Específicos por Tipo:**

**Para Veículos:**

- `placa`
- `chassi`
- `renavam`
- `marca`
- `modelo`
- `ano_fabricacao`
- `ano_modelo`
- `combustivel`
- `zero_km`
- `uso_veiculo` - Particular/Comercial/App
- `pernoite_garagem`
- `cep_pernoite`
- `km_media_mensal`

**Para Imóveis:**

- `tipo_imovel` - Casa/Apartamento/Comercial
- `endereco_completo`
- `area_construida`
- `ano_construcao`
- `tipo_construcao` - Alvenaria/Madeira/Mista
- `uso_imovel` - Residencial/Comercial/Veraneio
- `possui_alarme`
- `possui_cerca_eletrica`