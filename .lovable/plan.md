## Objetivo

Gerar dois artefatos para download:

1. `ER_dados_teste_v8.json` — base de dados de teste com variabilidade máxima.
2. `Teleprompter_Apresentacao_ER.docx` — roteiro de fala para apresentar a ferramenta às lideranças, referenciando exatamente o PPT que será gerado a partir desse JSON.

---

## 1) Regras do JSON

### Alfabetização (preservar)
- Copiar **byte-a-byte** do arquivo enviado (`ER_dados_2026-06-24 (5).json`), **única alteração**: no Eixo 05, no item "Avaliação de Fluência…", desmarcar a flag de barreira de dados (`barreiraDados: false` / remover marcação equivalente) para que série histórica e metas já preenchidas sejam consideradas no cálculo. Nada mais muda — diagnóstico, visão, indicadores, estratégias, lacunas, barreirasContexto, estadoLacunas, recorte PPT, marcos: tudo igual.

### Demais 6 coalizões — variabilidade máxima

Cada coalizão recebe um "perfil" diferente para exercitar todos os caminhos do sistema:

| Coalizão | Modo gráfico PPT | Eixo 05 preenchimento | estadoLacunas | Barreiras de contexto | Cenário de status esperado |
|---|---|---|---|---|---|
| Pré-escola | BA×PPI separado | BA+PPI ano a ano (%) | mapeadas | nenhuma | vs Meta: Dentro / Geral: Avanço parcial |
| Anos Finais | Gap unificado | Só gaps históricos + só gap-alvo (ppt SAEB) | mapeadas | barreira no Eixo 02 | vs Meta: Abaixo |
| Ensino Médio Integral | BA×PPI separado | Misto: 1 item BA+PPI (%), 1 só gap (R$/aluno), 1 sem metas | em mapeamento (sem itens) | barreira no Eixo 05 | vs Meta: Acima |
| Gestão Escolar | Gap unificado | BA+PPI (IDEB), 1 item retrocesso | mapeadas, recomendações variadas | nenhuma | vs Meta: Retrocesso (trava no Geral → Gaps se abrindo) |
| Professores | BA×PPI separado | Gaps históricos + meta BA/PPI apenas em parte dos anos | não mapeadas (sem itens) | barreira no Eixo 03 | vs Meta: Sem redução |
| Tecnologia | Gap unificado | Sem indicadores no Eixo 05 (todos com barreira) | em mapeamento | barreira no Eixo 05 (todos itens) | vs Meta: Sem metas declaradas |

Variações adicionais a distribuir entre as coalizões:
- **Diagnóstico**: variar entre 2–5 itens, alguns com fonte preenchida, outros sem.
- **Indicadores (Eixo 02)**: variar quantidade (3 a 8), `tipo` (Alcance/Transformação), `statusQualidade` (Tem/Parcial/Não tem), `tipoRacial` (geral/racializado/racializável), com e sem fonte. Pré-escola fica com 8 indicadores para validar paginação.
- **Estratégias (Eixo 03)**: variar quantidade (2 a 7) e `status` (Ideia, Piloto, Em escala). Gestão Escolar com textos longos para testar wrap.
- **Lacunas (Eixo 04)**: variar quantidade, `criticidade` (Baixa/Moderada/Alta), `encaminhamento` (sim/não), e duas coalizões sem itens conforme tabela.
- **Visão de sucesso**: textos curtos e longos.
- **Recorte PPT e marcos**: todas as 7 coalizões com `recortePpt` e `marcos` preenchidos com 2–4 marcos cada, datas variadas.
- **Eixo 05 – unidades**: usar `%`, `pontos SAEB`, `R$/aluno`, `IDEB`, `nº absoluto`, `pontos`. Misturar séries históricas e metas em ranges diferentes (ex.: histórico 2015–2023, metas 2020–2035).

---

## 2) Teleprompter (DOCX)

Documento em estilo fala corrida (frases curtas, pausas marcadas com `//`), organizado conforme a ordem real dos slides gerados:

1. **Abertura** — para que serve a ferramenta, contexto de equidade racial.
2. **Slide-síntese** — explicar Índice de Redução de Gaps (com vs Meta em destaque e Operacional secundário), Maturidade Operacional, e as 5 faixas de status; como ler os chips Acima/Dentro/Abaixo/Sem redução/Retrocesso.
3. **Slides de coalizão** — explicar cada bloco: Diagnóstico, Visão, Indicadores (com fonte entre parênteses), Estratégias, Lacunas, e a faixa do Índice (status principal vs meta + contagens + status geral).
4. **Slide de Gaps BA×PPI** — explicar dupla badge (Geral + vs Meta).
5. **Slides de gráficos** — explicar os dois modos (Gap unificado / BA×PPI separado), linhas reais vs projetadas, símbolo 🎯 para gap-alvo.
6. **Como o cálculo funciona (linguagem simples)** — Eixo 02 (atributos somados, média), Eixo 05 (média dos itens × cobertura × medição, com trava de coerência entre vs Meta e Geral).
7. **Glossário** — BA, PPI, SAEB, IDEB, PARC, Gap-alvo, Marcos, Recorte.
8. **Fechamento** — convite à validação pelas coalizões.

Cada seção citará exemplos concretos do JSON gerado (ex.: "olhem o slide de Professores, onde aparece 'Sem redução'…") para que a fala bata com o que projeta.

---

## Entrega

Ambos os arquivos em `/mnt/documents/` com tags `<presentation-artifact>` para download direto. Faço QA do DOCX (conversão para imagem) antes de entregar.
