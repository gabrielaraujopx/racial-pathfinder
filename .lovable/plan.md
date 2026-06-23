## Eixo 05 — Redesenho do Índice + JSON de teste, docs e gráficos BA×PPI no PPT

Tudo entregue numa rodada só. Mudanças em `public/Ferramenta_ER.html` + 3 artefatos novos.

---

### Parte A — Lógica do Índice de Impacto (Eixo 05)

#### A1. Barreira de dados por item (não por eixo)
- Remove o toggle de barreira geral do Eixo 05; cada item ganha `barreira:{ativa, justificativa}`.
- Item com barreira: status **`Barreira de dados`** (⚖), não pontua, não entra na média, mas conta como "listado" para a cobertura cruzada.
- Sem compensação numérica.

#### A2. Status por antes × depois com janela simétrica
```
marco        = item.coalizaoAnoInicio || ctx.coalizaoAnoInicio
N            = min(#pts ano<marco, #pts ano>=marco)
gap_antes    = média dos N pontos imediatamente antes do marco
gap_depois   = média dos N pontos a partir do marco
delta        = (gap_depois - gap_antes) normalizado pelo sentidoDesejado
thr          = max(0.05·|gap_antes|, 0.05·limiarGapAbsoluto)
delta<=-thr → Fechando | delta>=thr → Abrindo | senão → Estagnado
```
- `N==0` em algum lado → `Insuficiente`.
- Sem marco → fallback regressão `geral` + flag `semBaseline`.
- `item.barreira.ativa` curto-circuita tudo.

#### A3. Cobertura cruzada Eixo 02 × Eixo 05 entra no cálculo
```
nIndic02       = total de indicadores do Eixo 02
nListados05    = itens listados (inclui barreira)
nPontuaveis05  = itens com status Fechando/Estagnado/Abrindo
nBarreira05    = itens Barreira de dados
denomCob       = max(3, nIndic02)
fatorCobertura = nListados05 / denomCob          // penaliza ausência
fatorMedicao   = nPontuaveis05 / max(1, nListados05 - nBarreira05)
mediaScore     = média scores dos pontuáveis
pctBruto       = round(mediaScore · fatorCobertura · fatorMedicao)
```
- `nPontuaveis05==0` → pct=0, nível **`Sem medição`**.
- Sem Eixo 02 → `denomCob = max(3, nListados05)`.

Travas qualitativas (após pctBruto):
- `nFechando==0` → teto **Avanço incipiente**.
- `nFechando==0 ∧ nAbrindo>nEstagnado` → **Gaps se abrindo**.
- `nFechando==0 ∧ nAbrindo==0 ∧ nEstagnado>0` → **Estagnado**.

Faixas (se nenhuma trava): ≥70 Reduzindo gaps · ≥40 Avanço parcial · ≥20 Avanço incipiente · <20 Sem avanço.

#### A4. Visibilidade antes×depois + cobertura cruzada (UI + PPT)
- **Card UI** e **faixa Impacto do slide-resumo** + **rodapé do slide do Eixo 05**: 3 linhas — (1) nível·pct, (2) `Gap médio antes→depois: X → Y (Δ ±z)`, (3) `Cobertura Eixo 02→05: N/M · barreira em K · Fech/Estag/Abr/Insuf`.

#### A5. Aviso de barreira em massa
- `nBarreira05 / max(1,nListados05) >= 0.5` → badge laranja "⚠ Metade ou mais dos gaps sob barreira — interpretar com cautela" no card UI e faixa amarela fina no topo do slide Eixo 05.

---

### Parte B — Visualização gráfica BA × PPI no PPT (novo)

- **Checkbox no Eixo 05**: "Incluir slide(s) de visualização gráfica BA×PPI no PPT" (`e6.incluirGraficos`, default false). Persistido na coalizão.
- Quando marcado, **após o slide do Eixo 05** de cada coalizão, gera 1+ slides com **gráficos de linha** (um por indicador listado):
  - 2 séries por gráfico: linha **BA** (`brancosAmarelos`) e linha **PPI** (`pretosPardosIndigenas`), eixo X = ano da série temporal do item (não usa o gap calculado).
  - Marco da coalizão desenhado como linha vertical pontilhada com rótulo "Início coalizão".
  - Título do gráfico = nome do indicador; subtítulo = "Fonte: …" se houver.
  - Item com `barreira.ativa` aparece como card vazio com selo ⚖ "Sem dados — barreira reconhecida".
- **Layout por slide**: grid 2×2 (até 4 gráficos por slide). Se a coalizão tiver >4 itens, quebra em slides adicionais (`Eixo 05 — Gráficos BA×PPI (2/3)`).
- Usa `pres.addChart(pres.ChartType.line, ...)` do pptxgenjs (já carregado no arquivo). Cores: BA = `#7A6A60` (marrom escuro do tema), PPI = `#C97B3A` (laranja do tema). Linha do marco em cinza tracejada.

---

