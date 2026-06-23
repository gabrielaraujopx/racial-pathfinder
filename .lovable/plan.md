## Eixo 05 — Visão dupla (geral × meta), renomeações, unidade por indicador e reordenação do PPT

Todas as mudanças em `public/Ferramenta_ER.html` + atualização dos 2 docs em `/mnt/documents/` + `public/dados_teste_variabilidade.json`.

---

### Parte A — Renomeação global do índice e dos status

- **"Índice de Impacto em Equidade" → "Índice de Redução de Gaps"** em UI, PPT (faixa do slide-síntese, título do slide do Eixo 05, rodapés), JSON (`tituloEixo6` se houver) e nos 2 documentos.
- **Renomeação dos status dos itens** (em tudo: UI, PPT, JSON interno, docs, migração de coalizões legadas):
  - `Fechando` → `Avançando` (▼ verde mantém)
  - `Abrindo` → `Retrocesso` (▲ vermelho mantém)
  - `Estagnado` permanece
- Migração: `normalizeCoalizao` reescreve `status` antigos nos JSONs carregados; helpers `corStatus`/`iconeStatus` e `SCORES` passam a usar os novos nomes; chips do card e legendas do PPT refletem.
- Travas qualitativas atuais (`nFechando==0` → teto Avanço incipiente, etc.) passam a olhar para `nAvancando==0`.

---

### Parte B — Unidade própria por indicador

Hoje a unidade já existe no item (`it.unidade`, default `%`), mas vários pontos do PPT/UI ainda forçam `%`:

- **Slide gráficos BA×PPI**: eixo Y rotulado com `it.unidade` (Ponto SAEB, IDEB, %, valores absolutos…), sem `%` fixo.
- **Tabela do item no slide do Eixo 05**: colunas BA/PPI/Gap usam `it.unidade` (já parcialmente faz; corrigir Gap e cabeçalho do slide).
- **Card UI do Índice**: linha "Gap médio antes→depois" passa a omitir unidade quando coalizão mistura unidades, e por-item respeita `it.unidade`.
- **Validação**: `it.unidade` vira campo obrigatório (texto livre, default `%`). Vazio bloqueia salvar o item.

---

### Parte C — Visão dupla: geral × vs-meta

#### C1. Estrutura de meta no item
Cada item do Eixo 05 ganha:
```
metas: [{ ano: number, valorMetaGap: number }, ...]   // ordenada por ano
```
- Coalizão lista ano-a-ano até o horizonte que quiser (ex.: 2025–2035).
- `valorMetaGap` é o gap-alvo naquele ano, **na mesma unidade do indicador**, com sinal coerente com `sentidoDesejado` (gap esperado entre BA e PPI).
- UI: tabela editável dentro do item (linhas {ano, valor}, botão "+ ano"), abaixo da série BA×PPI.

#### C2. Cálculo do status vs-meta por item
Para o último ano `Y` da série temporal do item que tenha meta declarada (`Y_meta = max ano em metas com ano ≤ último ano da série`):
```
gapReal(Y_meta)     = média(gap dos pontos do ano Y_meta)   // se múltiplos
gapMeta(Y_meta)     = valorMetaGap em Y_meta
gapMarco            = gap no ano do marco (mesma janela do cálculo geral)
delta_real          = (gapReal − gapMarco) normalizado pelo sentidoDesejado
delta_esperado      = (gapMeta − gapMarco) normalizado pelo sentidoDesejado
                      // ambos: negativo = reduziu (bom)
tol                 = max(0.10·|delta_esperado|, 0.05·|gapMarco|)
```
Status vs-meta:
- `delta_real > +tol` → **"Em retrocesso"** (gap piorou)
- `|delta_real| ≤ tol_zero` (tol_zero = 0.05·|gapMarco|) → **"Não há redução"**
- `delta_real ≤ delta_esperado − tol` (reduziu mais que o esperado) → **"Redução acima da meta"**
- `|delta_real − delta_esperado| ≤ tol` → **"Redução dentro da meta"**
- `delta_real < 0` mas insuficiente → **"Redução abaixo da meta"**

