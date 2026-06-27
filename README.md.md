# ATIVIDADE — PIPELINE DE DADOS (Café Finance)

## OBJETIVO

O grupo deve desenvolver um pipeline de dados (ETL) completo utilizando Apache NiFi, PostgreSQL e Hue.

O objetivo é transformar dados financeiros brutos do **Café Finance** em informações úteis para análise de:

- rateio de despesas em grupo
- pagamentos pendentes e atrasados
- transparência em contas compartilhadas
- visão de gastos pessoais e coletivos
- saldo de participantes dentro de grupos financeiros

---

## CONTEXTO DO PROJETO

O **Café Finance** é uma plataforma digital de gestão financeira pessoal e grupal com foco em rateio de despesas.

A solução permite que usuários registrem receitas, despesas pessoais, despesas compartilhadas, grupos financeiros, pagamentos, pendências, saldos individuais e histórico de movimentações.

O dataset desta atividade simula eventos e movimentações financeiras com problemas de qualidade propositalmente inseridos, como valores nulos, datas inconsistentes, duplicatas, erros de digitação e outliers.

---

## OBJETIVOS PEDAGÓGICOS

O grupo deve:

- entender a qualidade dos dados financeiros
- justificar cada tratamento realizado
- transformar dados brutos em informação útil
- identificar problemas que impactam rateios, saldos e pagamentos
- gerar consultas SQL para apoiar decisões dentro da plataforma

---

## ETAPAS DA ATIVIDADE

### 1. Extrair CSV (NiFi)

- Utilizar o processador `GetHTTP`
- Buscar o arquivo CSV no link fornecido pelo professor
- O arquivo representa movimentações financeiras do Café Finance

---

### 2. Converter para XLSX e Salvar no HUE

- Utilizar `ConvertRecord` + `PutFile`
- Gerar arquivo Excel `.xlsx`
- Salvar o arquivo no ambiente do Hue

---

### 3. Análise ANTES da limpeza

Baixe o arquivo no Hue e abra no Excel.

Responder:

- Quantas linhas existem no dataset?
- Quantos valores nulos existem por coluna?
- Existem duplicatas? Como identificou?
- Existem valores inválidos? Quais?
- Existem inconsistências de formato?
- Existem outliers? Em quais colunas?
- Os outliers parecem erros de cadastro ou situações reais de negócio?
- Quais são os principais problemas de qualidade?
- Quais problemas podem afetar o cálculo do rateio?
- Quais problemas podem afetar o saldo final dos participantes?

---

### 4. Carregar dados brutos no PostgreSQL

Criar tabela `cafe_finance_raw`.

Todos os campos devem ser `TEXT`.

Campos sugeridos:

```sql
CREATE TABLE cafe_finance_raw (
    registro_id TEXT,
    data_movimentacao TEXT,
    cidade TEXT,
    uf TEXT,
    canal TEXT,
    dispositivo TEXT,
    tipo_movimentacao TEXT,
    categoria TEXT,
    grupo_financeiro TEXT,
    titulo_movimentacao TEXT,
    valor_total TEXT,
    forma_pagamento TEXT,
    pago_por TEXT,
    participantes_qtd TEXT,
    percentual_rateio TEXT,
    valor_por_participante TEXT,
    status_rateio TEXT,
    status_pagamento TEXT,
    data_vencimento TEXT,
    dias_atraso TEXT,
    comprovante_enviado TEXT,
    origem_registro TEXT,
    saldo_participante TEXT,
    avaliacao_experiencia TEXT,
    observacao TEXT
);
```

Inserir os dados sem tratamento.

---

### 5. Limpeza via SQL

Criar tabela `cafe_finance_movimentacoes_limpo`.

Tratar os seguintes pontos:

#### Datas

- Padronizar formatos de data
- Converter `data_movimentacao` para `DATE`
- Converter `data_vencimento` para `DATE`
- Remover ou separar datas inválidas, como datas impossíveis ou texto no lugar de data

#### Números

Converter vírgula para ponto, tratar `NA`, vazio e `null`, e converter os campos:

- `valor_total`
- `participantes_qtd`
- `percentual_rateio`
- `valor_por_participante`
- `dias_atraso`
- `saldo_participante`
- `avaliacao_experiencia`

#### Textos

Padronizar textos e corrigir erros de digitação.

Exemplos:

- `mg` → `MG`
- `sp` → `SP`
- `alimentaçao` → `alimentacao`
- `transportee` → `transporte`
- `moraddia` → `moradia`
- `contaz` → `contas`
- `cartao credito` → `cartao_credito`
- `transf` → `transferencia`
- `boelto` → `boleto`
- `abertto` → `aberto`
- `parciall` → `parcial`
- `fechdao` → `fechado`
- `pendete` → `pendente`
- `pagoo` → `pago`
- `atrasdo` → `atrasado`
- `falha pagto` → `falha_pagamento`

#### Booleanos

Padronizar o campo `comprovante_enviado`:

