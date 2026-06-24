## Diagnóstico do problema atual

Em `public/Ferramenta_ER.html` (linhas 2760–2786) a paginação calcula um **gap distribuído**: divide o espaço livre entre os itens, limitado a `[gapMin=0.10", gapIdeal=0.18"]`. Com poucos itens curtos, o gap fica fixado em **0.18" (~13pt)**, gerando "buraco" sob itens pequenos. Além disso, `estH` adiciona margem fixa de **0.04"** a cada caixa.

## Plano de intervenção

### 1. Espaçamento proporcional ao conteúdo (Eixos 02, 03, 04)

- **Gap fixo entre itens:** `gapBase = 0.08"` (~5.8pt) entre todos os itens da página. Não cresce quando sobra espaço.
- **Caixa abraça o texto:** reduzir margem extra de `estH` de `0.04"` → `0.02"`. Caixa proporcional ao número real de linhas.
- **Empacotamento máximo (greedy):** o algoritmo continua adicionando itens enquanto `Σ(hItem) + (n−1)·gapBase ≤ avail`. Só quebra para próxima página quando o próximo item de fato não cabe nem com o gap mínimo. Ou seja, **se sobrar espaço no rodapé suficiente para o próximo item, ele entra ali** — paginação só ocorre quando matematicamente impossível encaixar.
- **Sobra sem item:** se o último item de uma coluna couber mas ainda restar espaço, esse espaço fica como margem inferior (sem inflar gaps).
- Demais elementos (bolinha numerada, Q/RSN²/RN, badges, triângulo de criticidade, encaminhamento) seguem ancorados em `yI`/`yE`/`yL`, acompanhando cada item.

### 2. Formato da fonte (Indicadores e Diagnóstico)

Trocar `"(Fonte: nome)"` por `"(nome)"` em fonte menor via rich text do pptxgenjs.

- **Indicadores (coalizão), linhas 2790 e 2992:** substituir string única por array `[{text: ind.texto, options:{fontSize:11}}, {text: ' ('+fonteTxt+')', options:{fontSize:8, italic:true, color:C.cinzaM}}]`.
- **Diagnóstico (faixa superior), linhas 2872–2880:** mesma ideia. Cada item vira dois runs (texto em fontSize 7, fonte em 6 itálico `C.cinzaM`), separados por `·`.
- `estH` continua recebendo a string completa para evitar subestimar a altura.

### Arquivos afetados

- `public/Ferramenta_ER.html`.

### Verificação

1. Coalizão com itens curtos → próximos itens grudados (gap ~0.08"), sobra concentrada no rodapé.
2. Coalizão com itens longos (2–3 linhas) → sem sobreposição, gap consistente.
3. **Aproveitamento de rodapé:** se ainda couber 1 item no fim da coluna, ele entra na página atual em vez de ir para a seguinte.
4. Pré-escola (8 indicadores) → paginação só quando inevitável; nenhum item perdido.
5. Indicadores exibem `texto (nome da fonte)` com a fonte em corpo menor itálico.
6. Diagnóstico exibe `texto (nome) · texto (nome) · …` com fonte em corpo menor itálico.
