## Objetivo

Atualizar `public/dados_teste_variabilidade.json` para que os indicadores do Eixo 6 cubram **todos os cenários** da regra de renderização dos gráficos BA×PPI no PPT, sem precisar editar a ferramenta manualmente.

## Regra atual do código (referência)

Para cada item do Eixo 6 com `incluirGraficos:true`:
- O gráfico só é desenhado se houver `serie` com pelo menos um ponto válido (`ano`, `valorBA`, `valorPPI`).
- **Projeção tracejada BA** aparece quando há `metas` com `ano > último ano histórico` e `valorMetaBA != null`.
- **Projeção tracejada PPI** idem com `valorMetaPPI`.
- **Anotação "🎯 Gap-alvo"** no rodapé do quadro aparece quando há metas futuras só com `valorMetaGap` (sem BA/PPI).
- Quando `serie` está vazia, mostra placeholder "Sem pontos válidos na série temporal".

## Cenários a cobrir (um por item)

Vou ajustar as coalizões com `incluirGraficos:true` (**Pré-escola**, **Alfabetização**, **Tecnologia**, **Ensino Médio Integral**, **Gestão Escolar**) para que os itens cubram, no conjunto, os 6 cenários abaixo. Cada cenário fica claramente identificado em pelo menos um item.

| # | Cenário | Onde | Como configurar |
|---|---|---|---|
| 1 | Projeção **BA + PPI** (duas tracejadas) | Tecnologia · item 1 "Acesso (%)" | metas futuras com `valorMetaBA` e `valorMetaPPI` preenchidos |
| 2 | Projeção **apenas BA** | Tecnologia · item 2 "Banda larga (%)" | metas futuras só com `valorMetaBA` (zerar `valorMetaPPI`) |
| 3 | Projeção **apenas PPI** | Tecnologia · item 3 "Letramento digital (%)" | metas futuras só com `valorMetaPPI` |
| 4 | **Sem projeção BA/PPI** + anotação 🎯 Gap-alvo | Gestão Escolar · item 1 "Diretoras PPI" | metas futuras só com `valorMetaGap` |
| 5 | **Sem metas futuras** (só linhas históricas) | Gestão Escolar · item 4 "Formação (h)" | apagar `metas` ou deixar todas com `ano <= último histórico` |
| 6 | **Série vazia** (placeholder "Sem pontos válidos") | Gestão Escolar · item 3 "Escolas c/ plano (%)" | `serie: []` |

Os demais itens (Pré-escola, Alfabetização, EMI) ficam como hoje — já cobrem variações de unidade (%, R$/aluno, horas, Ponto SAEB) e sentidoDesejado subir/descer, que também afetam o eixo, formato dos rótulos e cores das linhas.

## Outros ajustes pequenos no JSON

- Garantir que toda meta futura preserve `ano > último ano da série` (do contrário a projeção não aparece).
- Manter os totais de Eixo 6 coerentes para que o slide dedicado ainda calcule status e o índice.
- Não mexer nas outras coalizões nem em outros eixos.

## Arquivo afetado

- `public/dados_teste_variabilidade.json` (único arquivo alterado).

Nada será mudado na ferramenta (`Ferramenta_ER.html`).
