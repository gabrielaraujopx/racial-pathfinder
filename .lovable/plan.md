## Atualização do JSON de teste (`public/dados_teste_variabilidade.json`)

A estrutura do JSON já contempla os campos novos (`unidade`, `valorMetaBA`, `valorMetaPPI`, `valorMetaGap` por ano, `sentidoDesejado`), mas há lacunas que reduzem a cobertura de teste dos ajustes recentes. Proponho as seguintes mudanças, mantendo o que já existe:

### 1) Preencher itens vazios para cobrir todos os status novos

Hoje 6 itens estão com `serie: []` e `metas: []` (todos com `barreira.ativa: true`). Vou:

- **Manter como "barreira ativa"**: `Material didático adotado (%)` e `Frequência (%)` (Alfabetização) — preservam o cenário de "sem dados por barreira".
- **Preencher com série + metas**:
  - **Anos Finais → "Português 9º ano"** (unidade `Ponto SAEB`): série 2015–2023 com gap fechando bem → demonstrará **Acima** (vs meta) e unidade SAEB no gráfico/labels.
  - **Anos Finais → "Matemática 9º ano"** (unidade `Ponto SAEB`): série 2015–2023 com gap praticamente estável → demonstrará **Sem redução**.
  - **Alfabetização → "Alfabetização aos 7 (%)"**: série fechando, mas abaixo do ritmo de meta → **Abaixo**.
  - **Alfabetização → "Formação específica (%)"**: série SEM `metas` declaradas → **Sem meta declarada** (campo `metas: []` proposital, sem barreira).

### 2) Forçar a presença de "Retrocesso" no dataset

Ajustar a série de um indicador existente (sugiro `Tecnologia → Letramento digital (%)`) para que o gap aumente nos últimos 2 anos, produzindo o status **Retrocesso** vs meta.

### 3) Coalizão "Professores"

Definir `coalizaoAnoInicio: 2021` (hoje `null`) para que o cálculo do índice considere o período pós-coalizão.

### 4) Diversidade de unidades já coberta (sem mudança)

`%`, `R$/aluno`, `horas`, `anos`, `Ponto SAEB` — já presentes.

### Resultado esperado

Após a atualização, o dataset cobrirá os 6 status do "vs meta":
`Acima` · `Dentro` · `Abaixo` · `Sem redução` · `Retrocesso` · `Sem meta declarada`,
e o slide/dashboard de cada coalizão renderizará todos os elementos novos (linhas tracejadas Meta BA/PPI, anotações 🎯 Gap-alvo, chips de status, unidades reativas).

### Arquivo afetado

- `public/dados_teste_variabilidade.json` (somente este).

Confirma que posso aplicar?