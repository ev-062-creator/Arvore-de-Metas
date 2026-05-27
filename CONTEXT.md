# Árvore de Metas — CNJ · Contexto do Projeto

## O que é

Organograma interativo de desdobramento de metas para a rede CNJ de varejo.
Aplicação standalone em HTML/CSS/JS puro — abre direto no navegador, sem dependências externas.

---

## Hierarquia de desdobramento

```
Meta Empresa (R$)
└── Regional (7 regionais)
    └── Cluster de lojas (C1–C5 por regional)
        └── Loja (107 lojas no total)
            └── Categoria (5 categorias)
                └── Espécie (produtos dentro da categoria)
```

---

## Regionais

| ID   | Nome           | Lojas |
|------|----------------|-------|
| GO   | Regional GO    | 20    |
| DF   | Regional DF    | 20    |
| I1   | Interior 1     | 19    |
| I2   | Interior 2+TO  | 20    |
| MA   | Maranhão       | 11    |
| PA   | Regional PA    | 15    |
| PI   | Piauí          | 2     |

---

## Clusters de lojas

| ID | Range       | Critério de faturamento         |
|----|-------------|--------------------------------|
| C1 | Top 1–10    | acima de R$ 1,2M               |
| C2 | Top 11–30   | R$ 700k – R$ 1,2M              |
| C3 | Top 31–51   | R$ 500k – R$ 700k              |
| C4 | Top 52–84   | R$ 300k – R$ 500k              |
| C5 | Top 85–107  | abaixo de R$ 300k              |

**Importante:** o peso de cada cluster dentro da regional é calculado com base no
faturamento real das lojas daquela regional (não é um % fixo global).
A tabela `W_CL` no código armazena esses pesos por regional × cluster.

---

## Categorias e participação

| ID | Nome      | % participação | Ticket médio padrão |
|----|-----------|---------------|---------------------|
| BR | Branca    | 54,6%         | R$ 1.800            |
| MV | Móveis    | 23,1%         | R$ 1.400            |
| MR | Marrom    | 13,8%         | R$ 1.200            |
| PT | Portáteis | 8,5%          | R$ 180              |
| TL | Telefonia | 0% (a definir)| R$ 900              |

---

## Espécies por categoria

**Branca:** Lavadora, Fogão, Geladeira, Freezer, Micro-ondas, Ar cond., Outros
**Móveis:** Guarda-roupa, Cama/Box, Sofá, Mesa, Colchão, Estante
**Marrom:** TV, Home theater, DVD
**Portáteis:** Fritadeira, Liquidificador, Cafeteira, Batedeira, Ventilador, Outros
**Telefonia:** Smartphone, Tablet, Acessório

---

## Lógica de distribuição da meta

1. **Meta empresa** → digitada pelo usuário (valor do período, não anual)
2. **Meta regional** = meta empresa × `pR[regId]` (% configurável)
3. **Meta cluster** = meta regional × `W_CL[regId_clId]` (peso por faturamento real)
4. **Meta loja** = meta cluster × `W_STORE[regId_clId_rank]` (peso por faturamento real)
5. **Meta categoria** = meta loja × `pK[catId]` (% configurável, padrão tabela acima)
6. **Meta espécie** = meta categoria × `pS[catId_spId]` (% configurável)

### Distribuição por período (linear)

- O usuário define o **período da meta** (data de/até) no topbar
- O usuário define o **intervalo de visualização** nos controles (Total / Mês atual / Semana / Personalizado)
- O sistema calcula: `meta_visualizada = meta × (dias_overlap / dias_total_meta)`
- Se o intervalo de visualização não tiver sobreposição com a meta → banner de aviso

### Quantidade de vendas

- `qty_especie = meta_especie / ticket_medio_categoria`
- `qty_categoria = Σ qty_especie` (soma das espécies, consistente)
- `qty_loja = Σ qty_categoria` (soma das categorias, consistente)
- `qty_cluster`, `qty_regional`, `qty_empresa` seguem o mesmo padrão de soma

---

## Funcionalidades implementadas

- [x] Organograma expansível por clique (nó abre/fecha)
- [x] Meta digitável no topbar com atualização em tempo real
- [x] Período da meta (data de/até) no topbar
- [x] Visualização personalizada (Total / Mês / Semana / Custom) com datas livres
- [x] Distribuição linear proporcional ao período
- [x] Aviso quando visualização não tem sobreposição com a meta
- [x] Pesos por faturamento real por regional × cluster (tabela `W_CL`)
- [x] Pesos por faturamento real por loja dentro do cluster (tabela `W_STORE`)
- [x] Quantidade de vendas consistente em todos os níveis
- [x] Painel lateral de configuração de percentuais (⚙ Config)
  - Ticket médio por categoria
  - % por regional
  - % por cluster (por regional)
  - % por categoria
  - % por espécie (por categoria)
- [x] Filtro de regional (mostra só a regional selecionada)
- [x] Zoom (+/−/1:1)
- [x] Expandir tudo / Recolher tudo
- [x] Tooltip ao hover em qualquer nó
- [x] Export CSV com meta e qtd de vendas por linha
- [x] Legenda de cores dos níveis

---

## Problemas conhecidos / pendentes

- [ ] Painel ⚙ mostra % de clusters por regional mas a UI pode ser melhorada
- [ ] Telefonia tem 0% — aguarda definição do usuário
- [ ] Conectores SVG do organograma são redesenhados via `requestAnimationFrame`; em telas muito grandes pode haver delay visual
- [ ] Não há persistência de configuração (fechar o browser perde os % customizados)
- [ ] Não há autenticação/multi-usuário — arquivo único rodado localmente

---

## Estrutura do arquivo `organograma_metas.html`

```
HTML
├── <style>          CSS completo (variáveis, topbar, controles, nodes, painel)
└── <script>
    ├── DATA         REGS, CLS, STORES (107 lojas), CATS, espécies
    ├── W_CL         Pesos cluster por regional (faturamento real)
    ├── W_STORE      Pesos loja por cluster × regional (faturamento real)
    ├── STATE        meta, metaFrom, metaTo, viewFrom, viewTo, viewMode, OPN{}
    ├── DATE HELPERS fmt8601, fmtBR, diffDays, metaDays, effectiveDays, metaVis
    ├── CONTROLS     onMeta, onMetaDates, onViewDates, setView, doZoom, updateStats
    ├── TOOLTIP      showTT, moveTT, hideTT
    ├── NODE BUILDER mkNode, mkToggle
    ├── TREE ASSEMBLY stem, attach, col
    ├── RENDER       render() — monta todo o DOM do organograma
    ├── SIDE PANEL   toggleSide, buildSide, addSec
    ├── EXPORT CSV   exportCSV()
    └── INIT         initPC(), inicializa datas, chama render()
```

---

## Decisões de design

- **Sem frameworks** — HTML/CSS/JS puro para máxima portabilidade
- **Sem backend** — arquivo único que abre no navegador
- **Cores por nível:** Empresa (roxo), Regional (verde), Cluster (âmbar), Loja (azul médio), Categoria (marrom), Espécie (azul escuro)
- **Conectores SVG** com `pointer-events:none` para não bloquear cliques
- **Meta = valor do período** (não anual) — o usuário decide o que digitar
- **Pesos por faturamento** em vez de divisão igualitária — para refletir a realidade da rede

---

## Repositório

`https://github.com/ev-062-creator/Arvore-de-Metas`

Branch principal: `main`
Arquivo principal: `organograma_metas.html`
