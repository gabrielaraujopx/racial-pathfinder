
## Escopo

Quatro mudanças no `public/Ferramenta_ER.html` + geração de um JSON de teste. Tudo entregue em uma única rodada de implementação. A geração do JSON é a tarefa mais barata em créditos, então será priorizada e entregue mesmo se algum dos outros itens precisar de refino posterior.

---

## 1. Status de avanço do Índice de Impacto = antes vs depois da coalizão

Hoje `calcItemEixo6` já produz três janelas (`geral`, `antes`, `depois`) usando o `coalizaoAnoInicio` como marco. O status oficial sai da janela `geral` (regressão linear sobre toda a série). Vamos trocar isso por uma comparação direta antes×depois, que mede de fato o impacto da coalizão.

**Nova regra de status (por item):**

```text
gap_antes  = gap médio dos pontos com ano ≤ marco
gap_depois = gap médio dos pontos com ano > marco
delta      = gap_depois - gap_antes   (sentido já normalizado)
thr        = max(0.05 * |gap_antes|, 0.05 * limiarGapAbsoluto)

se delta <= -thr            → Fechando
se delta >=  thr            → Abrindo
caso contrário              → Estagnado
```

**Regras de borda:**
- Sem `marcoAno` definido OU sem ≥1 ponto antes E ≥1 ponto depois → cai no fallback atual (status pela inclinação da janela `geral`) e o item ganha um sufixo `(sem baseline)` no PPT/UI.
- Coalizões com pontos só "depois" continuam medindo evolução intra-coalizão.
- Item segue só pontuando se `geral.pontuavel` (≥ `pontosMinimos`).

**Onde mexe:** `_classifica*` interno em `calcItemEixo6` (extrair função `_statusAntesDepois`) + qualquer leitura de `item.calc.status` (já é genérica). Tooltip e copy do card "antes/depois" passam a explicar a nova lógica.

---

## 2. Eliminar Eixo 05 — Evidências e mover "Fonte" para Diagnóstico e Indicadores

