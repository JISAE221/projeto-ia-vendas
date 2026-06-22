# Specs Deneb — IA Vendas

Visuais Vega-Lite (v5) prontos para colar no **Deneb** do Power BI. Todos usam
`{"data": {"name": "dataset"}}` (padrão do Deneb) e respondem **automaticamente**
aos slicers/filtros do relatório via contexto de filtro do PBI — basta arrastar os
campos abaixo para o painel **Data** do Deneb com os nomes indicados.

## Como usar
1. Adicione o visual **Deneb** ao relatório (AppSource).
2. Arraste os campos da coluna "Bind" abaixo para a área **Values** (papel `dataset`).
3. **Importante:** o nome do campo no painel deve bater com o nome usado no spec
   (ex.: a medida precisa se chamar exatamente `Total Receita`, a coluna `REGIAO` etc.).
   Renomeie em "Values" se necessário.
4. Edite o spec → cole o JSON correspondente → **Apply**.

## Mapeamento campo → visual

| Página | Arquivo | Tipo | Campos (nome no `dataset`) |
|---|---|---|---|
| Clientes | `clientes/clientes-por-regiao.json` | Barra | `REGIAO` (dCliente[REGIAO]), `Total Receita` |
| Clientes | `clientes/distribuicao-por-sexo.json` | Donut | `SEXO` (dCliente[SEXO]), `Clientes Únicos`* |
| Clientes | `clientes/top-clientes-receita.json` | Barra H (Top 10) | `NOME` (dCliente[NOME]), `Total Receita` |
| Produtos | `produtos/mix-por-categoria.json` | Donut % | `NOME` (dCategoria[NOME]), `Total Receita` |
| Produtos | `produtos/top-produtos-receita.json` | Barra H (Top 10) | `NOME` (dProduto[NOME]), `Total Receita` |
| Produtos | `produtos/margem-por-categoria.json` | Barra + média | `NOME` (dCategoria[NOME]), `Margem %` |
| Fornecedores | `fornecedores/top-fornecedores-custo.json` | Barra H (Top 10) | `NOME` (dFornecedor[NOME]), `Total Custo` |
| Fornecedores | `fornecedores/fornecedores-por-categoria.json` | Barra | `NOME` (dCategoria[NOME]), `Fornecedores`* |
| Regiões | `regioes/receita-por-regiao.json` | Barra + rótulo | `REGIAO` (dCliente[REGIAO]), `Total Receita` |
| Regiões | `regioes/top-estados-receita.json` | Barra H (Top 15) | `ESTADO`, `REGIAO`, `Total Receita` |
| Vendas | `vendas/receita-no-tempo.json` | Área/linha | `DATA` (dTempo[DATA]), `Total Receita` |
| Vendas | `vendas/receita-por-forma-pagamento.json` | Barra H | `FORMA` (dForma[FORMA]), `Total Receita` |
| Vendas | `vendas/receita-custo-lucro.json` | Multi-linha | `DATA`, `Total Receita`, `Total Custo`, `Lucro Bruto` |
| Vendedores | `vendedores/ranking-vendedores.json` | Barra H (Top 15) | `NOME` (dVendedor[NOME]), `Total Receita`, `Rank Vendedor` |

\* **Medida de contagem** — criar se ainda não existir, ex.:
`Clientes Únicos = DISTINCTCOUNT(dCliente[IDCLIENTE])` e
`Fornecedores = DISTINCTCOUNT(dFornecedor[IDFORNECEDOR])`.

## ⚠️ Dependências no modelo atual
Os seguintes campos usados nos specs **não existem hoje** no modelo e precisam ser criados:

- **`dTempo[DATA]`** — a dimensão `dTempo` foi removida do modelo (ver `docs/.../documentacao.md`).
  Os visuais de Vendas no tempo (`receita-no-tempo`, `receita-custo-lucro`) exigem recriar
  uma tabela calendário e o relacionamento `fVendas[IDTEMPO] → dTempo`.
- **`[Total Quantidade]`** e **`[Receita YTD]`** — citados no escopo mas ausentes em `_Medidas.tmdl`.
  Criar antes de usar visuais que dependam deles.
- Medidas de contagem `Clientes Únicos` / `Fornecedores` (ver acima).

Medidas que **já existem** em `_Medidas`: `Total Receita`, `Total Custo`, `Lucro Bruto`,
`Margem %`, `Rank Vendedor`, `Receita Eletronicos`.

## Notas de design
- Tema escuro alinhado ao Design System (`powerbi/tema/ia-vendas-tema.json`).
- Accent `#8B5CF6`; fundo do visual `#0E0E21`; fundo de página `#09091A`.
- `autosize: fit` → cada visual preenche o container do Deneb de forma responsiva.
- Fontes Space Grotesk/Inter caem para `Segoe UI` no Power BI Service (fontes não nativas).
