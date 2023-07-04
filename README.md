![image](https://github.com/adaylson99/Projeto_Data_analitycs_digital_college/assets/137455643/5648f457-ed49-4f62-9cbc-3230d9cae1b6)

# Projeto: Análise de Dados Pagamento de Empenhos - Governo do Estado do Ceará.
Está contido nesse repositório informações e ferramentas para análise de dados de pagamento de empenho provenientes do governo do estado do Ceará. Esse é um projeto que faz parte da finalização do Módulo 1 do curso de Data Analytics da Digital College para demonstração de conhecimento adquirido até aqui. 


## 1. Significado das siglas da base.

### 📋 Sumário

- [1- IMPORTAÇÃO](#importação)
- [2- ANÁLISE INÍCIAL](#análiseInícial)
- [3- TRANSFORMAÇÃO](#transformacao)
- [4- MODELAGEM LÓGICA](#modelagemlógica)
- [5- ANÁLISE EXPLORATÓRIA](#analiseexploratoria)
- [6- LIMPEZA INICIAL](#limpezainicial)
- [7- DOCUMENTAÇÃO PROJETO](#documentacaoprojeto)
- [7- DOCUMENTAÇÃO PROJETO](#documentacaoprojeto)
- 



## 🚀 Começando

Neste projeto, realizamos a análise de dados de pagamento de empenho disponibilizados pelo governo do estado do Ceará. O objetivo é extrair insights relevantes a partir desses dados, identificando padrões de gastos, tendências e possíveis áreas de melhoria na gestão financeira.

> **O projeto é dividido em várias etapas principais:**

1. **Restauração do Banco de Dados**: Nesta etapa, restauramos o banco de dados, disponibilizado pela Digital College, no programa PostgreSQL.

2. **Criação**: Após a instalação procuramos entender a base de dados e fizemos a criação do Dicionário de Dados, após isso fizemos a modelagem Conceitual e Lógiga da tabela execucao_financeira_despesa.

3. **Análise Exploratória SQL**: Nesta etapa, realizamos uma análise exploratória dos dados para obter uma compreensão inicial do conjunto de dados. Utilizamos consultas dinamicas e e funções em SQL para identificar padrões e valores nulos.

4. **Trasformação SQL**: Após a análise exploratória, fizemos a modelagem dimensional, com base na tabela do banco relacional e após isso criamos o banco de dados dimensional, juntamente com as tabelas dimensões e fato. Na criação das tabelas dimensionais e fato já fizemos o tratamento dos dados Nulos, Padrões, Valores Negativos e Duplicação, usando as funções ex: UPPER, COALESCE, TRIM, CONCAT.

4. **Medidas SQL**: Após a criação do banco dimensional, começamos a criar as medidas, conforme solicitado no projeto. Fizemos primeiro a análise de valores dos pagamentos, afim de encontrar divergências. Depois dessa análise, começamos a criar as medidas conforme exigido no projeto.

5. **Visualização dos Dados**: A visualização de dados desempenha um papel importante na comunicação dos resultados da análise. Utilizamos gráficos e visualizações interativas para apresentar os insights obtidos de forma clara e compreensível, utilizando a ferramenta PowerBI.

6. **Projeto em Python**: Afim de acrescentarmos informações no projeto, decidimos demonstrar que é possivel conectar o banco de dados e utilizarmos o Python para procedermos com as análises de dados.

Ao seguir essas etapas, buscamos obter uma compreensão aprofundada dos dados de pagamento de empenho do governo do estado do Ceará e fornecer insights valiosos para auxiliar na tomada de decisões informadas.


### 🛠️ Instalação/Ferramentas

> **Para elaboração do Projeto utilizamos as seguintes ferramentas:**

- **Visual Studio Code** - utilizado para organizar as etapas do projeto, testar as query em Sql e fazer o projeto em Python.
- **Postgres Sql** - Banco de Dados utilizado para restauração e manipulação.
- **BRModelo** - Site onde foi feita as modelagens conceitual e dimensional.
- **PowerBi** - Ferramenta utilizada para visualização das análises.
- **Github** - Plataforma de hospedagem de código, que utilizamos para subir o projeto.


## ⚙️ Exemplo de querys SQL do Projeto:

- **Analise do Banco**

```sql
SELECT * FROM information_schema.columns WHERE table_name = 'execucao_financeira_despesa' ORDER BY column_name ASC;

```

- **Transformação e Tratamento**

```sql
-- Tabela de Fato Empenho = fato_empenho

CREATE TABLE data_warehouse.fato_empenho AS
SELECT DISTINCT ON (codigo_empenho, ano_cadastro, codigo_credor, codigo_orgao)
    COALESCE(cod_ne, 'N/A') AS codigo_empenho,
    num_ano AS ano_cadastro,
    COALESCE(cod_credor, 'N/A') AS codigo_credor,
    COALESCE(codigo_orgao,'N/A') AS codigo_orgao,
    COALESCE(cod_np, 'N/A') AS codigo_numero_processo,
    COALESCE(cod_fonte, 'N/A') AS codigo_fonte,
    COALESCE(cod_funcao, 'N/A') AS codigo_funcao,
    COALESCE(cod_item_modalidade, 'N/A') AS codigo_item_modalidade,
    COALESCE(cod_programa, 'N/A') AS codigo_programa,
    COALESCE(cod_subfuncao, 'N/A') AS codigo_subfuncao,
    COALESCE(cod_item_grupo, 'N/A') AS codigo_item_grupo,
    COALESCE(cod_item_categoria, 'N/A') AS codigo_item_categoria,
    COALESCE(cod_item_elemento, 'N/A') AS codigo_item_elemento,
    COALESCE(cod_item, 'N/A') AS codigo_item,
    vlr_empenho AS valor_empenho,
    CASE
        WHEN valor_pago = vlr_empenho THEN valor_pago
        ELSE COALESCE(vlr_liquidado, 0.00)
    END AS valor_liquidado,
    COALESCE(valor_pago, 0.00) AS valor_pago,
    COALESCE(vlr_empenho - valor_pago, 0.00) AS valor_resto_a_pagar,
    dth_empenho AS data_empenho,
    dth_liquidacao AS data_liquidacao,
    dth_pagamento AS data_pagamento,
    dth_processamento AS data_processamento,
    COALESCE(num_sic, 'N/A') AS numero_identificacao_financeiro,
    CONCAT(num_ano::text, COALESCE(cod_ne, 'N/A'), COALESCE(codigo_orgao, 'N/A'), COALESCE(cod_credor, 'N/A')) AS id_fato_empenho
FROM public.execucao_financeira_despesa
WHERE 1 = 1;


-- Fazendo alteração na coluna data_liquidacao, para preencher com valores obedecendo a condição----

UPDATE data_warehouse.fato_empenho
SET data_liquidacao = data_pagamento
WHERE valor_liquidado > 0.00;

-- Fazendo alteração na coluna valor_empenho, para preencher apenas com os valores positivos----

UPDATE data_warehouse.fato_empenho
SET valor_empenho = 0.00
WHERE valor_empenho < 0.00;

-- Fazendo alteração na coluna valor_resto_a_pagar, para preencher com os valores do fato empenho, que tenha valor liquidado e pago igual a 0.
---Ele vai levar o mesmo valor do empenho para preencher o campo valor_resto_a_pagar.

UPDATE data_warehouse.fato_empenho
SET valor_resto_a_pagar = valor_empenho
WHERE valor_liquidado = 0.00 AND valor_pago = 0.00;

--Fazendo alteração na valor_resto_a_pagar, considerando a lógica de que quando o valor quando o (valor empenho - valor pagar) for um resultado negativo, 
--preencher o campo resto a pagar com 0.00 

```

- **Medidas**

```sql
SELECT
    f.codigo_item_grupo,
    g.descricao_item_grupo,
    f.codigo_item_categoria,
    c.descricao_item_categoria,
    SUM(f.valor_empenho) AS total_empenho,
    SUM(f.valor_liquidado) AS total_liquidado,
    SUM(f.valor_pago) AS total_pago
FROM (
    SELECT id_fato_empenho, codigo_item_grupo, codigo_item_categoria, valor_empenho, valor_liquidado, valor_pago
    FROM data_warehouse.fato_empenho
    WHERE valor_empenho >= 0
    GROUP BY id_fato_empenho, codigo_item_grupo, codigo_item_categoria, valor_empenho, valor_liquidado, valor_pago
    HAVING COUNT(*) = 1
) AS f
JOIN data_warehouse.dim_grupo g ON f.codigo_item_grupo = g.codigo_item_grupo
JOIN data_warehouse.dim_categoria c ON f.codigo_item_categoria = c.codigo_item_categoria
JOIN data_warehouse.fato_empenho fe ON f.id_fato_empenho = fe.id_fato_empenho
WHERE fe.data_empenho BETWEEN '2022-01-01' AND '2022-12-31' -- Período desejado
GROUP BY f.codigo_item_grupo, g.descricao_item_grupo, f.codigo_item_categoria, c.descricao_item_categoria
LIMIT 2;
```


## 🖇️ Projeto Python

Como um extra no projeto fizemos uma análise simples do banco de dados e elaboração de um gráfico conforme imagem abaixo:

([7- DOCUMENTAÇÃO PROJETO\imagembarra.png](https://github.com/NayaraWakewski/Projeto-Digitall-Equipe-3/blob/main/7-%20DOCUMENTA%C3%87%C3%83O%20PROJETO/imagembarra.png)
*Grafico de Barras Exemplo das Categorias por Valor Empenho.*


## :bar_chart: Dataviz

Para visualizar o dashboard do projeto, acessar o link abaixo:

[Dashboard de Análises - Empenhos Governo do Estado do Ceará](https://app.powerbi.com/groups/me/reports/cc6c6b41-5885-4c6f-9b54-30d217691028?ctid=ca76ac1c-cf3f-49e9-b6aa-128bad72b989&pbi_source=linkShare)


## ✒️ Equipe 3


* **Fernanda** - *Modelagem do projeto, criação de medidas, powerbi e documentação* - [Data Analytics]  Github (https://github.com/NayaraWakewski) | LinkedIn (https://www.linkedin.com/in/nayaraba/)
* **Adailson** - *Modelagem do Banco de Dados, criação das dimensões e fato* - [Data Analytics]
Github (https://github.com/ElniVeloso) | LinkedIn (https://www.linkedin.com/in/elni-veloso-191462228/)
* **Patricia Tavares** - *Analise Exploratória, conexão banco de dados* - [Data Analytics]
Github (https://github.com/Patricia-Tavares) | LinkedIn (https://www.linkedin.com/in/patr%C3%ADcia-tavares-06a92b199/)


## 🎁 Agradecimentos.

* Compartilhe com outras pessoas esse projeto 📢;
* Quer saber mais sobre o projeto? Entre em contato para tomarmos um :coffee:;
* Agradecimento ao nosso professor Alex Souza (https://github.com/aasouzaconsult) pelo conhecimento repassado;
* Agradecimento a Digital College pelo desafio proposto (https://digitalcollege.com.br/)

---
⌨️ com ❤️ por [Nayara Vakevskii](https://github.com/NayaraWakewski) 😊
