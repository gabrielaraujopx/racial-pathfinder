## Ajustes na ferramenta

### 1) Campo "Fonte" perde foco / scrolla até o topo a cada letra digitada

**Causa**
Os inputs de fonte chamam `renderCoalizao()` (re-render total da aba) a cada `oninput`:
- `public/Ferramenta_ER.html:1698` — fonte do Diagnóstico (`upDiagFonte` → `renderCoalizao()` na linha 2383)
- `public/Ferramenta_ER.html:2204` — fonte do Indicador (`upItem(...);renderCoalizao()` inline)
- Padrão também presente em outros campos que disparam re-render visual (ex.: badge "⚠ Sem fonte" e borda vermelha).

Cada keystroke destrói o `<input>` focado e recria toda a árvore, então o foco volta ao topo da página.

**Correção**
Mudar a estratégia "atualiza estado + re-render total no oninput" para:
1. **No `oninput`:** apenas mutar o estado (`saveState()` + `refreshMat()`, sem `renderCoalizao()`). Atualizar **somente o que é visual e local ao card** via DOM direto: remover/adicionar a classe da borda vermelha e ocultar/mostrar o badge "⚠ Sem fonte" conforme o input ficar vazio ou preenchido. Trocar o `background` do próprio input (`#FFF5F5` ↔ `#fff`) e a cor da borda.
2. **No `onchange` (blur):** aí sim chamar `renderCoalizao()` uma única vez para garantir consistência total (ex.: cobertura/lacuna do Eixo 2 que dependem da existência de fonte).

Aplicar o mesmo padrão a:
- Fonte do Diagnóstico (`upDiagFonte`) — remover `renderCoalizao()` do final, mover para blur.
- Fonte do Indicador (linha 2204) — remover `;renderCoalizao()` do `oninput`, adicionar `onchange="renderCoalizao()"`.
- Fonte do Eixo 6 série histórica (linha 2051, `upEixo6Ponto(...,'fonte',...)`) — mesma checagem (já não dispara render, mas validar).
- Demais inputs de texto livre que hoje chamem `renderCoalizao()` no `oninput` (varredura final para não deixar regressão).

Para os badges/bordas que dependem do valor: dar `id` estável ao card e ao badge (`item-card-ind-${i}`, `badge-fonte-ind-${i}`) para que o handler atualize só esses nós.

### 2) Eixo 5 (Gaps BA×PPI) — permitir gap-real direto na série histórica

Hoje a coluna **Gap** da série temporal (linhas 2040–2054) é **só leitura**: só é calculada se BA **e** PPI estiverem preenchidos. Se a coalizão só tem o gap ano-a-ano (sem BA/PPI individualizados), o ponto fica inválido e o gráfico ignora.

Replicar o comportamento já usado em **Metas** (`valorMetaGap`):

**Modelo de dados**
- Adicionar campo `valorGap` em cada ponto de `it.serie`.
- `addEixo6Ponto` (linha 2510): incluir `valorGap: null` no objeto criado.
- Normalização de importação JSON (`public/Ferramenta_ER.html:840`): preservar `valorGap` quando presente.

**Handler**
- `upEixo6Ponto` (linha 2518): tratar `campo === 'valorGap'` igual aos demais numéricos.

**UI da linha da série (linha 2046–2053)**
- Substituir o `<td>` somente-leitura por um `<input type="number">` editável, espelhando o padrão de Metas (linha 2089):
  - Se BA e PPI estiverem ambos preenchidos → mostrar valor calculado como **placeholder** (e o campo fica opcional; quando vazio usa o derivado).
  - Se a pessoa digita um valor manual em Gap, esse valor **prevalece** sobre o derivado.
  - Mostrar uma dica curta (igual Metas) indicando "deixe vazio para usar o cálculo BA−PPI".

**Cálculo e gráficos** (`renderEixo6Chart`, linhas 3178–3215 e equivalentes para células B/C)
- Trocar o filtro atual (linha 3179) que exige `valorBA` **e** `valorPPI` por: aceitar pontos válidos se **(BA e PPI)** OU **`valorGap` numérico**.
- Ao montar `gapReal` (linha 3191): se `p.valorGap != null && Number.isFinite(p.valorGap)`, usar ele; caso contrário, calcular via BA/PPI.
- Aplicar a mesma lógica nas demais células do Eixo 5 que dependem da série (B/C/etc.) onde o cálculo é feito a partir de BA/PPI — varrer e padronizar.

**Validação de "recorte racial trabalhado"** (`_indicadorTemRecorteRacial`, linha 1034)
- Hoje só aceita ponto com BA **e** PPI numéricos.
- Atualizar para considerar trabalhado também quando o ponto tem `valorGap` numérico (consistente com a regra "se só temos o gap, ainda é recorte racial sendo monitorado").

**Texto explicativo**
- Atualizar o `<div>` de instrução do bloco (algo como "Preencha BA e PPI para cálculo automático, **ou apenas o Gap** se só houver o gap ano-a-ano") — espelhando o texto já existente em Metas (linha 2065).

### Fora do escopo
- Nenhuma mudança em layout, fontes, cores, exportação PPT (heights/pagination), cálculo de notas, cobertura racial, ou no estruturação dos eixos. Apenas comportamento de input + 1 campo novo opcional.

### Arquivos afetados
- `public/Ferramenta_ER.html` (único arquivo).
