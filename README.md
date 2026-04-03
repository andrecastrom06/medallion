# Projeto de Engenharia de Dados: E-Commerce Data Lakehouse (Arquitetura Medalhão) 🚀
Este repositório contém a solução para a Atividade de Engenharia do Rocket Lab 2026.1. O objetivo do projeto foi estruturar os dados de um grande e-commerce (baseado no dataset Olist do Kaggle) construindo um processo completo de ETL utilizando a arquitetura Data Lakehouse no Databricks.

## 📌 Visão Geral do Projeto
O pipeline extrai dados transacionais brutos e cotações de moedas, transformando-os através das camadas Bronze, Silver e Gold para fornecer Data Marts analíticos prontos para a tomada de decisão das áreas Comercial e de Relacionamento com Clientes.

## Tecnologias Utilizadas:
* Databricks (Ambiente de processamento e orquestração)
* PySpark & Spark SQL (Processamento distribuído de dados)
* Delta Lake (Armazenamento ACID, versionamento e otimização)
* Python (Extração de dados via API)

## 🏗️ Arquitetura Medalhão
O pipeline foi construído seguindo rigorosamente o padrão de três camadas:

### 🥉 Camada Bronze (Ingestão)
Responsável por receber os dados em seu formato original, garantindo o histórico exato do momento da ingestão.

Fontes de Dados: Arquivos CSV do dataset Olist carregados como Volumes no Databricks e dados da API do Banco Central.
Transformações: Nenhuma alteração de tipagem, apenas a adição da coluna timestamp_ingestion em todas as tabelas para rastreabilidade.

Desafio Superado (API BCB): Devido a limitações de NAT Gateway (acesso à internet) em instâncias gratuitas/comunitárias do Databricks, a requisição direta à API do Banco Central via notebook falhou. A solução aplicada foi extrair o JSON localmente via script Python e subi-lo como volume (cotacao_dolar.json), processando-o com sucesso via PySpark no notebook.

### 🥈 Camada Silver (Limpeza e Padronização)
Responsável por transformar os dados brutos em um formato padronizado, limpo e tipado, pronto para relacionamentos.

Tradução e Padronização: Nomes de colunas e status de pedidos/pagamentos traduzidos do inglês para o português. Padronização de Estados e Cidades para Upper Case.
Deduplicação Sênior: Utilização de Window Functions (row_number() over window) particionadas por IDs e ordenadas pelo timestamp_ingestion mais recente para eliminar registros duplicados de consumidores, produtos e vendedores.

Tratamento de Cotações: Criação de um calendário contínuo com Window Function (last(ignorenulls=True)) para preencher as cotações do dólar em finais de semana utilizando o valor de fechamento da sexta-feira anterior.

Tabela Fato Consolidada: Criação da silver.fat_pedido_total unindo informações de pedidos, pagamentos e a conversão de BRL para USD.

### 🥇 Camada Gold (Data Marts de Negócio)
Camada final com tabelas agregadas para responder às perguntas de negócio estabelecidas pelas diretorias.

* Projeto 1: Visão Comercial (gold.fat_vendas_comercial)

Agrupamento de vendas por Ano, Mês e Categoria.

Métricas calculadas: Contagem de pedidos, quantidade de itens, receita total (BRL e USD arredondados em 2 casas decimais) e Ticket Médio.

Exibição em tela dos Top 5 Produtos Mais e Menos Vendidos.

* Projeto 2: Satisfação de Clientes (gold.fat_avaliacoes_clientes)

Avaliação de performance cruzando categorias, vendedores e notas.

Métricas calculadas: Total de avaliações, média de notas, contagem de promotores (notas >= 4) e detratores (notas <= 2), gerando um % de satisfação.

Exibição em tela dos Melhores e Piores Produtos e Vendedores, utilizando um critério de desempate por volume de avaliações.

## ⚙️ Orquestração (Databricks Workflows)
Todo o pipeline foi automatizado utilizando o Databricks Jobs (Workflows).

<img width="1919" height="962" alt="Captura de tela 2026-04-03 125255" src="https://github.com/user-attachments/assets/f2573156-c8df-443e-b486-0391c4018711" />

Estrutura do Job: As tarefas foram configuradas de forma sequencial com dependências estritas: Landing_Bronze ➔ Bronze_Silver ➔ Silver_Gold.

Agendamento (Trigger): Configurado para rodar diariamente às 13:00 (Fuso horário America/Recife via Expressão Cron).

Boas Práticas: Os notebooks foram configurados com tolerância a mudanças de esquema (.option("overwriteSchema", "true")) e as cargas ocorrem em modo overwrite nas tabelas consumidas.

O arquivo de configuração exportado (Medalhão JOB.yaml) e a evidência de execução com sucesso estão presentes neste repositório.

## 📂 Estrutura do Repositório
Bronze.ipynb: Código PySpark para ingestão e criação do banco de dados Bronze.

Silver.ipynb: Transformações, limpezas e aplicação de regras de negócio.

Gold.ipynb: Gerações das agregações de negócio e queries de análise de Top 5.

Medalhão JOB.yaml: Arquivo de deploy do workflow do Databricks.
