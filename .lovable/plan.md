# Plano — Recorte temporal e análise segmentada no Eixo 06

Aprimora o Eixo 06 com dois recursos complementares: controle de quais anos da série aparecem no PPT e segmentação da análise estatística em três janelas (Geral, Antes e Depois do início da atuação consistente da coalizão).

## 1. Novos parâmetros no Eixo 06

Adicionados na configuração geral do Eixo 06 (nível coalizão) e replicáveis por indicador:

- **Recorte para o PPT** — `pptAnoInicial` e `pptAnoFinal` (globais da coalizão).
- **Marco da coalizão** — `coalizaoAnoInicio`: último ano em que a atuação consistente da coalizão **ainda não existia** (ex.: se a atuação começou em 2020, o marco é 2019, último ponto pré-atuação).

Cada indicador ganha campos opcionais de override:
- `pptAnoInicialOverride`, `pptAnoFinalOverride`
- `coalizaoAnoInicioOverride`

Quando o override está vazio, herda o valor da coalizão. UI mostra "(herdado: 2019)" como hint.

## 2. Segmentação Antes / Depois (ponto de sobreposição)

Dado o marco `M` (ano medido mais próximo ao início da atuação consistente):

- **Antes** = pontos com `ano ≤ M`
- **Depois** = pontos com `ano ≥ M`
- O ponto `M` aparece nos dois períodos (continuidade visual e estatística).

Cada período exige ao menos 2 pontos para calcular inclinação/Δ; com 1 ponto só, exibe "—" e nota "amostra insuficiente".

## 3. Três análises por indicador

O `calcItemEixo6` passa a devolver três blocos com a mesma estrutura já existente (inclinação, R², Δ acumulado %, modo absoluto, volátil, status):

- **geral** — todos os anos preenchidos.
- **antes** — anos ≤ marco.
- **depois** — anos ≥ marco.

O status oficial do indicador (para pontuação/agregação do Eixo 06) continua vindo do bloco **geral**, mantendo a fórmula atual intacta.

## 4. UI da ferramenta (Eixo 06)

No cabeçalho do Eixo 06:
- Inputs para `pptAnoInicial`, `pptAnoFinal`, `coalizaoAnoInicio`, com tooltip explicando o ponto de sobreposição.

Em cada card de indicador:
- Linha de overrides opcionais (3 inputs pequenos, em branco = herdado).
- Três chips/blocos lado a lado: **Geral**, **Antes**, **Depois com destaque** (borda/fundo mais forte), cada um com inclin./ano, Δ acumulado (ou Δ absoluto), R² e flag Volátil.

## 5. PPT — slides do Eixo 06

Para cada indicador:
- **Tabela da série** filtrada ao recorte (`pptAnoInicial`–`pptAnoFinal` efetivos).
- **Três blocos lado a lado** com Inclinação/ano, Δ acumulado, R² e Volátil — **Depois** destacado (cor de fundo da coalizão / borda mais grossa).
- Linha-legenda: "Marco da atuação consistente: <ano>".

Se o recorte do PPT exclui o marco, mostra aviso textual "Marco fora do recorte exibido".

## 6. Documento de lógica

Nova subseção 6.x explicando:
- Como o recorte do PPT é apenas visual (não afeta cálculo).
- Como Antes/Depois são definidos com ponto de sobreposição.
- Por que o status oficial usa "Geral" (estabilidade da pontuação).
- Exemplo numérico com IDEB pré-escolar dividido em duas janelas.

## Detalhes técnicos

Arquivo: `public/Ferramenta_ER.html`.

Funções afetadas:
- `defaultEixo6`, `normalizeEixo6` — novos campos coalizão + override por indicador.
- `calcItemEixo6` — refatorar para função interna `_calcJanela(pontos)` reutilizada por geral/antes/depois; retornar `{ geral, antes, depois, recortePpt: {de, ate}, marco }`.
- `refreshImpacto` (render Eixo 06) — novos inputs + três blocos por card.
- Geração de PPT — novas tabelas/colunas e destaque visual do "Depois".
- Geração de DOCX/relatório (se aplicável) — refletir três janelas resumidas.

Regras:
- Recorte do PPT **não** altera cálculos (são feitos sobre a série completa, conforme janela).
- Override em branco = herda.
- Validações: ano inicial ≤ final; marco dentro da série; mensagens amigáveis sem bloquear o uso.

## Fora do escopo

- Mudar a fórmula de agregação do Eixo 06.
- Tocar em Eixos 1–5.
- Recorte/segmentação em outros eixos.
- Gráficos interativos novos (a tabela + chips continuam sendo a representação).
