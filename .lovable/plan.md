## Ajustes finais — padronização de status, metas BA/PPI ano-a-ano e títulos dos campos de gap

Três frentes de ajuste em `public/Ferramenta_ER.html` (UI + PPT) e atualização das docs.

---

### 1. Padronização do vocabulário de status (Índice de Redução de Gaps)

Hoje o índice "vs Meta" usa rótulos diferentes (`Superando metas`, `No ritmo da meta`, `Aquém da meta`, `Sem redução vs meta`, `Em retrocesso vs meta`) e por item (`Redução acima/dentro/abaixo da meta`, `Não há redução`, `Em retrocesso`). Vamos unificar tudo no vocabulário das chips de contagem:

| Onde | Antes | Depois |
|------|-------|--------|
| Status por item (`statusMeta`) | Redução acima da meta | **Acima** |
| | Redução dentro da meta | **Dentro** |
| | Redução abaixo da meta | **Abaixo** |
| | Não há redução | **Sem redução** |
| | Em retrocesso | **Retrocesso** |
| | (sem meta no item) | **Sem meta declarada** |
| Nível agregado da coalizão (`nivelMeta`) | Superando metas | **Acima** |
| | No ritmo da meta | **Dentro** |
| | Aquém da meta | **Abaixo** |
| | Sem redução vs meta | **Sem redução** |
| | Em retrocesso vs meta | **Retrocesso** |
| | Sem metas declaradas | **Sem metas declaradas** (mantido) |

Refletido em: `statusMetaCor`, `nivelMetaCor`, contagens (`contagensMeta`), trava qualitativa, chips do dashboard, card do índice, tabela de síntese comparativa, JSON interno (`calc.statusMeta`, `meta.nivel`) e textos das docs.

### 2. Slide da coalizão — exibir status vs-meta no Índice

No slide individual de cada coalizão (PPT), a área dedicada ao "Índice de Redução de Gaps" hoje só mostra o agregado. Vamos:

- Acrescentar, na faixa de cabeçalho do índice no slide da coalizão, a chip-resumo das contagens (mesmo padrão visual do dashboard da ferramenta): `▲▲ Acima · ✓ Dentro · ↘ Abaixo · = Sem redução · ▲ Retrocesso · — Sem meta declarada`.
- Em cada card de item desse slide, garantir que o badge "vs Meta" exiba o status novo (Acima/Dentro/Abaixo/Sem redução/Retrocesso/Sem meta declarada) com a cor correspondente.
- Mesmo bloco de contagens também aparece na visualização do índice dentro da ferramenta (card e síntese), já consistente com a UI atual.

### 3. Metas: BA, PPI e/ou gap-alvo ano-a-ano

Hoje a tabela METAS aceita só `valorMetaGap` por ano. Vamos ampliar o modelo e a UI:

**Modelo de dados** (`it.metas[j]`):
- `ano: number`
- `valorMetaBA: number|null` (novo)
- `valorMetaPPI: number|null` (novo)
- `valorMetaGap: number|null` (mantido; derivável de BA/PPI se ambos preenchidos)

Regra de derivação: se `BA` e `PPI` preenchidos e `valorMetaGap` vazio, calcula gap conforme `sentidoDesejado` (`subir`: BA−PPI; `descer`: PPI−BA). Se `valorMetaGap` foi preenchido manualmente, prevalece. Cálculo de `statusMeta` continua usando `valorMetaGap` (direto ou derivado) — sem mudança na fórmula.

**UI de preenchimento** (linha de meta-ano):
- Colunas: `Ano | Meta BA | Meta PPI | Gap-alvo | ×`
- Texto auxiliar: "Preencha BA e PPI para projeção visual nos gráficos, ou só o Gap-alvo para indicar a redução esperada."

**Gráficos em linha (UI + PPT)**:
- Se houver pelo menos 2 pontos de meta com `valorMetaBA` preenchido → desenhar uma série tracejada "Meta BA" estendendo a linha BA até o último ano-meta.
- Mesma coisa para `valorMetaPPI` → série tracejada "Meta PPI".
- Se houver só `valorMetaGap` (sem BA/PPI) → desenhar marcadores/anotação no gráfico mostrando o gap-alvo de cada ano-meta (linha pontilhada horizontal entre BA e PPI projetados a partir do último ponto real, com label "gap-alvo: X"), apenas como referência visual.
- Aplica a `renderGraficoEixo6` (UI Chart.js) e ao gerador de gráfico do PPT (chart.js → imagem).

### 4. Títulos dos campos de gap acompanham a unidade

Atualmente os labels dos inputs no Eixo 02 (gaps) embutem a unidade no título (ex.: `Gap-alvo (R$/aluno)`, `BA (%)`, `PPI (X)`) e não re-renderizam quando o usuário troca o campo "unidade". Solução:

- Manter `( )` no título, mas torná-lo reativo: ao alterar `it.unidade`, re-renderiza o item (já existe `renderEixoItem`/`upItem`; vamos garantir que mudar unidade dispare `render()` do eixo, não só um `up` mudo).
- Sub-rotina utilitária `unidadeLabel(u)` que retorna `(u)` se `u` preenchido e não-vazio, ou string vazia caso contrário — usada em todos os títulos: `BA${unidadeLabel(it.unidade)}`, `PPI${unidadeLabel(it.unidade)}`, `Gap-alvo${unidadeLabel(it.unidade)}` e nos novos `Meta BA${...}`, `Meta PPI${...}`.
- Mesma reatividade aplica nos gráficos (eixo Y e datalabels) — já é dinâmico via `it.unidade`, só precisa do re-render.

---

### Arquivos tocados

- `public/Ferramenta_ER.html`
  - `statusMetaCor`, `nivelMetaCor`, blocos de contagem (`contagensMeta`), `_calcStatusVsMeta` (renomes de strings retornadas), chips do dashboard (linhas ~1770), card do índice e síntese comparativa, tabela de metas (linhas ~2017–2042) com colunas BA/PPI, `upEixo6Meta` aceitando `valorMetaBA`/`valorMetaPPI`, função de gráfico em linha (UI + PPT), labels de input de série/meta com `unidadeLabel`, listener de `upEixo6Item('unidade', …)` disparando re-render do item.
  - Geração do PPT: slide da coalizão recebe chips de contagem vs-meta na faixa do índice; itens individuais usam novos rótulos de status; gráficos com projeção BA/PPI tracejada ou anotação de gap-alvo.
- `/mnt/documents/Logica_Sistematizacao_Executivo.md` e `Logica_Sistematizacao_Detalhado.md`: atualizar nomenclatura dos status (Acima/Dentro/Abaixo/Sem redução/Retrocesso/Sem meta declarada) e descrever as novas metas BA/PPI/gap-alvo + projeção no gráfico.
- `public/dados_teste_variabilidade.json`: ampliar alguns cenários para incluir `valorMetaBA`/`valorMetaPPI` em alguns itens (mantendo outros só com `valorMetaGap`) para validar ambas as visualizações.

### Fora de escopo
- Fórmula de cálculo do índice ou faixas (mantidas — só renomeio de strings).
- Hierarquia vs-Meta principal / Geral secundário (já implementado).
