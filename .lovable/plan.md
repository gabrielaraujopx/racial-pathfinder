## Ajustes em `public/Ferramenta_ER.html`

Três correções no gerador de PPT, sem alterar a UI da ferramenta nem o JSON de teste.

### 1. Linhas independentes para Meta BA e Meta PPI (gráfico Eixo 05)

**Hoje:** as projeções tracejadas só aparecem para metas com `ano > último ano histórico`, e existe uma "ponte" artificial que injeta o último ponto histórico no início da linha de meta (linhas 3034–3048). Isso impede comparar como as metas declaradas se comportavam *durante* a série histórica.

**Mudança:**
- Remover o filtro `m.ano > ultAnoHist`. As linhas Meta BA e Meta PPI passam a usar **todas** as metas declaradas (`metasArr`), inclusive anos sobrepostos ao histórico.
- Eliminar a "ponte" `metaBAByAno[ultAnoHist] = histBAByAno[ultAnoHist]` (e a equivalente de PPI). Cada série Meta usa apenas os anos onde de fato existe um valor declarado.
- O eixo X (`anosUni`) passa a ser a união de todos os anos com qualquer valor (hist BA, hist PPI, meta BA, meta PPI, meta Gap). Isso garante que uma meta declarada para 2019, com histórico iniciando em 2015, apareça como ponto isolado/segmento próprio.
- A anotação "🎯 Gap-alvo" continua exclusiva dos casos sem `valorMetaBA`/`valorMetaPPI`, mas passa a considerar todos os anos (não apenas futuros).

### 2. Eliminar sobreposição dos rótulos de dados no gráfico

**Hoje:** todas as 4 séries usam `showValue:true`, `dataLabelPosition:'t'`, `dataLabelFontSize:7` e `lineDataSymbolSize:7`. Resultado: números empilhados em cima uns dos outros e por baixo dos pontos.

**Mudança (via opções por série — pptxgenjs aceita `options` por entrada do array de dados):**
- Posicionar rótulos de forma intercalada:
  - `BA` → label position `t` (acima)
  - `PPI` → label position `b` (abaixo)
  - `Meta BA` → label position `t`, deslocamento sutil + cor mais clara
  - `Meta PPI` → label position `b`, deslocamento sutil + cor mais clara
- Reduzir `lineDataSymbolSize` de 7 → 4 (pontos menores deixam o número legível ao lado).
- Reduzir `dataLabelFontSize` de 7 → 6 e mudar cor das metas para um cinza/laranja mais suave, deixando os históricos com peso visual maior.
- Quando o nº de anos no eixo X for ≥ 8, exibir rótulos **apenas das séries históricas** (Meta BA/PPI ficam sem `showValue` mas mantêm a linha). Em séries curtas mantém todos.
- Garantir `dataLabelPosition` por série usando o formato `{ name, labels, values, options:{ dataLabelPosition, showValue, dataLabelFontSize, dataLabelColor } }` suportado pelo pptxgenjs.

### 3. Paginação dos itens nos slides de coalizão

**Hoje:** o slide da coalizão renderiza Indicadores/Estratégias/Lacunas em 3 colunas com `c.indicadores.slice(0,10).forEach(...)` mais um corte `if (yI + totalH > colBot) return;` (linhas 2879, 2911, 2936). Resultado: Pré-escola tem 8 indicadores, mas só 4 cabem na coluna e o restante é silenciosamente descartado.

**Mudança:**
- Antes de desenhar cada coluna, fazer um **layout pass**: percorrer todos os itens (sem o `slice(0,10)`) acumulando `totalH` e quebrar em "páginas" sempre que `yCol + totalH > colBot`.
- O número total de páginas da coalizão = `max(páginasIndic, páginasEstrat, páginasLacunas)`.
- Para cada página `p ∈ [0, totalPaginas)`:
  - Criar um slide novo (já existe `pres.addSlide()` no loop — vira loop interno).
  - Header recebe sufixo `· (parte p/total)` quando `totalPaginas > 1`.
  - Repetir as faixas superiores (header marrom, índice de redução de gaps, diagnóstico + visão de sucesso, headers das 3 colunas, avisos de barreira).
  - Cada coluna renderiza apenas o slice de itens correspondente à sua página; se uma coluna tem menos páginas que o total, exibe os itens da última página apenas no último slide (ou fica vazia nos slides extras — preferência: deixar vazia com label "(continua nos demais eixos)" para evitar confusão).
- Manter a numeração dos itens contínua entre páginas (Indicador 5, 6, 7, 8…), não reiniciar do 1.
- Remover o teto rígido `slice(0,10)` e `slice(0,8)` — passa a ser limitado apenas pelo espaço vertical, agora resolvido pela paginação.

### Verificação

Após editar, regenerar o PPT a partir de `public/dados_teste_variabilidade.json` (que já cobre os 6 cenários de gráfico + Pré-escola com 8 indicadores) e validar:
1. Cada item Eixo 05 com meta BA/PPI mostra duas linhas tracejadas próprias, sem ponte forçada.
2. Nenhum rótulo de valor fica sobreposto a outro ou ao ponto da linha em qualquer um dos 6 cenários.
3. O slide da coalizão Pré-escola lista os 8 indicadores (provavelmente em 2 slides "parte 1/2" e "parte 2/2").

### Arquivos afetados

- `public/Ferramenta_ER.html` (única alteração).
- Nenhuma mudança no JSON de teste nem em outros arquivos.
