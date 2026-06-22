# DOCUMENTAÇÃO – REGISTRO DOS PROCESSOS

**Título:** Análise de Vendas | IA Vendas

**Disciplina:** Análise Exploratória de Dados

**Integrantes do Grupo:**
- Daniel
- Renato
- Ricardo
- Gabriel
- Felipe
- Matheus

---

## Objetivo

Este projeto visa a criação de um Dashboard para analisar o cenário das vendas realizadas do comércio **IA Vendas**. O projeto fornecerá insights sobre as vendas realizadas, vendedores com maior volume de vendas, fornecedores, produtos mais vendidos e outras métricas relevantes.

Com isso, é esperado que o grupo obtenha compreensão e vivência prática de Análise Exploratória de Dados e da construção de relatórios e dashboards através da ferramenta de visualização gráfica Power BI.

---

## INTRODUÇÃO

Este documento descreve os processos envolvidos na construção de um Dashboard em Power BI para o comércio IA Vendas, detalhando desde a configuração do ambiente de versionamento e exploração dos dados até a publicação e apresentação do dashboard ao público-alvo.

O projeto foi desenvolvido pelo Grupo 4 da disciplina de Análise Exploratória de Dados da Biopark Edu, utilizando um conjunto de dados composto por 8 tabelas (1 tabela fato e 7 dimensões) com aproximadamente 35.762 registros de vendas.

Para garantir organização, rastreabilidade e colaboração entre os membros, o projeto adotou controle de versão com **Git e GitHub**, estratégia de branches com `main`, `develop` e `feature/*`, além de uma esteira de CI/CD com **GitHub Actions** para validação automática de PRs e notificações no Discord.

---

## OBJETIVOS DO PROJETO

- Realizar Análise Exploratória de Dados sobre o cenário de vendas do comércio IA Vendas
- Responder perguntas de negócio: melhores clientes, vendedores, produtos, fornecedores e regiões
- Utilizar técnicas de limpeza e normalização de dados com Power Query
- Modelar os dados em esquema Snowflake no Power BI
- Desenvolver um dashboard interativo no Power BI para visualização e análise dos dados
- Publicar e apresentar o dashboard aos stakeholders

---

## 1. EXPLORAÇÃO DOS DADOS DISPONÍVEIS

### 1.1. Exploração dos Dados

**Objetivo:** Compreender a estrutura, tabelas, colunas e tipos de dados existentes nos arquivos disponíveis.

**Atividades:**
- Identificação das 8 tabelas e suas relações
- Revisão dos dados para identificar problemas de qualidade e necessidades de limpeza
- Mapeamento da hierarquia de vendedores (gerentes e subordinados)
- Identificação de colunas SCD (Slowly Changing Dimensions) com campos `INICIO` e `FIM`

**Ferramentas Utilizadas:**
- Power Query
- M Language
- Power BI Desktop

**Tabelas identificadas:**

| Arquivo | Tipo | Registros | Descrição |
|---|---|---|---|
| `fVendas.xlsx` | Fato | 35.762 | Itens de venda com receita, custo e lucro pré-calculados |
| `dCliente.xlsx` | Dimensão | 1.002 | Clientes com cidade, estado e região (SCD tipo 2) |
| `dProduto.xlsx` | Dimensão | 234 | Produtos com custo médio e valor unitário |
| `dVendedor.xlsx` | Dimensão | 24 | Vendedores com hierarquia de gerentes (self-join) |
| `dFornecedor.xlsx` | Dimensão | 42 | Fornecedores (SCD tipo 2) |
| `dNota.xlsx` | Dimensão | 25.400 | Notas fiscais |
| `dForma.xlsx` | Dimensão | 26 | Formas de pagamento |
| `dCategoria.xlsx` | Dimensão | 9 | Categorias de produtos |
| `dTempo` | Dimensão | — | Calendário (chave `IDTEMPO`): data, ano, mês, dia, dia da semana e estação do ano |

