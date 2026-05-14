# Ordem de Execução do Projeto — Sorveteria do Will

Este documento descreve a sequência recomendada para configurar, executar e validar o projeto de Data Warehouse e Business Intelligence da Sorveteria do Will.

---

## 1. Preparar o ambiente

Antes de executar o projeto, certifique-se de ter instalado:

- Python 3.8 ou superior
- PostgreSQL 14 ou superior
- Microsoft Power BI Desktop
- Bibliotecas Python listadas no arquivo `requirements.txt`

Instalação das dependências:

```bash
pip install -r requirements.txt
```

---

## 2. Criar o banco de dados PostgreSQL

Crie o banco de dados utilizado pelo projeto:

```sql
CREATE DATABASE dw_sorveteria;
```

Após criar o banco, conecte-se a ele antes de executar os scripts SQL.

---

## 3. Executar o script DDL

Execute o script SQL responsável por criar a estrutura do Data Warehouse:

- tabelas dimensão
- tabela fato
- tabela de staging
- índices
- chaves estrangeiras
- view analítica `vw_vendas_completa`

A execução do DDL deve ocorrer antes do ETL.

---

## 4. Conferir os arquivos de entrada

Verifique se as bases estão disponíveis na pasta `Bases`:

- `base_sorvetes_alagoas_v4.csv`
- `base_supervisores_metas_v2.csv`

Esses arquivos são utilizados pelo pipeline ETL para carregar vendas, devoluções, clientes, produtos, vendedores, supervisores, rotas e datas.

---

## 5. Ajustar os caminhos no script ETL

Antes de rodar o pipeline, confira no script Python:

- caminho da base de vendas
- caminho da base de supervisores
- host do PostgreSQL
- nome do banco
- usuário
- senha

Recomenda-se usar variáveis de ambiente em versões futuras para evitar credenciais fixas no código.

---

## 6. Executar o pipeline ETL

Execute o script ETL principal:

```bash
python etl_dw_sorvetes.py
```

O pipeline realiza:

1. leitura dos arquivos CSV;
2. limpeza e transformação dos dados;
3. carga na tabela de staging;
4. carga nas dimensões;
5. carga na tabela fato;
6. recriação das chaves estrangeiras;
7. geração de logs de execução.

---

## 7. Validar a carga no PostgreSQL

Após executar o ETL, valide se os dados foram carregados corretamente.

Exemplos de consultas:

```sql
SELECT COUNT(*) FROM public.stg_vendas;
SELECT COUNT(*) FROM public.fat_vendas;
SELECT COUNT(*) FROM public.vw_vendas_completa;
```

Também é recomendado validar os principais indicadores:

```sql
SELECT
    SUM(valor_liquido) AS faturamento,
    SUM(quantidade_liquida) AS quantidade,
    SUM(valor_devolucao) AS valor_devolvido,
    SUM(quantidade_devolucao) AS quantidade_devolvida
FROM public.vw_vendas_completa;
```

---

## 8. Abrir o dashboard no Power BI

Abra o arquivo:

```text
Will Dashboard.pbix
```

No Power BI Desktop:

1. confira a conexão com o PostgreSQL;
2. atualize as credenciais, se necessário;
3. clique em `Atualizar`;
4. valide se os visuais foram carregados corretamente.

---

## 9. Navegar pelas páginas do dashboard

O dashboard foi estruturado para responder perguntas comerciais e operacionais, incluindo:

- análise de clientes;
- clientes em atenção, pré-churn e churn;
- clientes VIP;
- produtos mais vendidos e produtos críticos;
- rotas com melhor e pior desempenho;
- sazonalidade e períodos de pico/queda;
- desempenho de vendedores e supervisores;
- devoluções por produto, motivo, rota e vendedor.

---

## 10. Consultar a documentação

A pasta `Documentações` contém materiais técnicos e funcionais do projeto, incluindo:

- documentação do script SQL;
- documentação do pipeline ETL;
- dicionário de dados;
- arquitetura do projeto;
- perguntas de negócio.

A documentação das medidas DAX pode ser adicionada progressivamente conforme evolução do projeto.

---

## 11. Ordem resumida

```text
1. Instalar dependências
2. Criar banco dw_sorveteria
3. Executar script DDL
4. Conferir bases CSV
5. Ajustar caminhos e credenciais no ETL
6. Executar pipeline ETL
7. Validar carga no PostgreSQL
8. Abrir e atualizar o PBIX
9. Navegar pelos dashboards
10. Consultar documentação técnica
```

---

## Observação

Este projeto foi desenvolvido com foco em inteligência comercial, análise de clientes, produtos, rotas, vendedores, supervisores, devoluções e sazonalidade, utilizando uma arquitetura baseada em Data Warehouse, pipeline ETL em Python, PostgreSQL e Power BI.