### Parte C — JSON de teste atualizado

Substitui `public/dados_teste_variabilidade.json` (também copiado para `/mnt/documents/`) cobrindo:

- **C1**: coalizão com Eixo 02 = 8 indicadores, Eixo 05 = só 3 itens → testa queda do `fatorCobertura` (3/8).
- **C2**: coalizão com Eixo 02 = 8, Eixo 05 = 8, sendo 5 com `barreira.ativa=true` → testa que **não derruba** mas dispara aviso de barreira em massa (5/8 ≥ 50%).
- **C3**: Anos Finais — 2 itens com barreira (substitui o caso "Sem medição"), valida nível `Sem medição`.
- **C4**: Tecnologia — Estag/Estag/Abrindo, sem Fechando → valida trava "Avanço incipiente" + nível "Gaps se abrindo" se `nAbr>nEstag` (mantido aqui em "Estagnado prevalece").
- **C5**: coalizão com marco 2021 e série 2015–2025 com pontos assimétricos → valida janela simétrica (toma 4 antes / 4 depois).
- **C6**: coalizão totalmente positiva — todos Fechando, ≥70% → "Reduzindo gaps".
- **C7**: coalizão sem `coalizaoAnoInicio` em alguns itens → valida fallback `semBaseline` e badge "sem baseline".
- **C8**: coalizão com `incluirGraficos=true` e séries BA/PPI ricas para validar visualmente os slides de gráfico (inclusive um item com barreira para validar card vazio).

Garante cobertura de todos `statusQualidade`, `tipoRacial`, `criticidade`, `sentidoDesejado`, barreiras de Eixos 02/03/04 (intactas).

---

### Parte D — Documentação (2 documentos em `/mnt/documents/`)

#### D1. `Logica_Sistematizacao_Executivo.md` (objetivo, ~3 páginas)
- O que mede a ferramenta, para quem.
- Os 5 eixos em 1 parágrafo cada.
- Como o **Índice de Impacto** é lido: significado dos níveis, o que conta como "avanço", por que cobertura cruzada importa, papel da barreira de dados — **em linguagem de gestor, com 1 exemplo numérico narrado** ("se a coalizão lista 8 indicadores no Eixo 02 mas só acompanha 3 gaps, o índice começa em 3/8 da nota possível…").
- Sem fórmulas: tabelas comparativas e analogias.

#### D2. `Logica_Sistematizacao_Detalhado.md` (completo, ~10 páginas)
- Estrutura por eixo: campos, regras de obrigatoriedade, cálculo da maturidade, barreiras.
- Eixo 05 detalhado: fórmula janela simétrica, cobertura cruzada, fatores, travas qualitativas, faixas, casos de borda.
- Cada fórmula seguida de **leitura em português** ("isso quer dizer que…") e **2 exemplos numéricos** trabalhados.
- Tabela de status possíveis × cor × significado.
- Apêndice: campos do JSON, mapeamento Eixo 02 ↔ Eixo 05, regras de migração de coalizões legadas.

Ambos referenciam o JSON de teste e indicam quais coalizões ilustram cada regra.

---

### Onde mexe (resumo técnico)

`public/Ferramenta_ER.html`:

| Bloco | Linhas | Mudança |
|---|---|---|
| `defaultEixo6` / `normalizeEixo6` | 809–855 | Remove `e6.barreira`, adiciona `e6.incluirGraficos`, `item.barreira` |
| `calcItemEixo6` | 1135–1206 | `_statusAntesDepois` com janela simétrica; curto-circuito de barreira |
| `calcEixo6` | 1207–1259 | Recebe `c`; nova fórmula cobertura cruzada; travas; nível `Sem medição` |
| Chamadas a `calcEixo6` | 1400, 1941, 2146, 2246, 2664 | Passar `c` |
| UI Eixo 05 | 1547–1601 | Remove barreira geral, adiciona checkbox `incluirGraficos` e toggle barreira por item |
| Card Índice (UI) | 1511+ | 3 linhas + badge aviso barreira em massa |
| PPT slide-resumo | 2246–2275 | Faixa Impacto com 3 linhas + badge |
| PPT slide Eixo 05 | 2540–2560 | Rodapé com 3 blocos + faixa amarela se barreira em massa |
| PPT slides gráficos (novo) | após 2560 | Geração condicional de slides com `addChart` BA/PPI |
| Migração | `normalizeCoalizao` | `e6.barreira` antigo → ignorado; itens herdam `barreira:{ativa:false}` |

Novos artefatos:
- `public/dados_teste_variabilidade.json` (substituído) + cópia em `/mnt/documents/`
- `/mnt/documents/Logica_Sistematizacao_Executivo.md`
- `/mnt/documents/Logica_Sistematizacao_Detalhado.md`

---

### Fora de escopo
- Barreiras dos Eixos 02/03/04 (mantidas).
- DOCX (sem alteração).
- Radar/dashboard (apenas novos rótulos de nível).
