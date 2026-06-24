## Ajustes em `public/Ferramenta_ER.html`

Dois ajustes no gerador de PPT. Sem alterar lógica de cálculo nem o JSON.

---

### 1. Paginação inteligente da coalizão (Eixos 02, 03, 04)

**Problema atual:** o layout pass criado no ajuste anterior paginava cedo demais — Alfabetização sobrava espaço na coluna de Indicadores mas mesmo assim quebrava após o 3º indicador.

**Resposta à dúvida do espaçamento:** sim, compactar com valor fixo pode gerar sobreposição quando o texto do indicador é longo (2-3 linhas). A solução é **espaçamento adaptativo por item**, calculado a partir da altura real do bloco de texto, e não um valor fixo.

**Como vai funcionar:**

1. **Medição real de cada item** (por coluna, antes de desenhar):
   - Para cada indicador/estratégia/lacuna, calcular `hTexto` = altura real ocupada pelo texto principal (já existe via `estH`, baseada em chars/linha e fontSize).
   - Considerar também a altura do bloco lateral (bolinha numerada + Q + RSN + RN no caso de indicadores) — o item ocupa `max(hTexto, hLateral)`.

2. **Gap adaptativo entre itens:**
   - Definir `gapMin = 0.08"` (~5.8pt) e `gapIdeal = 0.18"` (~13pt).
   - Calcular `espacoLivre = colBot - colTop - Σ(hItem)`.
   - `gapReal = clamp(espacoLivre / (n-1), gapMin, gapIdeal)`.
   - Se `gapReal >= gapMin` para todos os itens da coluna → cabem todos sem paginar.
   - Se `gapReal < gapMin` → começar a paginar (move últimos itens para próxima página, recalcula gap da página atual, repete).

3. **Garantia anti-sobreposição:**
   - O cálculo parte de `hTexto` real (não de um valor fixo), então um indicador com texto longo "empurra" naturalmente o próximo, e o gap é só o espaço *adicional* entre eles. Não há colapso de texto sobre texto mesmo no `gapMin`.
   - O bloco lateral (bolinha + Q/RSN/RN) é desenhado com a mesma `y` inicial e altura do bloco do texto principal, garantindo que **acompanha o item em todas as páginas**.

4. **Paginação independente por coluna:**
   - Indicadores, Estratégias e Lacunas calculam suas próprias páginas. `totalPaginasCoalizao = max(pInd, pEstrat, pLac)`.
   - Numeração contínua entre páginas (Indicador 5, 6, 7...).
   - Colunas que terminaram antes ficam vazias nas páginas extras (sem rótulo extra, para não poluir).
   - Header do slide recebe sufixo `· (parte p/total)` apenas quando `totalPaginas > 1`.

**Resultado esperado em Alfabetização:** se 4 indicadores cabem com `gap ≥ gapMin`, o slide única página os 4. Se ainda sobrar espaço, o gap se expande até `gapIdeal`. Só paginar quando matematicamente não couber.

---

### 2. Modo de visualização dos gráficos (Eixo 05)

**Nova UI na ferramenta** — toggle global no topo da seção de gráficos do Eixo 05, com duas opções:

- **(A) Gap unificado** *(padrão)*: 1 gráfico por indicador, com linhas de Gap Real (série histórica) + Gap-Alvo (todos os anos preenchidos pela coalizão) no mesmo eixo. Até **4 gráficos por slide**. Pagina quando há mais de 4 indicadores.
- **(B) BA × PPI separado**: 2 gráficos por indicador (Real à esquerda: BA hist + PPI hist; Projetado à direita: Meta BA + Meta PPI). Até **2 indicadores (= 4 gráficos) por slide**, lado a lado. Pagina quando há mais de 2 indicadores.

**Detalhes de implementação:**

- **Toggle UI**: select/radio "Modo de visualização" no header da aba Eixo 05, persistido em `localStorage` (`er_chart_mode`) para não perder ao recarregar.
- **Valores de meta visíveis**: nos dois modos, todos os pontos de meta exibem o valor (resolve queixa anterior de que metas ficavam sem rótulo).
- **Anti-sobreposição de rótulos**: manter o intercalado top/bottom já implementado, mas agora aplicado também ao modo Gap unificado (Gap Real `t`, Gap-Alvo `b`).
- **Modo B**:
  - Gráfico Real usa apenas pares (ano, BA) e (ano, PPI) históricos.
  - Gráfico Projetado usa apenas Meta BA e Meta PPI declarados, com seu eixo X independente (todos os anos com meta declarada, inclusive sobrepostos ao histórico).
  - Título de cada gráfico fica acima: `Indicador X — Real` / `Indicador X — Projetado`.
  - Largura de cada gráfico ≈ metade da área útil; altura idêntica ao modo A.
- **Modo A**:
  - Linha sólida (Gap Real) sobre os anos com histórico.
  - Linha tracejada (Gap-Alvo) sobre todos os anos com meta declarada (sem ponte artificial).
  - Eixo X = união de todos os anos com qualquer valor.

**Paginação dos slides de gráfico:** já existe lógica de slide-por-bloco; ajustar para considerar `itensPorSlide = modo === 'B' ? 2 : 4`.

---

### Arquivos afetados

- `public/Ferramenta_ER.html` (única alteração).
- Sem mudanças no JSON de teste nem em outros arquivos.

### Verificação

Após editar, regenerar PPT a partir de `public/dados_teste_variabilidade.json`:
1. **Alfabetização (coalizão)**: confirmar que indicadores 4/5/6 entram na parte 1 se houver espaço; só pagina quando texto + gap mínimo extrapolam.
2. **Pré-escola (8 indicadores)**: deve continuar paginando em "parte 1/2" e "parte 2/2" sem perder itens.
3. **Eixo 05 modo A**: 4 gráficos por slide, Gap Real + Gap-Alvo juntos, rótulos de meta visíveis.
4. **Eixo 05 modo B**: 2 indicadores por slide × 2 gráficos cada, Real à esquerda e Projetado à direita.
5. Nenhum texto de indicador sobreposto a outro em nenhuma coalizão.
