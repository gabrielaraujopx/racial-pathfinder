# Plano — Refinar análise temporal do Eixo 06

## Objetivo
Tornar a análise de evolução do gap BA×PPI mais fiel à série histórica, evitando duas distorções da lógica atual (que só compara primeiro vs último ano):
1. Ignora oscilações intermediárias e a velocidade real do fechamento.
2. Gera percentuais inflados quando o gap inicial é muito pequeno.

## O que muda no cálculo (`calcItemEixo6`)

### 1. Dupla métrica de tendência
Para cada item com ≥ pontos mínimos, calcular e expor **as duas métricas lado a lado**:

- **Δ acumulado (%)** — mantém o cálculo atual `(gap_final − gap_inicial) / |gap_inicial| × 100`. É a síntese "de ponta a ponta".
- **Inclinação anual** — regressão linear simples (mínimos quadrados) sobre todos os pontos `(ano, gap)`. Resultado em "pontos de gap por ano" + R² (0–1) como medida de aderência da reta.

O **status** (Fechando / Estagnado / Abrindo) passa a usar a **inclinação anual** como critério primário (mais fiel à tendência real), mantendo o Δ acumulado como informação complementar exibida em todos os relatórios.

Limiares do status (configuráveis por coalizão, herdam defaults):
- Inclinação ≤ −5% do gap inicial por ano → Fechando
- Entre −5% e +5% → Estagnado
- ≥ +5% → Abrindo

### 2. Tratamento de gap inicial pequeno
Novo parâmetro por coalizão: `limiarGapAbsoluto` (default: 1,0 ponto).

- Se `|gap_inicial| < limiarGapAbsoluto`: reportar **Δ absoluto** (em pontos do indicador) em vez de Δ%. O status é classificado pela mesma regra, mas sobre o delta absoluto comparado ao limiar.
- A UI e relatórios mostram a unidade correta ("−0,3 pts" vs "−12%") com tooltip explicando por que mudou de unidade.

### 3. Flag "Volátil"
Calcular após a regressão:
- Contar **reversões de direção** ano a ano (sinal do Δ anual muda).
- Se `R² < 0,5` **ou** ≥ 2 reversões em séries de 4+ pontos → marcar item como `volatil: true`.

A flag **não altera** o status nem a pontuação — apenas adiciona um aviso visual ("⚠ Volátil") na UI, PPT e DOCX, com tooltip "tendência não monotônica; ver série completa".

## O que muda na UI (formulário Eixo 06)
- Cabeçalho de cada item: chips com **Inclinação/ano**, **Δ acumulado**, **R²** e flag **Volátil** quando aplicável.
- Campo configurável `limiarGapAbsoluto` no bloco de parâmetros do Eixo 06 (junto com `pontosMinimos`).
- Mini-sparkline opcional ao lado da tabela da série, mostrando a reta de regressão sobre os pontos.

## O que muda nos relatórios

### PPT (slides dedicados Eixo 06)
Tabela de itens ganha colunas: **Inclinação/ano**, **Δ acumulado**, **Status**, **Flag**. Coluna "Δ gap" atual é substituída pelas duas métricas.

### DOCX (seção Eixo 06)
- Texto descritivo cita ambas as métricas.
- Lista de itens "Volátil" aparece em destaque no resumo da coalizão.

### Documento de lógica (`Sistematizacao_ER_Logica_v4.docx`)
Nova seção 6.x explicando:
- Fórmula da regressão linear (com exemplo numérico do IDEB pré-escolar: pontos 2019–2025 → inclinação ≈ −1,35 pts/ano, R² ≈ 0,99, Δ acumulado −73%).
- Quando o sistema troca % por pontos absolutos.
- Como interpretar a flag Volátil.
- Exemplo contrastante: gap pequeno (0,4 → 0,1) → mostra "−0,3 pts" em vez de "−75%".

## Dados de teste
Atualizar `ER_dados_teste_variabilidade_v6.json` → `v7`:
- 1 coalizão com série monotônica clara (R² alto, sem flag).
- 1 coalizão com série volátil (oscilações, dispara flag).
- 1 indicador com gap inicial < 1 ponto (dispara modo absoluto).

## Detalhes técnicos

**Arquivo:** `public/Ferramenta_ER.html` (monolítico, vanilla JS).

**Funções afetadas:**
- `calcItemEixo6` — adicionar regressão, dual-metric, detecção de volatilidade e modo absoluto.
- `calcEixo6` — agregação mantém a mesma estrutura (média de status × cobertura), apenas usa o novo status baseado na inclinação.
- `defaultEixo6` / `normalizeEixo6` — incluir `limiarGapAbsoluto` (default 1.0).
- `refreshImpacto` e renderização do formulário Eixo 06 — exibir novos chips.
- Geração de PPT (`dimsSint`, slides dedicados) e DOCX — novas colunas/campos.

**Cálculo da regressão (mínimos quadrados):**
```text
n  = nº de pontos válidos
x  = anos, y = gaps
m  = (n·Σxy − Σx·Σy) / (n·Σx² − (Σx)²)   ← inclinação (pts/ano)
b  = (Σy − m·Σx) / n
ŷᵢ = m·xᵢ + b
R² = 1 − Σ(yᵢ − ŷᵢ)² / Σ(yᵢ − ȳ)²
```

**Compatibilidade:** dados v6 existentes continuam carregando; campos novos preenchem com defaults na normalização.

## Fora de escopo
- Mudar a fórmula de agregação do índice de impacto (mantém média × cobertura).
- Alterar Eixos 1–5 ou Eixo 7.
- Gráficos interativos novos além da sparkline opcional.
