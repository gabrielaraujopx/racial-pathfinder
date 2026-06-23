## Ajustes finais: hierarquia vs-meta na síntese + renomeações de rótulos

Soma-se ao plano já aprovado (JSON com metas, unidades variadas, inversão vs-meta como principal). Tudo aqui é apresentação — sem mexer em cálculo.

---

### 1. Síntese comparativa (UI ferramenta + slide-síntese PPT)

Aplicar a mesma inversão de destaque já planejada no card do Índice no bloco de síntese comparativa:

**UI — síntese comparativa (`Ferramenta_ER.html`, bloco que lista coalizões lado a lado)**
- Coluna/linha do Índice de Redução de Gaps passa a exibir:
  - **Principal (fonte grande, chip colorido):** `vs Meta: <nivelMeta> · <pctMeta>%`
  - **Secundário (fonte menor, cinza/muted, abaixo):** `Geral: <nivel> · <pct>%`
- Quando `nVsMetaValido == 0` na coalizão: principal mostra `Sem metas declaradas` (cinza) e secundário mantém o geral.

**PPT — slide-síntese**
- Faixa do Índice em 2 linhas com hierarquia invertida:
  - L1 (grande, cor de destaque): `Índice de Redução de Gaps — vs Meta: <nivelMeta> · <pctMeta>%`
  - L2 (menor, cinza): `Geral: <nivel> · <pct>%   ·   Cobertura Eixo 02→05: N/M`
- Por-coalizão na tabela de síntese: célula do índice mostra `nivelMeta · pctMeta` em destaque e `geral` em fonte pequena abaixo (mesmo padrão da UI).

---

### 2. Renomeações de rótulos no PPT

**Slide-síntese (PPT)**
- Cabeçalho/coluna hoje rotulada como **"Impacto BA×PPI"** → renomear para **"Índice de Redução de Gaps"**.
- Demais colunas/eixos: remover a palavra **"Eixo"** dos rótulos curtos, mantendo só o nome:
  - `Eixo 01 — Posicionamento` → `Posicionamento`
  - `Eixo 02 — Indicadores` → `Indicadores`
  - `Eixo 03 — Estratégias` → `Estratégias`
  - `Eixo 04 — Articulação` → `Articulação`
  - `Eixo 05 — …` (na síntese) → exibido como `Índice de Redução de Gaps` (não duplica)
- Aplicar a mesma remoção da palavra "Eixo" em rótulos compactos da síntese comparativa da **UI**, mantendo "Eixo" apenas nos títulos longos dos slides individuais (ex.: título do slide do Eixo 02 continua "Eixo 02 — Indicadores").

**"Lacuna" → "Lacunas Identificadas"**
- Toda ocorrência do rótulo `Lacuna` / `Lacunas` (chips, cabeçalhos, legendas no PPT e na UI) → **`Lacunas Identificadas`**.
- Inclui slide-síntese, slide do Eixo 02, dashboard, card de coalizão, exports.
- Singulares ("uma Lacuna") viram "uma Lacuna Identificada" para coerência; rótulos plurais usam "Lacunas Identificadas".

---

### Arquivos tocados

- `public/Ferramenta_ER.html`
  - Bloco de síntese comparativa (UI): hierarquia vs-meta + remoção de "Eixo" + relabel Lacuna.
  - PPT slide-síntese (~2360–2430): texto da faixa do índice, cabeçalhos de coluna, "Impacto BA×PPI" → "Índice de Redução de Gaps", relabel Lacuna, remoção de "Eixo".
  - Demais usos de "Lacuna" no PPT/UI (busca global e substituição contextual).
- `/mnt/documents/Logica_Sistematizacao_Executivo.md` e `Logica_Sistematizacao_Detalhado.md`: usar "Lacunas Identificadas" e mencionar a inversão de hierarquia na síntese.

### Fora de escopo
- Cálculo do índice, faixas vs-meta, metas (mantidos como já aprovado).
- Títulos dos slides individuais por eixo (mantêm "Eixo 0X — Nome").
