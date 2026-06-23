# Specs Deneb — Versão Mobile (IA Vendas)

Variantes **otimizadas para o layout de celular** do Power BI dos visuais em
`powerbi/deneb/`. Mesmos dados, mesmos *binds* e mesmo tema escuro — apenas
ajustados para tiles estreitos (retrato).

## O que muda em relação ao desktop
- **Título** 14→12px e **subtítulo removido** (economia vertical).
- **Eixos** 11→9px e **títulos de eixo removidos** (`title: null`).
- **Top N reduzido**: Top 10 → **Top 6**, Top 15 → **Top 8**.
- **Donuts**: legenda movida para **baixo** (horizontal/colunas) e raios menores.
- **Barras de categoria**: passam a **horizontais** quando havia rótulos rotacionados
  (margem por categoria, fornecedores por categoria) — leem melhor no celular.
- **Séries temporais**: menos ticks (`tickCount: 4`, `labelOverlap`) e pontos menores.
- `autosize: fit` mantido → o visual preenche o tile do Deneb.

## Como aplicar (Power BI Desktop)
1. Abra o relatório e ative a **Visualização → Layout para celular** (ícone de celular).
2. Para cada visual Deneb, arraste-o para o canvas mobile (os mesmos campos do desktop
   já vêm vinculados — ver tabela de binds no `../README.md`).
3. Edite o spec do Deneb → cole o JSON correspondente desta pasta → **Apply**.
4. Repita por página. O layout mobile aparece no **app do Power BI** e no
   **Power BI Service** quando aberto em telas estreitas.

> ⚠️ O embed "publish to web" (iframe usado no site em `web/site/`) **sempre mostra o
> canvas desktop** — o layout mobile do Power BI só é renderizado no app/serviço.
> A responsividade do site em si é tratada no HTML (barra inferior em telas ≤768px).

## Mapeamento (mesma origem do desktop)
| Página | Arquivo | Adaptação mobile |
|---|---|---|
| Clientes | `clientes/clientes-por-regiao.json` | rótulo de valor na barra |
| Clientes | `clientes/distribuicao-por-sexo.json` | legenda embaixo, donut menor |
| Clientes | `clientes/top-clientes-receita.json` | Top 6 |
| Produtos | `produtos/mix-por-categoria.json` | legenda 3 colunas embaixo |
| Produtos | `produtos/top-produtos-receita.json` | Top 6 |
| Produtos | `produtos/margem-por-categoria.json` | barras horizontais + média |
| Fornecedores | `fornecedores/top-fornecedores-custo.json` | Top 6 |
| Fornecedores | `fornecedores/fornecedores-por-categoria.json` | barras horizontais |
| Regiões | `regioes/receita-por-regiao.json` | rótulo de valor na barra |
| Regiões | `regioes/top-estados-receita.json` | Top 8, legenda embaixo |
| Vendas | `vendas/receita-por-forma-pagamento.json` | rótulo reduzido |
| Vendas | `vendas/receita-no-tempo.json` | menos ticks, pontos menores |
| Vendas | `vendas/receita-custo-lucro.json` | legenda embaixo, menos ticks |
| Vendedores | `vendedores/ranking-vendedores.json` | Top 8, rótulo de valor |

As dependências de modelo (`dTempo[DATA]`, `Clientes Únicos`, `Fornecedores`, etc.)
são as mesmas do `../README.md`.