**Remover (UI + dados + PPT):**
- Seção `Evidências` no formulário (`lista-evidencias`, `addItem('evidencias')`, `renderItemEvidencia`).
- Array `c.evidencias`, default e normalização.
- Faixa "05 — EVIDÊNCIAS" no slide da coalizão (libera ~0,55" de altura).
- Linha do dashboard `Evidências registradas`.

**Adicionar campo `fonte` obrigatório em dois lugares:**

| Item | Estrutura | Onde aparece |
|---|---|---|
| Diagnóstico (`c.diagnostico[i]`) | `{ texto, fonte }` | UI: input "Fonte" abaixo do texto, com asterisco vermelho se vazio. PPT: o texto vai como `"<texto> (Fonte: <fonte>)"` no slide. |
| Indicador (`c.indicadores[i]`) | adiciona `fonte` | UI: input "Fonte" na linha já compacta. PPT: texto vira `"<texto> (Fonte: <fonte>)"`. |

**Obrigatoriedade:** validação visual apenas (badge ⚠ "Sem fonte" + borda vermelha no input). Sem bloqueio de salvamento, pra não quebrar coalizões legadas.

**Migração:** itens antigos recebem `fonte = ''` no `normalizeCoalizao`. Texto antigo do array `evidencias` é descartado (já existia conceitualmente solto, vira responsabilidade da fonte por item).

**Renumeração de eixos no PPT/UI:**
- `01 Diagnóstico` (faixa) · `02 Indicadores` · `03 Estratégias` · `04 Lacunas` · `05 Gaps BA×PPI` (era 06).
- Renomear referências textuais a "Eixo 06" → "Eixo 05" no documento de lógica e nos rótulos do slide do Índice de Impacto. **Identificadores internos (`eixo6`, `lista-eixo6`, ids de campo) ficam como estão** para não quebrar JSONs salvos.

---

## 3. Realocação de espaço no slide da coalizão

Slide é 10×5,625". Hoje: header ~0,42 + faixa Impacto ~0,32 + faixa Diagnóstico ~0,38 + 3 colunas (eixos 1–4) + faixa Evidências `evH ≈ 0,55`. Sai Evidências, sobra ~0,55" para redistribuir.

**Nova distribuição:**
- Faixa Índice de Impacto: altura sobe de `0,32"` → **`0,55"`**, fonte do título de 8 → **11** (negrito), badge de % maior. Cor de fundo levemente tingida pela cor do nível pra ganhar destaque visual.
- Faixa Diagnóstico: mantém ~0,38" mas o texto interno ganha 1 linha extra (já cabia mal).
- Colunas de Indicadores/Estratégias/Lacunas: altura sobe ~0,30" cada → cabem **+2 itens** por coluna em média (slice atual 8/8/6 sobe para 10/10/8) e o texto dos itens pode subir de `fontSize 10 → 11` mantendo `shrinkText`.

**Constantes a alterar (linhas atuais ~2280–2470):** `evH` zerada, `colH` recalculada (`H - colY`), `itemFS = 11`, slices de `c.indicadores`/`c.estrategias`/`c.lacunas`, faixa Impacto (`impH`, `fontSize`, `bold`).

---

## 4. JSON de teste com máxima variabilidade

Arquivo novo `public/dados_teste_variabilidade.json` (carregável pelo botão "Importar JSON" atual).

**7 coalizões**, cada uma combinando cenários diferentes:

| # | Coalizão | Cenário coberto |
|---|---|---|
| 1 | Alfabetização | "Tudo cheio, alto desempenho" — 5 diagnósticos com fonte, 5 indicadores (3 gerais + 2 já-racializados), 5 estratégias todas com status diferentes, 4 lacunas com criticidade variada, Eixo 5 com 4 itens BA/PPI, séries de 8 anos, marco da coalizão = 2018, todos Fechando. |
| 2 | Ensino Médio | "Pontos só depois do marco" — força fallback de status sem baseline. Mix Abrindo/Fechando/Estagnado. |
| 3 | Equidade | "Coalizão nova" — poucos pontos, vários `Insuficiente`, barreira de dados públicos ativa em 2 eixos. |
| 4 | Profissionalização | "Volátil + reversões" — séries com R² baixo e reversões deliberadas; mistura `tipoRacial`. |
| 5 | Gestão | "Diagnóstico sem fonte" — gera badges ⚠ para validar UI; lacunas declaradas como "Não identificamos lacunas relevantes". |
| 6 | Financiamento | "Indicadores em lacuna racial" — vários `Geral racializável` sem contrapartida no Eixo 5, derruba cobertura para ~30%. |
| 7 | Primeira Infância | "Estado intermediário" — tudo Em desenvolvimento, com 1 item de cada status em Eixo 5, recorte de PPT customizado por item. |

**Cobertura garantida pelo JSON:**
- Todos os valores de `statusQualidade` (Tem / Tem parcialmente / Não tem) e `resultadoEstados`/`resultadoNacional` (S/P/N).
- Ambos `tipoRacial` (`geral`, `especifico_racial`).
- Todos os `criticidade` e `encaminhamento` de lacunas.
- Todos os `status` de estratégias.
- `barreirasContexto.{indicadores,estrategias,lacunas}` ativa em pelo menos 1 coalizão cada.
- Eixo 5: `sentidoDesejado` subir/descer; séries com 1, 2, 3, 5 e 8 pontos; `coalizaoAnoInicio` global + override por item; `pptAnoInicial/Final` global + override por item; `limiarGapAbsoluto` variado.
- Casos de fallback de status (sem baseline) e casos de baseline cheio.

JSON entregue como artefato pronto para `/mnt/documents/dados_teste_variabilidade.json` (download direto) **e** copiado para `public/` para o botão de carga na ferramenta. Validação mínima: rodar o `normalizeCoalizao` mentalmente sobre cada coalizão.

---

## Detalhes técnicos — onde mexer no arquivo

`public/Ferramenta_ER.html`:

1. **Defaults/normalize** (~750–890): remover `evidencias`, adicionar `fonte` a `defaultDiagnostico` e `defaultIndicador`/normalizeIndicador.
2. **`_statusAntesDepois`** novo + chamada dentro de `calcItemEixo6` (~1134–1177). `geral` segue computado mas o `status` propagado vem do antes×depois (com fallback `geral`).
3. **Render dos cards de Diagnóstico (~700–720) e Indicador (~1809–1865)**: novo input `fonte` + badge ⚠ condicional.
4. **Render do Eixo 5/Evidências (~1468–1475)**: remove a seção.
5. **PPT slide coalizão (~2200–2480)**: nova altura/fonte da faixa Impacto, removida faixa Evidências, slices ampliados, texto do diagnóstico/indicador com `(Fonte: …)`.
6. **Plano `.lovable/plan.md`**: atualizar.

---

## Fora de escopo

- Não mexer no Eixo 5 (Gaps BA×PPI) além da renumeração e da nova regra de status.
- Não alterar fórmula de cobertura racial nem barreiras.
- Não alterar exportação DOCX nesta rodada (só PPT) — caso o DOCX referencie evidências, remove só a seção correspondente.