Casos especiais:
- `metas.length == 0` → **"Sem meta declarada"** (cinza); não entra no agregado vs-meta.
- `item.barreira.ativa` → **"Barreira de dados"** (já existente); idem.
- Sem ponto no ano da última meta acessível → cai para a meta cujo ano é o mais próximo ≤ último ano com dado; se nenhum, "Sem meta declarada".

#### C3. Nível agregado vs-meta (paralelo ao geral)
Sobre os itens com vs-meta válido (exclui "Sem meta declarada" e "Barreira de dados"):
```
SCORES_META = { 'Redução acima da meta':100, 'Redução dentro da meta':80,
                'Redução abaixo da meta':40, 'Não há redução':10, 'Em retrocesso':0 }
mediaMeta   = média dos scores
pctMeta     = round(mediaMeta · fatorCobertura · fatorMedicaoMeta)
              // fatorCobertura = mesmo da visão geral
              // fatorMedicaoMeta = nVsMetaValido / max(1, nListados - nBarreira)
```
Níveis vs-meta (faixas paralelas, nomes próprios):
- ≥80 **"Superando metas"**
- ≥60 **"No ritmo da meta"**
- ≥30 **"Aquém da meta"**
- ≥10 **"Sem redução vs meta"**
- <10 ou nenhum item com meta válida e ≥1 retrocesso → **"Em retrocesso vs meta"**
- `nVsMetaValido == 0` → **"Sem metas declaradas"** (cinza, não compara).

Travas qualitativas mantidas: se `nRetrocesso > nAcima+nDentro` → teto "Aquém da meta".

#### C4. Onde a visão dupla aparece
**Card UI (`Índice de Redução de Gaps`)**: passa a ter 2 colunas lado a lado:
- Esquerda — **Visão geral**: nível·pct, gap médio antes→depois, contagens Avançando/Estagnado/Retrocesso/Insuf.
- Direita — **Visão vs meta**: nível·pct, contagens Acima/Dentro/Abaixo/Sem redução/Retrocesso/Sem meta.
- Aviso de barreira em massa permanece (rodapé do card).

**Por-item (UI e PPT)**: cada item exibe 2 selos lado a lado — selo geral (▼/■/▲) + selo vs-meta colorido (verde/azul/amarelo/laranja/vermelho/cinza).

**Slide-síntese (PPT)**: faixa do índice em 2 linhas:
- L1: `Índice de Redução de Gaps — Geral: <nivel> · <pct>%`
- L2: `vs Meta: <nivelMeta> · <pctMeta>%   ·   Cobertura Eixo 02→05: N/M`

**Slide do Eixo 05 (PPT)**: rodapé em 3 blocos:
- Bloco 1 — Geral (nivel·pct + Avan/Est/Retr/Insuf + gap antes→depois)
- Bloco 2 — vs Meta (nivelMeta·pctMeta + Acima/Dentro/Abaixo/SemRed/Retr/SemMeta)
- Bloco 3 — Cobertura + aviso barreira se ≥50%

Cada card de item no slide do Eixo 05 ganha 2ª linha de status: `Geral: Avançando · vs Meta: Dentro da meta`.

---

### Parte D — PPT: nova ordem e gráficos com rótulos

#### D1. Ordem nova
```
1. Capa
2. Slide-síntese  (subiu para logo após a capa)
3. Slides dos Eixos 01–04
4. Slide do Eixo 05 (Índice de Redução de Gaps)
5. Eixo 05 — Gráficos BA × PPI   ← antes dos Gaps
6. Eixo 05 — Gaps BA × PPI
7. Demais slides finais (se houver)
```
Reordenação simples no loop de geração do PPT em `Ferramenta_ER.html` (~2540–2800).

#### D2. Rótulos nos pontos dos gráficos
Em `pres.addChart(pres.ChartType.line, …)` dos gráficos BA×PPI:
- `showValue: true`, `dataLabelPosition: 't'` (topo do ponto), `dataLabelFontSize: 8`, `dataLabelFormatCode` derivado da unidade:
  - `%` → `'0.0"%"'`
  - inteiro (SAEB, valores, IDEB sem decimais) → `'#,##0'`
  - default decimal → `'#,##0.0'`
- Linha do marco vertical mantém rótulo "Início coalizão".
- Eixo Y: rótulo = `it.unidade`.

