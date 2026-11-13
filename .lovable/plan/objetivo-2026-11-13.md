## Objetivo

Gerar um documento Word novo, `Manual_Funcionamento_Ferramenta_ER.docx`, explicando em detalhe como a ferramenta funciona hoje, baseado na leitura direta do código atual da ferramenta (não nos documentos antigos, que podem estar desatualizados).

## Estrutura do documento

1. **Visão geral** — propósito, público, fluxo de uso (preencher, salvar/carregar JSON, sincronizar, exportar PPT).
2. **Navegação e coalizões** — as 7 coalizões, abas, salvamento local, importação/exportação de JSON, envio ao backend.
3. **Eixo 01 – Diagnóstico** — campos, fonte obrigatória, regras de pontuação.
4. **Eixo 02 – Indicadores** — campos (tipo, qualidade, tipo racial, fonte), atributos somados, média 0–4 e faixas de maturidade.
5. **Eixo 03 – Estratégias** — campos, status (Ideia/Piloto/Em escala), cálculo.
6. **Eixo 04 – Lacunas Identificadas** — estado do mapeamento, criticidade, encaminhamento, cálculo.
7. **Eixo 05 – Gaps BA×PPI** — série histórica (BA+PPI ou só gap), metas (BA+PPI ou só gap-alvo), unidades, barreira de dados.
8. **Índice de Redução de Gaps** — leitura vs Meta (Acima/Dentro/Abaixo/Sem redução/Retrocesso/Sem metas), leitura Geral (faixas), fatores de cobertura e medição, trava de coerência, com exemplo numérico.
9. **Maturidade Operacional** — como os eixos se combinam.
10. **Questões complementares** — barreiras de contexto, recorte do PPT, marcos.
11. **Exportação do PPT** — ordem dos slides, slide-síntese, slides de coalizão (espaçamento adaptativo e paginação), slide de gaps, modos de gráfico (gap unificado / BA×PPI separado), linhas tracejadas, 🎯.
12. **Glossário** — BA, PPI, SAEB, IDEB, PARC, gap-alvo, recorte, marcos.
13. **Tabela-resumo de regras** — todas as faixas e limites em um só lugar.

## Como faço

- Leio as funções de cálculo e de geração do PPT no código atual para tirar as regras exatas (limites, pesos, faixas).
- Gero o Word com títulos, tabelas e exemplos; converto em imagens e confiro cada página antes de entregar.
- Nada na ferramenta é alterado.
