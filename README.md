# IA Vendas — Dashboard de Análise de Vendas

> Projeto Aplicado — Análise Exploratória de Dados | Biopark Edu · Grupo 4

Dashboard interativo em Power BI para análise do cenário de vendas do comércio **IA Vendas**, respondendo perguntas estratégicas sobre clientes, vendedores, produtos, fornecedores e regiões.

---

## Equipe

| Membro | Papel | Função no Projeto |
| --- | --- | --- |
| Daniel | Tech Lead | Modelagem Snowflake, DAX, GitHub, publicação |
| Renato | Dev Backend | ETL Power Query, relacionamentos, documentação |
| Ricardo | IoT & Dados | Dicionário de dados, regras de negócio, docs |
| Gabriel | Dev Frontend | Dashboard Clientes & Regiões |
| Felipe | Eng. IA | Dashboard Produtos & Categorias |
| Matheus | Eng. IA | Dashboard Vendedores & Fornecedores |

---

## Perguntas respondidas

- Quais são os melhores clientes?
- Quais são os melhores vendedores?
- Qual categoria traz maior rendimento?
- Qual a relação com os fornecedores?
- Qual é o melhor e o pior produto?
- Em qual região se vende mais?

---

## Estrutura do repositório

```
ia-vendas/
├── data/                   # Arquivos xlsx originais
├── powerbi/
│   ├── ia-vendas.pbip      # Arquivo principal do relatório
│   ├── ia-vendas.Report/   # Definição PBIR das páginas
│   ├── ia-vendas.SemanticModel/  # Modelo semântico (TMDL)
│   ├── deneb-specs/        # Specs Vega-Lite/Vega versionados
│   ├── deneb/              # Variantes mobile dos specs Deneb
│   ├── html-viewer/        # HTMLs auxiliares usados no relatório
│   └── tema/               # Arquivo de tema (.json)
├── docs/                   # Documentação do projeto (PA)
├── web/
│   └── site/               # Site frontend (embed do Power BI)
└── README.md
```

---

## Stack

- **Power BI Desktop** — modelagem e dashboards
- **Power Query (M)** — ETL e limpeza dos dados
- **DAX** — medidas e KPIs
- **Snowflake Schema** — modelagem dimensional
- **Deneb (Vega-Lite / Vega)** — visuais customizados
- **HTML Content** — cards de KPI e tabelas via DAX
- **Azure Maps** — mapa de bolhas por estado
- **Power BI Service / Fabric** — publicação e embed
- **GitHub Actions** — CI/CD e notificações no Discord
- **GitHub** — versionamento

---

## Dados

| Arquivo | Tipo | Linhas |
| --- | --- | --- |
| `fVendas.xlsx` | Fato | 35.762 |
| `dCliente.xlsx` | Dimensão | 1.002 |
| `dProduto.xlsx` | Dimensão | 234 |
| `dVendedor.xlsx` | Dimensão | 24 |
| `dFornecedor.xlsx` | Dimensão | 42 |
| `dNota.xlsx` | Dimensão | 25.400 |
| `dForma.xlsx` | Dimensão | 26 |
| `dCategoria.xlsx` | Dimensão | 9 |
| `dTempo` | Dimensão | — | Calendário (chave `IDTEMPO`) com data, ano, mês, dia e estação do ano |

> **Nota:** `dTempo` foi **reincorporada ao modelo** (22/06/2026) para viabilizar a análise temporal de vendas (série histórica e sazonalidade). A dimensão havia sido removida por orientação inicial do professor, mas sua ausência inviabilizava perguntas de negócio baseadas em tempo. Os valores de `IDTEMPO` em `fVendas` (originalmente com anos 2101–2104, por erro de digitação na origem) foram corrigidos para 2011–2014 e o relacionamento `fVendas[IDTEMPO] → dTempo` foi reativado.

---

## Como executar

1. Clone o repositório
```bash
git clone https://github.com/JISAE221/projeto-ia-vendas.git
```

2. Abra o arquivo `powerbi/ia-vendas.pbip` no Power BI Desktop

3. Verifique os caminhos dos arquivos em `data/` no Power Query

4. Atualize as fontes de dados se necessário e clique em **Refresh**

---

## Git — Leia antes de começar

Se você nunca usou Git, leia essa seção inteira antes de rodar qualquer comando.

### O que é Git?

Git é uma ferramenta que **salva o histórico de tudo que você faz no projeto**. Cada vez que você faz um `commit`, é como tirar um print do estado dos arquivos naquele momento. Se algo der errado, dá pra voltar a qualquer ponto.

O **GitHub** é só onde esse histórico fica armazenado na nuvem, pra todo mundo do grupo acessar.

### O que é uma branch?

Pensa numa branch como uma **cópia de trabalho isolada** do projeto. Você trabalha na sua cópia sem mexer no que os outros estão fazendo. Quando terminar, você propõe juntar sua cópia com o projeto principal através de um **Pull Request**.

```
main               ──────────────────────────────────  (versão final, intocável)
                       \
develop              ──────────────────────────────  (integração do grupo)
                           \                    \
feature/renato           ───                     merge via PR
feature/gabriel                  ───             merge via PR
```

No projeto de vocês:
- **`main`** — versão final entregável. Só o Daniel faz merge aqui.
- **`develop`** — onde o grupo integra o trabalho. Todo PR vai pra cá primeiro.
- **`feature/seu-nome`** — sua branch de trabalho. Cada um tem a sua.

### Por que não commitar direto na main?

Porque se duas pessoas editam o mesmo arquivo ao mesmo tempo na main, o Git não sabe qual versão manter — isso é um **conflito**, e resolver conflito na main é estressante com prazo curto. Cada um na sua branch, conflito fica isolado e fácil de resolver.