---

### Parte E — JSON de teste atualizado

Substituir `public/dados_teste_variabilidade.json` (+ cópia em `/mnt/documents/`) para cobrir o novo modelo:

- **C1** (Anos Iniciais): metas declaradas 2024→2035 com valorMetaGap em pontos SAEB; itens hoje "dentro" e "abaixo" da meta.
- **C2** (Anos Finais): meta agressiva → status vs-meta = "Redução abaixo da meta"; geral = Avançando (mostra o split que motivou a mudança).
- **C3** (Tecnologia): unidade `%`, sem meta declarada em 2 itens → testa "Sem meta declarada" no agregado.
- **C4** (Alfabetização): metas batidas → "Redução acima da meta"; visão geral Avançando + vs-meta "Superando metas".
- **C5**: coalizão com gap em retrocesso real → testa "Em retrocesso" geral e vs-meta.
- **C6** (massa de barreiras): 5/8 itens com barreira; aviso laranja persiste.
- **C7**: coalizão com `incluirGraficos=true`, unidades mistas (SAEB, IDEB, %, valor absoluto) para validar rótulos dos pontos.
- Renomeia todos `status:"Fechando"`/`"Abrindo"` herdados para os novos rótulos.

---

### Parte F — Documentação (atualizar os 2 arquivos)

`/mnt/documents/Logica_Sistematizacao_Executivo.md` e `Logica_Sistematizacao_Detalhado.md`:
- Renomeação do índice e dos status (com tabela de-para).
- Nova seção **"Visão geral × Visão vs meta"** com:
  - O que cada uma responde (gestor).
  - Tabela das 5 faixas vs-meta com cores e significado.
  - Exemplo narrado: coalizão com meta 2024→2035 atingindo 60% do esperado em 2026 → "Redução abaixo da meta".
- Detalhado ganha as fórmulas de `delta_real`, `delta_esperado`, `tol`, `pctMeta` e 2 exemplos numéricos.
- Seção PPT atualizada com nova ordem dos slides e rótulos nos pontos.
- Apêndice JSON: novo campo `item.metas[]` com schema e exemplo.

---

### Onde mexe (resumo técnico)

`public/Ferramenta_ER.html`:

| Bloco | Linhas aprox. | Mudança |
|---|---|---|
| Constantes `SCORES`, helpers `corStatus`/`iconeStatus` | 1256, 1346–1356 | Novos nomes (Avançando/Retrocesso) |
| `calcItemEixo6` | 1135–1235 | Status geral renomeado + novo `statusMeta` por item |
| `calcEixo6` | 1207–1340 | Retorna `{ geral:{...}, meta:{...} }`; novas faixas/travas vs-meta |
| `normalizeEixo6`/item | 809–865 | Adiciona `item.metas[]`; valida `unidade` obrigatória |
| `normalizeCoalizao` | migração | Mapeia status legados Fechando/Abrindo |
| UI card índice | 1589–1640 | 2 colunas (geral × meta) + nome do índice |
| UI item Eixo 05 | 1740–1830 | Sub-tabela `metas` + selo vs-meta |
| `upEixo6Item`/`upEixo6Config` | 2180+ | Handlers para `metas[]` (add/remove/edit) |
| PPT slide-síntese | 2360–2390 | Faixa em 2 linhas com nome novo |
| PPT loop principal | 2540–2800 | Reordena: síntese 2º, gráficos antes dos gaps |
| PPT slide Eixo 05 | 2540–2760 | Rodapé 3 blocos, selo duplo por item, unidade própria |
| PPT gráficos | 2761–2800 | `showValue:true`, format por unidade, eixo Y com unidade |

Artefatos:
- `public/dados_teste_variabilidade.json` (substituído + cópia `/mnt/documents/`)
- `/mnt/documents/Logica_Sistematizacao_Executivo.md` (atualizado)
- `/mnt/documents/Logica_Sistematizacao_Detalhado.md` (atualizado)

---

### Fora de escopo
- Maturidade dos Eixos 01–04, radar, dashboard (apenas refletem novos rótulos onde citarem o índice).
- DOCX.
- Validação cruzada entre meta e marco (ex.: meta antes do marco) — apenas avisa, não bloqueia.