- `sim`, `SIM` → `true`
- `nao`, `NÃO`, vazio → `false` ou `null`, conforme justificativa do grupo

#### Duplicatas

- Remover duplicatas por `registro_id`
- Manter apenas um registro por identificador
- Justificar o critério usado para escolher o registro mantido

#### Outliers

Identificar valores muito fora do padrão em:

- `valor_total`
- `participantes_qtd`
- `percentual_rateio`
- `valor_por_participante`
- `dias_atraso`
- `saldo_participante`
- `avaliacao_experiencia`

O grupo deve classificar cada outlier como:

- erro de cadastro
- caso real extremo
- dado suspeito que precisa ser separado

#### Campos derivados

Criar os seguintes campos:

- `flag_rateio_em_aberto`
- `flag_pagamento_pendente`
- `flag_pagamento_atrasado`
- `flag_falha_pagamento`
- `flag_sem_comprovante`
- `valor_rateio_calculado`
- `diferenca_rateio`
- `classificacao_movimentacao`

Sugestões de classificação:

- `rateio_em_aberto`
- `pagamento_atrasado`
- `falha_pagamento`
- `saldo_negativo`
- `movimentacao_regular`

---

### 6. Gravar dados limpos no PostgreSQL

- Criar a tabela final
- Inserir os dados tratados
- Separar, se necessário, uma tabela de registros suspeitos ou outliers

Sugestões:

- `cafe_finance_movimentacoes_limpo`
- `cafe_finance_movimentacoes_suspeitas`

---

### 7. Consultar no PostgreSQL

Executar consultas SQL para gerar insights.

---

## ATIVIDADES SQL

### Atividade 1 — Rateios em aberto por grupo

Mostrar:

```grupo_financeiro, quantidade_rateios, rateios_em_aberto, valor_total_em_aberto, taxa_rateio_aberto```

---

### Atividade 2 — Pagamentos pendentes e atrasados

Mostrar:

```status_pagamento, quantidade_ocorrencias, valor_total, media_dias_atraso```

---

### Atividade 3 — Gastos por categoria

Mostrar:

```categoria, quantidade_movimentacoes, valor_total, valor_medio, percentual_do_total```

---

### Atividade 4 — Saldo dos participantes

Mostrar:

```pago_por, quantidade_movimentacoes, saldo_medio, saldo_minimo, saldo_maximo```

---

### Atividade 5 — Impacto da forma de pagamento

Mostrar:

```forma_pagamento, quantidade_movimentacoes, valor_medio, quantidade_falhas, taxa_falha_pagamento```

---

### Atividade 6 — Transparência e comprovantes

Mostrar:

```grupo_financeiro, quantidade_movimentacoes, comprovantes_enviados, sem_comprovante, taxa_sem_comprovante```

---

### Atividade 7 — Experiência do usuário por canal

Mostrar:

```canal, dispositivo, avaliacao_media, quantidade_movimentacoes, valor_medio_movimentado```

---

## INTERPRETAÇÃO

Responder:

### Rateio de despesas

- Quais grupos possuem mais rateios em aberto?
- Quais categorias concentram os maiores valores compartilhados?
- Existem inconsistências entre `valor_total`, `participantes_qtd` e `valor_por_participante`?
- Que ações poderiam reduzir conflitos no rateio?

### Pagamentos

- Quais status de pagamento aparecem com maior frequência?
- Os atrasos estão concentrados em algum grupo, categoria ou forma de pagamento?
- Quais melhorias poderiam ser feitas no fluxo de cobrança?

### Transparência

- A ausência de comprovantes pode comprometer a confiabilidade dos dados?
- Quais grupos possuem maior taxa de movimentações sem comprovante?
- Como o histórico de alterações poderia ajudar na auditoria?

### Qualidade dos dados

- Quais problemas de qualidade mais impactaram os resultados?
- Os outliers alteraram médias ou conclusões?
- Quais dados deveriam ser separados antes da análise final?
- Quais melhorias deveriam ser feitas no cadastro e nas validações da aplicação?

---

## RELATÓRIO FINAL

O grupo deve entregar:

### Pipeline

- Print do fluxo no NiFi
- Explicação de cada etapa
- Justificativa dos processadores utilizados

### Excel

- Respostas do diagnóstico antes da limpeza
- Identificação de nulos, duplicatas, inválidos e outliers

### Limpeza

- SQL utilizado
- Justificativas dos tratamentos
- Explicação dos outliers encontrados e decisão tomada

### Banco

- Print da tabela bruta
- Print da tabela limpa
- Print da tabela de suspeitos, se criada

### SQL

- Queries das atividades
- Prints dos resultados
- Interpretação dos indicadores

### Interpretação

- Respostas e conclusões sobre rateio, pagamentos, transparência e qualidade dos dados

---

## CONCLUSÃO

O grupo deve demonstrar capacidade de:

- analisar qualidade de dados financeiros
- justificar transformações em um pipeline ETL
- gerar dados confiáveis para análise
- identificar riscos em rateios e pagamentos compartilhados
- produzir insights úteis para a evolução do Café Finance
