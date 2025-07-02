# Data Product Lifecycle Events

## Domínio: -------

### Contexto de Negócio

**Entrevistados:**

- Maria Silva - Gerente de Operações de Apólices
- João Santos - Diretor de Produtos
- Ana Costa - Coordenadora de Atendimento
- Carlos Oliveira - Gerente de Compliance

---

### Problema de Negócio Atual

**Maria Silva (Operações):** "Hoje não temos visibilidade em tempo real do que está acontecendo com nossas apólices. Quando um corretor liga perguntando sobre o status de uma emissão ou endosso, precisamos entrar em 3 sistemas diferentes para rastrear onde está o processo. Às vezes descobrimos que uma apólice foi cancelada há dias e ninguém foi notificado."

**João Santos (Produtos):** "Não consigo saber quantas apólices foram emitidas hoje, quantas renovações estão pendentes para os próximos 30 dias, ou qual é a taxa de cancelamento por produto. Recebo relatórios semanais que já estão desatualizados quando chegam em minhas mãos."

**Ana Costa (Atendimento):** "Os clientes reclamam que não são avisados quando sua apólice está próxima do vencimento, quando um endosso é processado, ou quando há alguma pendência. Acabamos sendo reativos em vez de proativos."

### Necessidades de Negócio

#### 1. Visibilidade Completa do Ciclo de Vida

**Requisito:** Precisamos saber exatamente o que aconteceu com cada apólice desde sua cotação até o encerramento.

**Eventos Críticos que Precisamos Rastrear:**

- Apólice emitida
- Apólice ativada (início de vigência)
- Endosso solicitado
- Endosso aprovado/rejeitado
- Pagamento recebido/em atraso
- Renovação iniciada
- Apólice renovada
- Apólice cancelada (com motivo)
- Apólice suspensa por inadimplência
- Apólice reativada
- Apólice expirada

**Maria Silva:** "Cada evento desses precisa ter data, hora, quem fez, e o que mudou. É fundamental para auditoria."

#### 2. Notificações e Alertas Automáticos

**Ana Costa:** "Precisamos avisar automaticamente:"

- **Cliente:**
    - 60, 30 e 7 dias antes do vencimento
    - Quando endosso é processado
    - Quando pagamento não é identificado
    - Quando apólice é suspensa

- **Corretor:**
    - Quando apólice do seu cliente é emitida
    - Alertas de renovação dos seus clientes
    - Mudanças de comissão

- **Interno:**
    - Apólices com pagamento atrasado
    - Endossos pendentes há mais de 48h
    - Cancelamentos em massa (possível problema)

### Casos de Uso Prioritários

#### Caso 1: Renovação Proativa

**João Santos:** "Quero que 90 dias antes do vencimento, já tenhamos uma lista de todas as apólices para renovar, com o histórico de sinistros e pagamentos. Hoje perdemos 15% das renovações porque abordamos o cliente tarde demais."

**Informações Necessárias:**

- Lista de apólices vencendo nos próximos 90/60/30 dias
- Histórico de sinistralidade
- Histórico de pagamento
- Mudanças de preço sugeridas
- Últimas interações com o cliente

#### Caso 2: Gestão de Inadimplência

**Maria Silva:** "Precisamos agir rápido quando um pagamento atrasa. As regras são:"

- 5 dias de atraso: Notificar cliente
- 10 dias: Notificar corretor
- 15 dias: Suspender apólice
- 30 dias: Iniciar processo de cancelamento
- 45 dias: Cancelar apólice

"Hoje esse processo é manual e perdemos muito dinheiro com atrasos."

#### Caso 3: Análise de Cancelamentos

**João Santos:** "Preciso entender por que estamos perdendo clientes:"

- Motivos de cancelamento
- Tempo de vida da apólice antes do cancelamento
- Se houve sinistros
- Se houve problemas de pagamento
- Se mudou de seguradora ou saiu do mercado

#### Caso 4: Compliance e Auditoria

**Carlos Oliveira:** "A SUSEP pode pedir a qualquer momento o histórico completo de uma apólice. Preciso saber:"

- Todos os eventos com timestamp
- Quem autorizou cada mudança
- Documentos anexados em cada etapa
- Comunicações enviadas ao cliente
- Valores de prêmio e comissão em cada alteração

