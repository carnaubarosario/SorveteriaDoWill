# Sorveteria do Will — Data Warehouse & BI

Projeto de inteligência comercial construído do zero para uma fábrica de sorvetes de Alagoas. O objetivo é transformar dados brutos de vendas em respostas claras para a operação: quais clientes estão sumindo, quais rotas dão prejuízo, quando preparar estoque e onde focar nos períodos de queda.

---

## Visão geral

| | |
|---|---|
| **Período dos dados** | Janeiro/2023 — Maio/2026 |
| **Volume** | ~487 mil registros de vendas |
| **Clientes** | 1.493 estabelecimentos |
| **Municípios** | 45 municípios em Alagoas |
| **Vendedores** | 13 vendedores em 5 rotas |
| **Faturamento registrado** | R$ 232 milhões |

---

## Stack

| Camada | Tecnologia |
|---|---|
| Armazenamento | PostgreSQL 14+ |
| Pipeline ETL | Python 3.8+ (pandas, psycopg2) |
| Visualização | Power BI Desktop |
| Modelagem | Star Schema (modelo estrela) |

---

## Arquitetura

```
CSV (vendas)          CSV (supervisores)
      │                      │
      └──────────┬───────────┘
                 ▼
          [ stg_vendas ]        ← staging area
                 │
      ┌──────────▼──────────┐
      │   Transformações     │   Python / pandas
      │   Limpeza de dados   │
      └──────────┬──────────┘
                 │
    ┌────────────▼────────────┐
    │      dim_cliente        │
    │      dim_produto        │
    │      dim_vendedor       │   PostgreSQL
    │      dim_localizacao    │   (Star Schema)
    │      dim_tempo          │
    │         ▼               │
    │      fat_vendas         │
    └────────────┬────────────┘
                 │
    ┌────────────▼────────────┐
    │   vw_vendas_completa    │   View analítica
    └────────────┬────────────┘
                 │
    ┌────────────▼────────────┐
    │     Power BI (.pbix)    │   Dashboards
    └─────────────────────────┘
```

---

## Perguntas de negócio respondidas

- Quais clientes compram regularmente (pelo menos 1x por mês)?
- Quais clientes estão há 60 dias sem comprar e precisam de atenção?
- Quais clientes estão inativos há 90+ dias — e quanto isso representa em receita?
- Quais clientes têm alto ticket médio e merecem tratamento diferenciado?
- Quais são os produtos mais e menos vendidos?
- Quais rotas têm baixo faturamento e alto índice de devolução?
- Quais são os principais motivos de devolução?
- Onde existe sazonalidade? Quando preparar estoque?
- Quais vendedores e supervisores têm mais clientes em risco?

---

## Como rodar

### 1. Pré-requisitos

```bash
pip install -r requirements.txt
```

Versões necessárias: Python 3.8+, PostgreSQL 14+, Power BI Desktop.

### 2. Criar o banco

```sql
CREATE DATABASE dw_sorveteria;
```

Depois execute o script DDL para criar as tabelas, índices, foreign keys e a view analítica.

### 3. Configurar variáveis de ambiente

O ETL não aceita credenciais no código. Configure as variáveis antes de rodar:

**Windows (PowerShell):**
```powershell
$env:DB_HOST="localhost"
$env:DB_NAME="dw_sorveteria"
$env:DB_USER="postgres"
$env:DB_PASSWORD="sua_senha"
$env:ARQUIVO_SORVETES="C:\caminho\para\base_sorvetes_alagoas_v4.csv"
$env:ARQUIVO_SUPERVISORES="C:\caminho\para\base_supervisores_metas_v2.csv"
```

**Linux / Mac:**
```bash
export DB_HOST="localhost"
export DB_NAME="dw_sorveteria"
export DB_USER="postgres"
export DB_PASSWORD="sua_senha"
export ARQUIVO_SORVETES="/caminho/para/base_sorvetes_alagoas_v4.csv"
export ARQUIVO_SUPERVISORES="/caminho/para/base_supervisores_metas_v2.csv"
```

### 4. Rodar o ETL

```bash
python etl_dw_sorvetes.py
```

O pipeline executa nesta ordem: leitura dos CSVs → limpeza → staging → dimensões → fato → foreign keys. Logs gerados em `logs/etl_dw.log`.

### 5. Validar a carga

```sql
SELECT COUNT(*) FROM fat_vendas;
SELECT COUNT(*) FROM vw_vendas_completa;

SELECT
    SUM(valor_liquido)     AS faturamento_liquido,
    SUM(quantidade_liquida) AS volume_liquido,
    SUM(valor_devolucao)   AS total_devolvido
FROM vw_vendas_completa;
```

### 6. Abrir o dashboard

Abra `Will Dashboard.pbix` no Power BI Desktop, atualize a conexão com o PostgreSQL e clique em Atualizar.

---

## Estrutura do projeto

```
Sorveteria do Will/
├── Bases/
│   ├── base_sorvetes_alagoas_v4.csv
│   └── base_supervisores_metas_v2.csv
├── Consultas SQL/
│   ├── Produtos mais vendidos.txt
│   ├── Clientes com 90+ dias sem compra.txt
│   ├── Principais motivos de devolução.txt
│   ├── Qual período tem pico e queda de vendas.txt
│   ├── Vendedores com mais clientes 60+ dias sem compra.txt
│   └── Supervisores com mais clientes em risco.txt
├── Documentações/
│   ├── Dicionário de Dados.docx
│   ├── Documentação Script SQL.docx
│   └── Script ETL.docx
├── Scripts/
│   └── ETL.ipynb
├── Will Dashboard.pbix
├── requirements.txt
└── README.md
```

---

## Documentação

| Documento | Conteúdo |
|---|---|
| `Dicionário de Dados.docx` | Todas as tabelas, colunas, tipos e regras de negócio |
| `Documentação Script SQL.docx` | DDL comentado: modelagem, índices, decisões técnicas |
| `Script ETL.docx` | Arquitetura do pipeline, fluxo de dados e tratamentos |

---

## Decisões técnicas relevantes

**Staging area** — os dados brutos entram primeiro na `stg_vendas` sem transformação. Só depois são carregados nas dimensões e na fato. Isso permite reprocessar sem perder rastreabilidade.

**Carga em batch** — o ETL carrega a staging em lotes de 50 mil linhas com `COPY`, evitando estouro de memória em bases grandes.

**Idempotência** — o pipeline pode ser reexecutado quantas vezes forem necessárias. Foreign keys são removidas antes do truncate e recriadas ao final. Conflitos na fato usam `ON CONFLICT DO UPDATE`.

**Padronização na origem** — campos como `estado` e `pais` vinham inconsistentes no CSV (`AL`, `Alagoas`, vazio). A correção é feita no ETL antes da staging, não no banco.

**View analítica** — `vw_vendas_completa` desnormaliza todas as dimensões em uma única view pronta para o Power BI, com métricas calculadas: `valor_liquido`, `quantidade_liquida`, `ticket_medio` e `pct_devolucao`.