### Configuração inicial (faça uma vez na sua máquina)

```bash
# Identificar quem é você nos commits
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"

# Verificar se ficou certo
git config --list
```

---

## Erros comuns e como evitar

**1. Commitar na branch errada**
```bash
# Antes de qualquer coisa, sempre verifique em qual branch você está
git branch
# A branch atual aparece com * na frente
# Se estiver na main ou develop, mude antes de trabalhar:
git checkout feature/sua-branch
```

**2. Esquecer de atualizar a develop antes de criar a branch**
```bash
# ERRADO — criar branch com develop desatualizada
git checkout -b feature/minha-tarefa

# CERTO — sempre atualizar primeiro
git checkout develop
git pull origin develop
git checkout -b feature/minha-tarefa
```

**3. Fazer push sem commitar**
```bash
# Git push não sobe arquivos soltos — precisa commitar antes
git add .
git commit -m "feat: minha alteração"
git push origin feature/minha-tarefa
```

**4. Mensagem de commit vazia ou genérica**
```bash
# ERRADO
git commit -m "alterações"
git commit -m "update"
git commit -m "."

# CERTO — descreve o que foi feito
git commit -m "feat: dashboard de clientes com filtro por região"
```

**5. Deletar arquivos sem querer com `git checkout`**
```bash
# CUIDADO — esse comando descarta suas mudanças não commitadas
git checkout -- arquivo.pbip    # DESFAZ tudo que você fez no arquivo

# Se quiser só trocar de branch sem perder o trabalho, use stash:
git stash                        # guarda as mudanças temporariamente
git checkout develop             # troca de branch com segurança
git stash pop                    # recupera as mudanças
```

**6. Conflito de merge**

Se aparecer isso ao fazer merge:
```
CONFLICT (content): Merge conflict in arquivo.pbip
Automatic merge failed; fix conflicts and then commit the result.
```
Chama o Daniel. Conflito em arquivo `.pbip` (binário) é resolvido escolhendo uma versão — não tenta editar manualmente.

**7. Dados não carregam no Power BI após git clone/pull**

Se, ao abrir o arquivo .pbip, você se deparar com as mensagens:
```
"Um ou mais objetos calculados precisam ser atualizados manualmente"
"Algumas das tabelas estão incompletas ou não têm dados"
```
Abra a pasta `projeto-ia-vendas/powerbi/ia-vendas.SemanticModel/definition` e procure o arquivo `expressions.tmdl`. Nela, você encontrará a expressão:
```
expression BasePath = "C:\projeto-ia-vendas\data\" (...)
```
Substitua o valor entre aspas pelo caminho absoluto da pasta `\data\` na sua máquina (não esqueça a `\` no final). Por fim, abra de novo o arquivo .pbip e clique no botão "atualizar agora".

---

## Branches

| Branch | Propósito |
| --- | --- |
| `main` | Produção — só entra via PR aprovado pelo Tech Lead |
| `develop` | Integração — todos fazem merge aqui primeiro |
| `feature/*` | Uma por membro/tarefa |

### Criando sua branch de feature

```bash
# Sempre partir da develop atualizada
git checkout develop
git pull origin develop

# Criar sua branch
git checkout -b feature/seu-nome-tarefa
# Exemplos:
# git checkout -b feature/renato-etl
# git checkout -b feature/gabriel-dashboard-clientes
# git checkout -b feature/felipe-dashboard-produtos
```

---

## Comandos do dia a dia

### Fluxo completo de trabalho

```bash
# 1. Antes de começar — sempre atualizar a develop
git checkout develop
git pull origin develop

# 2. Criar ou mudar para sua branch
git checkout -b feature/minha-tarefa    # criar nova
git checkout feature/minha-tarefa       # ou mudar para existente

# 3. Trabalhar, salvar os arquivos...

# 4. Ver o que mudou
git status

# 5. Adicionar os arquivos
git add .                                      # tudo
git add powerbi/ia-vendas.pbip                # ou arquivo específico

# 6. Commitar com mensagem descritiva
git commit -m "feat: adiciona dashboard de clientes por região"

# 7. Subir para o GitHub
git push origin feature/minha-tarefa

# 8. Abrir Pull Request no GitHub (interface web)
#    base: develop <- compare: feature/minha-tarefa
```

### Padrão de mensagens de commit

```bash
git commit -m "feat: descrição do que foi adicionado"
git commit -m "fix: descrição do que foi corrigido"
git commit -m "docs: atualiza documentação"
git commit -m "style: ajuste de formatação visual"

# Exemplos reais do projeto:
git commit -m "feat: ETL Power Query com filtro de SCDs"
git commit -m "feat: medidas DAX de receita e margem"
git commit -m "feat: dashboard vendedores com hierarquia gerente"
git commit -m "docs: preenche dicionário de dados"
```

### Mantendo sua branch atualizada com a develop

```bash
# Se alguém fez merge na develop enquanto você trabalhava
git checkout develop
git pull origin develop
git checkout feature/minha-tarefa
git merge develop
# Resolver conflitos se houver, depois:
git push origin feature/minha-tarefa
```

### Comandos úteis

```bash
git log --oneline         # histórico resumido de commits
git diff                  # ver o que mudou antes de commitar
git stash                 # guardar mudanças temporariamente
git stash pop             # recuperar mudanças guardadas
git branch                # listar branches locais
git branch -a             # listar todas (local + remoto)
```

---

## Entrega

Projeto entregue em **23 de junho de 2026**.

---

*Disciplina: Análise Exploratória de Dados — Biopark Edu · Grupo 4*