> **Nota (22/06/2026):** `dTempo` foi **reincorporada ao modelo**. A dimensão havia sido removida por orientação inicial do professor (sob o entendimento de que pertenceria a outro projeto), mas sua ausência inviabilizava a análise temporal — central para um dashboard de vendas (série histórica de receita, sazonalidade, mapa de calor por dia). Decidiu-se restaurá-la. Ver registro detalhado na seção 2.1.

**Problemas identificados:**
- `dCliente`, `dVendedor` e `dFornecedor` possuem colunas `INICIO` e `FIM` (SCD tipo 2) — filtrar apenas registros com `FIM = null` para obter o registro atual
- `dVendedor` possui self-join via coluna `IDGERENTE` com 3 gerentes (IDs 1, 10 e 16)

---

### 1.2. Medidas

**Descrição:** Medidas DAX criadas para análise exploratória dos dados.

**Ferramentas Utilizadas:**
- Power BI Desktop
- DAX (Data Analysis Expressions)

```dax
Total Receita = SUM(fVendas[TOTAL_ITEM])
```

```dax
Total Custo = SUM(fVendas[CUSTO_TOTAL])
```

```dax
Lucro Bruto = SUM(fVendas[LUCRO_TOTAL])
```

```dax
Margem % = DIVIDE([Lucro Bruto], [Total Receita], 0)
```

```dax
Total Quantidade = SUM(fVendas[QUANTIDADE])
```

---

## MODELAGEM DIMENSIONAL DE DADOS

**Tipo de modelo:** Snowflake Schema

O modelo foi estruturado com `fVendas` no centro, conectada às 8 dimensões. A diferença em relação ao Star Schema está na sub-dimensão: `dProduto` se liga a `dCategoria`, formando um nível adicional de granularidade. `dVendedor` possui auto-relacionamento via `IDGERENTE` para representar a hierarquia gerente → vendedor.

> **Nota (22/06/2026):** `dTempo` foi **reincorporada ao modelo** com relacionamento ativo `fVendas[IDTEMPO] → dTempo[IDTEMPO]` (N:1), habilitando a análise temporal. Os valores de `IDTEMPO` (anos 2101–2104 na origem, por erro de digitação) foram corrigidos para 2011–2014. Isso reverte a remoção registrada anteriormente na seção 2.1.

**Relacionamentos:**

| Tabela Origem | Chave | Tabela Destino | Cardinalidade |
|---|---|---|---|
| fVendas | IDCLIENTE | dCliente | N:1 |
| fVendas | IDVENDEDOR | dVendedor | N:1 |
| fVendas | IDPRODUTO | dProduto | N:1 |
| fVendas | IDFORNECEDOR | dFornecedor | N:1 |
| fVendas | IDNOTA | dNota | N:1 |
| fVendas | IDFORMA | dForma | N:1 |
| fVendas | IDTEMPO | dTempo | N:1 |
| dProduto | ID_CATEGORIA | dCategoria | N:1 |
| dVendedor | IDGERENTE | dVendedor | N:1 (self-join) |

