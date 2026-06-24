## Diagnóstico

O ajuste anterior do `estH` (medição via canvas + line-height 0.0170) foi aplicado **apenas aos cálculos de altura compartilhados** entre Indicadores, Estratégias e Lacunas — então em tese todas as colunas usam o mesmo estimador. Mas o usuário observou:

- **Eixo Estratégias (Gestão Escolar e Pré-escola)**: itens 1 e 2 quase colados — "equitativa" do item 1 encostando em "mentoria" do item 2.
- Lacunas: precisa ser revalidada com o mesmo padrão.

Causas prováveis a investigar com o PPT real gerado a partir do JSON do usuário (`/mnt/user-uploads/ER_dados_2026-06-24.json`):

1. **Coluna Estratégias é mais estreita** (~1.93") que Indicadores e Lacunas porque reserva 0.90" para o badge de status à direita. Texto longo quebra em mais linhas → o `estH` precisa ser ainda mais preciso aqui. Se o canvas browser estiver renderizando com fonte de fallback (Arial em vez de Calibri), a largura medida é ~3-5% menor que a real do PowerPoint, e em coluna estreita isso vira 1 linha a menos no estimador → colisão.
2. O `_gapBase = 0.08"` (≈ 5.8pt) é o único espaço entre itens. Se o `estH` subestimar mesmo 0.05" por item, dois itens consecutivos com texto longo se sobrepõem visualmente.
3. Em Lacunas, há a complicação extra do bloco `[texto + recomendação]` numa única `addText` multi-runs — o cálculo soma `tH + recH` como se fossem caixas separadas, mas o PowerPoint renderiza tudo numa caixa só, podendo herdar line-height diferente.

## Plano de correção

Tudo no arquivo `public/Ferramenta_ER.html`, mexendo apenas no estimador `estH` e nas funções de altura `_hEstPre` / `_hLacPre`:

1. **Reproduzir o problema**: rodar o export com o JSON real do usuário via Playwright, renderizar os slides de Gestão Escolar e Pré-escola (eixo Estratégias) e Lacunas em imagens, medir o overlap em polegadas.

2. **Calibrar o `estH` para underestimação de largura**:
   - Aplicar um fator de segurança de largura no canvas: usar `maxPx = wIn * 96 * 0.96` (assumir que 4% do espaço útil é consumido pela diferença Calibri vs fallback do browser). Isso aumenta a contagem de linhas só nas situações em que o texto realmente está perto do limite — não afeta itens curtos.
   - Manter o line-height 0.0170 e margem 0.03" (já validados em Indicadores).

3. **Reforçar a folga vertical nas funções de altura por coluna**:
   - `_hEstPre`: somar uma pequena folga de descender (`+ 0.02`) só quando `estH` retornar > 1 linha, para evitar colisão de descenders de uma linha com ascenders da próxima do item seguinte.
   - `_hLacPre`: quando há recomendação, calcular `estH(txt+'\n'+recTxt, lTW, max(itemFS,7))` como bloco único (em vez de soma separada), garantindo que a estimativa reflita o que o PowerPoint efetivamente renderiza dentro da mesma caixa multi-run.

4. **Validar visualmente** rodando o export novamente, renderizando especificamente:
   - Gestão Escolar parte 1 (coluna Estratégias)
   - Pré-escola parte 1 e parte 2 (coluna Estratégias)
   - Todos os slides de eixo de qualquer coalizão com Lacunas preenchidas
   
   Critério de sucesso: nenhum par de itens com gap visual < ~0.05" e nenhuma palavra de um item encostando em outro. Se ainda houver colisão, aumentar o `_gapBase` de 0.08 → 0.10 (último recurso, custa ~0.1" por item paginado, mas pode forçar +1 paginação em casos extremos — só faço se a calibração de width não resolver).

5. **Não mexer** em: layout, fontes, cores, badges, largura de colunas, capa, sumário, gráficos BA×PPI ou qualquer slide fora dos eixos de coalizão.

## Arquivos afetados

- `public/Ferramenta_ER.html` — apenas o bloco do `estH` (linhas 2625–2667) e as funções `_hEstPre` / `_hLacPre` (linhas 2821–2829).

## Risco

Baixo. O ajuste é conservador: aumentar levemente a altura estimada pode disparar paginação 1 slide a mais em casos limítrofes, mas continua respeitando o princípio aprovado (só pagina quando o próximo item realmente não cabe no rodapé). Validação visual obrigatória antes de declarar concluído.