### Regras de Negócio Específicas

#### Temporalidade dos Eventos

**Maria Silva:** "Alguns eventos têm regras específicas de tempo:"

1. **Emissão**: Deve acontecer em até 24h após aprovação
2. **Endosso**: Processamento em até 48h
3. **Cancelamento**:
    - A pedido do cliente: Imediato
    - Por inadimplência: Após 45 dias
    - Por sinistralidade: Apenas no vencimento
4. **Reativação**: Apenas até 30 dias após suspensão

#### Hierarquia de Eventos

**João Santos:** "Alguns eventos geram outros automaticamente:"

- Pagamento Atrasado → Suspensão → Cancelamento
- Solicitação de Endosso → Análise → Aprovação/Rejeição → Emissão
- Aproximação do Vencimento → Oferta de Renovação → Renovação/Não Renovação

### Volumetria e Performance Esperada

**Maria Silva forneceu os números:**

- 2 milhões de apólices ativas
- 15.000 novos contratos/mês
- 25.000 endossos/mês
- 12.000 renovações/mês
- 3.000 cancelamentos/mês
- 200.000 eventos totais/mês (média)

**Picos de Movimento:**

- Início do mês: 3x o volume normal (pagamentos)
- Final do mês: 2x o volume normal (renovações)
- Janeiro: 4x o volume normal (renovação de frotas)

### Critérios de Sucesso

**João Santos:** "O projeto será um sucesso se conseguirmos:"

1. **Reduzir em 50% o tempo** para responder perguntas sobre status de apólices
2. **Aumentar renovações em 10%** através de abordagem proativa
3. **Reduzir inadimplência em 20%** com ações automatizadas
4. **Eliminar multas de compliance** por falta de informação
5. **Zerar reclamações** sobre falta de comunicação

### Integrações Necessárias

**Maria Silva:** "Precisamos que os eventos alimentem automaticamente:"

- **Sistema de Cobrança:** Para gerar faturas e controlar pagamentos
- **CRM:** Para registrar todas as interações
- **Portal do Cliente:** Para mostrar timeline da apólice
- **Portal do Corretor:** Para acompanhamento de carteira
- **Sistema de Comissões:** Para calcular pagamentos
- **Sinistros:** Para validar cobertura vigente
- **BI Corporativo:** Para análises gerenciais

### Expectativas de Prazo

**João Santos:** "Precisamos de algo funcionando em 3 meses. Pode ser faseado:"

- **Mês 1:** Eventos básicos (emissão, cancelamento, renovação)
- **Mês 2:** Notificações automáticas
- **Mês 3:** Dashboard gerencial
- **Mês 4+:** Análises avançadas e predições

### Preocupações e Restrições

**Carlos Oliveira (Compliance):**

- "Todos os dados são sensíveis. Precisamos log de quem acessou o quê"
- "Dados de apólice de seguro de vida têm regras especiais de sigilo"
- "Precisamos manter histórico por 5 anos no mínimo"
- "LGPD: cliente pode pedir exclusão de dados pessoais"

**Ana Costa (Atendimento):**

- "Não podemos bombardear o cliente com notificações"
- "Precisa respeitar preferências de contato (WhatsApp, SMS, E-mail)"
- "Horário de notificações: apenas 8h às 20h"

### Impacto Financeiro Esperado

**João Santos apresentou a projeção:**

- **Aumento de renovações:** R$ 2 milhões/ano
- **Redução de inadimplência:** R$ 1,5 milhão/ano
- **Redução de custos operacionais:** R$ 800 mil/ano
- **Multas evitadas:** R$ 500 mil/ano
- **Total:** R$ 4,8 milhões/ano

"O investimento se paga em 8 meses."

### Observações Finais dos Entrevistados

**Maria Silva:** "O mais importante é que seja em tempo real. Não adianta saber amanhã que algo aconteceu hoje."

**João Santos:** "Quero dashboards que eu possa acessar do celular. Preciso ver os números em qualquer lugar."

**Ana Costa:** "A interface para consultar eventos precisa ser simples. Minha equipe não é técnica."

**Carlos Oliveira:** "Qualquer solução precisa passar pela área de segurança da informação e compliance antes de ir para produção."