**MER/DER do Modelo**
```mermaid
erDiagram
    fVendas {
        int IDNOTA FK
        int IDCLIENTE FK
        int IDVENDEDOR FK
        int IDFORMA FK
        int IDPRODUTO FK
        int IDFORNECEDOR FK
        int QUANTIDADE
        float TOTAL_ITEM
        float CUSTO_TOTAL
        float LUCRO_TOTAL
    }
 
    dCliente {
        int IDCLIENTE PK
        string NOME
        string SEXO
        string EMAIL
        date NASCIMENTO
        string CIDADE
        string ESTADO
        string REGIAO
        date INICIO
        date FIM
    }
 
    dVendedor {
        int IDVENDEDOR PK
        string NOME
        string SEXO
        int IDGERENTE FK
    }
 
    dProduto {
        int IDPRODUTO PK
        string NOME
        int ID_CATEGORIA FK
        float CUSTO_MEDIO
        float VALOR_UNITARIO
    }
 
    dCategoria {
        int IDCATEGORIA PK
        string NOME
    }
 
    dFornecedor {
        int IDFORNECEDOR PK
        string NOME
        date INICIO
        date FIM
    }
 
    dNota {
        int IDNOTA PK
    }
 
    dForma {
        int IDFORMA PK
        string FORMA
    }
 
    dTempo {
        int IDTEMPO PK
        date DATA
        int ANO
        int MES
        int DIA
        string DIASEMANA
        string ESTACAOANO
    }
 
    fVendas }o--|| dCliente : "IDCLIENTE"
    fVendas }o--|| dVendedor : "IDVENDEDOR"
    fVendas }o--|| dProduto : "IDPRODUTO"
    fVendas }o--|| dFornecedor : "IDFORNECEDOR"
    fVendas }o--|| dNota : "IDNOTA"
    fVendas }o--|| dForma : "IDFORMA"
    fVendas }o--|| dTempo : "IDTEMPO"
    dProduto }o--|| dCategoria : "ID_CATEGORIA"
    dVendedor }o--o| dVendedor : "IDGERENTE (self-join)"
```
---

## 2. CONSTRUÇÃO DO DASHBOARD

### 2.1. Importação dos Dados

**Responsável:** Renato

*10/06/2026*
- Carregado os 9 arquivos .xlsx no Power BI
    - dCategoria
    - dCliente
    - dForma
    - dFornecedor
    - dNota
    - dTempo
    - dProduto
    - dVendedor
    - fVendas
- Ajustado as tipagens das colunas (e.g, colunas de ID que vieram como texto foram trocadas para números inteiros)
- Verificado pelas colunas **FIM** para que não sejam incluídas as entradas com valores não-nulos

*11/06/2026*
- Alterado as datas da coluna **IDTEMPO** em fVendas
    - As datas originalmente estavam datadas com anos de 2101 - 2104, então assumimos que houve um erro de digitação e que as datas deveriam ser dos anos 2011 - 2014
    - Convertemos elas para seu formato numérico (inteiro que representa a diferença de dias entre a data atual e janeiro de 1900) e então subtraímos a quantia de dias para subtrair exatamente 90 anos (considerando os 21 dias adicionais provindos de anos bissextos)
- Tratadas as duplicatas
    - Na tabela dProduto, haviam 2 entradas diferentes com IDs iguais. Para tratá-la, apenas alteramos manualmente o ID da entrada cuja coluna **INICIO** tinha o maior valor
    - Todas as outras duplicatas eram entradas com exatamente os mesmos valores, e portanto, foram excluídas
- Não foram encontrados valores nulos nas tabelas (com exceção das colunas **FIM**)
- Ajustado o relacionamento entre tabelas para seguir o modelo SnowFlake

*17/06/2026*
- Removida a tabela `dTempo` do modelo a orientação do professor — o arquivo `dTempo.xlsx` pertence a outro projeto e não deve ser utilizado no IA Vendas
- Desfeito o tratamento das datas da coluna **IDTEMPO** em `fVendas`, retornando aos valores originais
- O relacionamento `fVendas → dTempo` foi removido; a coluna `IDTEMPO` permanece em `fVendas` mas sem vínculo ativo no modelo

*22/06/2026*
- **Reincorporada a dimensão `dTempo`** ao modelo, revertendo a remoção de 17/06. Justificativa: a ausência de uma dimensão de tempo inviabilizava a análise temporal de vendas (série histórica de receita, sazonalidade, mapa de calor por dia), que é central ao objetivo do dashboard.
- **Tratamento dos outliers de data** na coluna **IDTEMPO**: os registros com anos 2101–2104 (erro de digitação na origem) foram corrigidos para 2011–2014 aplicando `Date.AddYears(_, -90)` às datas com ano > 2100 (o `Date.AddYears` já ajusta automaticamente os anos bissextos).
- Reativado o relacionamento `fVendas[IDTEMPO] → dTempo` (N:1).
- Desativada a **Detecção automática de data/hora** do Power BI para eliminar as tabelas de data redundantes geradas automaticamente (que inflavam o modelo e causavam instabilidade no carregamento).

