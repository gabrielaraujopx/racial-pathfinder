## Diagnóstico (validado no PPT exportado)

Gerei o PPT com o JSON `ER_dados_2026-06-24.json` e inspecionei os slides:

- **Pré-escola parte 1/2**: 4 indicadores, ~1.4" de coluna vazia abaixo.
- **Pré-escola parte 2/2**: 4 indicadores (5-8), mesma folga.
- **Alfabetização parte 1/2**: 2 indicadores, espaço de sobra suficiente para o 3º.
- **Alfabetização parte 2/2**: existe só para abrigar o 3º indicador.

A lógica de empacotamento (linhas 2761-2778) está correta — só pagina quando `projH > avail`. O problema é o **estimador de altura `estH`** (linhas 2627-2631):

```
chPerLine = floor(wIn * 96 * 0.55 / (fs * 0.6))
```

Para a coluna Indicadores (`_indTWpre ≈ 1.86"`, fs 11pt) isso dá ~14 caracteres por linha. O render real do PowerPoint cabe ~30-32 caracteres. O resultado é que **cada item é estimado com o dobro de linhas**, a coluna parece cheia no papel mesmo quando está pela metade, e a paginação dispara cedo.

## Intervenção

Substituir a heurística por **medição real via canvas** em `public/Ferramenta_ER.html`, sem alterar layout, fontes, cores, gaps, badges ou qualquer outro elemento visual.

### Mudança técnica

Reescrever o `estH(txt, wIn, fs)` (≈15 linhas):

1. Criar um `_measureCtx = document.createElement('canvas').getContext('2d')` uma única vez por export.
2. `ctx.font = fs + 'pt Calibri'` (família usada nos slides).
3. Simular wrap: dividir `txt` em palavras, somar `ctx.measureText(word + ' ').width` até estourar `wIn * 96` pixels → incrementa contador de linhas; respeitar quebras explícitas `\n` (relevante para o eixo Lacunas com `→ recomendação`).
4. Retornar `lines * fs * 0.0140 + 0.02` (fator de altura de linha mais próximo do Calibri no PPT real; o atual `0.0155` também superestima ~10%).
5. Fallback defensivo se `document` indisponível: fórmula antiga com fator `0.85` em vez de `0.55`.

### Efeito esperado (validável)

- Pré-escola: 6-7 indicadores em uma página (talvez única); pagina só se restar 1-2.
- Alfabetização: 3 indicadores numa única página, sem paginação.
- Coalizões com itens realmente longos continuam paginando, agora com base em altura real.

### Validação

Após aplicar, executar novamente: gerar PPT com este mesmo JSON via Playwright, converter para imagens e conferir os slides de Pré-escola e Alfabetização. Critérios:

1. Pré-escola não fica com >1" de coluna vazia em nenhuma parte.
2. Alfabetização cabe em 1 slide.
3. Nenhuma sobreposição entre itens em qualquer coalizão.

### Arquivos

- `public/Ferramenta_ER.html` (somente o bloco do `estH` e a criação do helper de canvas).
