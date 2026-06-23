## Ajustes no PPT (4 pontos)

### 1) Slide da coalizão — faixa do Índice de Redução de Gaps
Arquivo: `public/Ferramenta_ER.html`, bloco `~2737-2759`.

Remover:
- Linha `l3` inteira (cobertura Eixo 02→05, Avan/Estag/Retr/Insuf, marco, aviso barreira em massa).
- Pedaço de `l2` com `Gap antes→depois ... (Δ ...)`.
- `%` numérico após o status `vs Meta` (em `l1`).
- `%` numérico após o status `Geral` em `l2` (manter só o rótulo, ex.: `Geral (sem comparar): Avanço parcial`).

Manter:
- `l1` (vs Meta): "Índice de Redução de Gaps · vs Meta: **Acima**" (sem `%`).
- `lChips` com a contagem por status (▲▲ Acima N · ✓ Dentro N · ↘ Abaixo N · = Sem redução N · ▲ Retrocesso N · — Sem meta N).
- `l2` (geral, sem comparar): "Geral (sem comparar): **Avanço parcial**" (sem `%`).
- Realocar verticalmente as 3 linhas restantes dentro do `impH = 0.86` para ficarem com respiro (ex.: 0.04 / 0.34 / 0.62).

### 2) Slide dos gráficos BA×PPI — corrigir cores das linhas projetadas
Arquivo: `public/Ferramenta_ER.html`, bloco `~3058-3093`.

Hoje o array `chartColors` é montado com 2 (BA, PPI) + 2 cores tracejadas (Meta BA, Meta PPI), mas a chamada `slG.addChart(...)` reescreve `chartColors: ['5C2E0A','E05A1A']` (linha 3074), descartando as cores das projeções e fazendo as linhas de meta reciclarem a paleta.

Correções:
- Substituir o literal por `chartColors` (a variável já montada acima).
- Aplicar estilo tracejado **só** nas séries `Meta BA` / `Meta PPI` via `lineDataSymbol:'none'` específico por série, usando `chartColors[2/3]` em tom mais claro (mantendo `A88670` / `F2A87A`).
- Para a linha projetada usar `lineSize: 2`, `lineDataSymbol:'circle'`, mas `lineDash: 'dash'` via `lineSize`/opções globais do pptxgenjs (aplicaremos `chartColorsOpacity` por série não é possível; usaremos `dataLabelFormatCode` igual). Como o pptxgenjs não suporta dash por série em `LINE`, vamos no mínimo: garantir cores corretas e nome diferenciado "(projeção)" na legenda — já presente.
- Comportamento gap-alvo isolado: continua aparecendo o rodapé 🎯 (já feito).

### 3) Slide "Eixo 05 — Gaps BA×PPI" — coluna Status com dois olhares
Arquivo: `public/Ferramenta_ER.html`, bloco `~3238-3245`.

Hoje a célula da coluna **Status** mostra só `calc.status` (Avançando/Estagnado/Retrocesso/Insuficiente).

Mudança:
- Renderizar duas mini-badges empilhadas dentro da célula:
  - **Geral**: `Avançando | Estagnado | Retrocesso | Insuficiente` (cor por `statusEvolucaoCor`).
  - **vs Meta**: `Acima | Dentro | Abaixo | Sem redução | Retrocesso | Sem meta` (cor por `statusMetaCor`).
- Exigirá que `calcItemEixo6` exponha o `statusVsMeta` no objeto retornado (já calculado via `_calcStatusVsMeta` em outro ponto; vamos garantir que esteja em `it.calc.statusVsMeta`).
- Ajustar `linhasH2` para reservar altura suficiente (badge dupla + Volátil) — subir mínimo de `0.78` para `~0.95`.
- Cabeçalho da coluna passa de "Status" para "Status — Geral / vs Meta".

### 4) Slide Síntese — palavras de status cortadas
Arquivo: `public/Ferramenta_ER.html`, bloco `~2682-2697`.

Causa: células com `wrap:false` e altura pequena cortam rótulos longos como "Em consolidação", "Em desenvolvimento", "Sem lacunas relevantes", "Sem redução".

Correções:
- Trocar `wrap:false` por `wrap:true` em todas as linhas do bloco `dimsSint` (linhas 2693, 2694, 2696, 2689).
- Reduzir a fonte do nível para 9-10pt (linhas 2693/2696) e manter `shrinkText:true`.
- Reduzir levemente o padding interno do roundRect (`+0.04` em vez de `+0.06`) para dar +0.04 de largura útil.
- No caso do Índice (`isImpacto`): aumentar `h1` para 0.70 (de 0.62) e usar fonte 9pt no rótulo vs-Meta; "Geral: …" sem `%` (conforme item 1).

### Resumo das alterações de arquivos
- `public/Ferramenta_ER.html`:
  - Faixa de Índice no slide da coalizão (limpeza + sem %).
  - Bloco gráficos BA×PPI (cores corretas das linhas projetadas).
  - Tabela de gaps BA×PPI (coluna Status dupla + altura).
  - Slide Síntese (wrap/fonte/altura para evitar corte).
  - Expor `statusVsMeta` em `calcItemEixo6` se ainda não estiver disponível por item.

Sem mudanças em `dados_teste_variabilidade.json` (os dados atuais já cobrem todos os status).