---

### 2.2. Construção do Dashboard no Power BI

> _[Gabriel, Felipe e Matheus — descrever a construção de cada relatório]_

---

### 2.3. Medidas

# Atualização — 11 de junho de 2026

**Responsável:** Daniel (Tech Lead)

---

## Modelagem Snowflake

- Revisão e correção dos relacionamentos no Power BI
- Corrigido relacionamento `fVendas (IDNOTA) → dNota (IDNOTA)` que estava com direção invertida
- Confirmados 7 relacionamentos ativos com cardinalidade `*:1`
- `dTempo` removida do modelo a orientação do professor (ver seção 2.1)

**Relacionamentos finalizados:**

| De | Chave | Para | Cardinalidade |
|---|---|---|---|
| fVendas | IDCLIENTE | dCliente | *:1 |
| fVendas | IDFORMA | dForma | *:1 |
| fVendas | IDFORNECEDOR | dFornecedor | *:1 |
| fVendas | IDNOTA | dNota | *:1 |
| fVendas | IDPRODUTO | dProduto | *:1 |
| fVendas | IDVENDEDOR | dVendedor | *:1 |
| dProduto | ID_CATEGORIA | dCategoria | *:1 |

---

## Medidas DAX

Criação das medidas base na tabela `_Medidas`:

```dax
Total Receita = SUM(fVendas[TOTAL_ITEM])
```

```dax
Total Custo = SUM(fVendas[CUSTO_TOTAL])
```

```dax
Lucro Bruto = SUM(fVendas[LUCRO_TOTAL])
```

```dax
Margem % = DIVIDE([Lucro Bruto], [Total Receita], 0)
```

```dax
Total Quantidade = SUM(fVendas[QUANTIDADE])
```

```dax
Rank Vendedor = RANKX(ALL(dVendedor[NOME]), [Total Receita], , DESC, DENSE)
```

---

## Organização

- Criada tabela `_Medidas` separada das dimensões
- Todas as medidas movidas de `dCategoria` para `_Medidas`
- Convenção: prefixo `_` garante que a tabela aparece no topo do painel de campos

---

## GitHub

- Commit `feat: adiciona tabela _Medidas e medidas DAX base` na `main`
- `develop` sincronizada com a `main`
- Arquivo `_Medidas.tmdl` criado em `powerbi/ia-vendas.SemanticModel/definition/tables/`

```dax
-- Exemplo:
Soma = SUM(tabela[coluna])
```

---

### 2.4. Design e Desenvolvimento

> _[Descrever as visualizações criadas e as decisões de design]_

> 📸 _[Inserir prints dos dashboards finalizados]_

---

### 2.5. Validação das Visualizações

> _[Descrever como foram validadas as visualizações e os dados]_

---

### 2.6. Publicação do Dashboard no Power BI Service

#### 2.7. Publicação

> _[Descrever o processo de publicação no Power BI Service / Fabric]_

> 📸 _[Inserir print da publicação]_

#### 2.8. Compartilhamento

> _[Descrever como o dashboard foi compartilhado]_

---

### 2.9. Apresentação do Dashboard ao Público-Alvo

#### 2.10. Preparação da Apresentação

> _[Descrever como a apresentação foi preparada]_

#### 2.11. Demonstração

> _[Descrever os principais insights apresentados]_

> 📸 _[Inserir prints da apresentação final]_

---

## Repositório

[https://github.com/JISAE221/projeto-ia-vendas](https://github.com/JISAE221/projeto-ia-vendas)

---

*Disciplina: Análise Exploratória de Dados — Biopark Edu · Grupo 4*