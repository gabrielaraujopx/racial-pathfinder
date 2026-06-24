## Objetivo
Refletir nos artefatos as últimas mudanças aprovadas (renomeações de níveis Geral, mapeamento teto/piso vs Meta, "Maturidade Operacional", "LACUNAS", "Em desenv."), sem mexer no Eixo 5 (Alfabetização) além de nomenclatura.

## 1. JSON de teste atualizado
- Base: versão mais recente em `/mnt/documents/ER_dados_teste_variabilidade_v6.json`.
- Varrer todo o arquivo e substituir, somente em campos textuais de status/nível Geral do Eixo 6 (Índice de Redução de Gaps) e demais ocorrências da escala Geral:
  - `Reduzindo gaps` → `Redução substancial`
  - `Avanço parcial` → `Avanço significativo`
  - `Avanço incipiente` → `Avanço parcial`
  - (mantém `Gaps se abrindo`, `Sem avanço`, `Estagnado`)
- Onde houver rótulo `Maturidade Geral` usado como nome da dimensão na síntese, trocar para `Maturidade Operacional` (somente nesse contexto — manter "Maturidade Geral" em chaves estruturais usadas pelo cálculo, se existirem).
- Recalcular/ajustar coerência teto/piso vs Meta apenas se o JSON armazenar nível Geral já classificado; valores brutos (respostas, %, contadores) permanecem intactos.
- **Não alterar** nenhum conteúdo do Eixo 5 (Alfabetização) além de eventual renomeação textual que se aplique à mesma escala Geral, se presente.
- Salvar como `/mnt/documents/ER_dados_teste_variabilidade_v7.json` e entregar via `<presentation-artifact>`.

## 2. Doc executivo (`Logica_Sistematizacao_Executivo.md` + `.docx`)
- Atualizar a seção da escala Geral do Índice de Redução de Gaps com os novos rótulos.
- Adicionar parágrafo curto explicando a regra de coerência:
  - `Retrocesso` → teto `Gaps se abrindo`
  - `Sem redução` → teto `Estagnado`
  - `Abaixo` → teto `Avanço parcial`
  - `Dentro` → piso `Avanço significativo`
  - `Acima` → piso `Redução substancial`
- Renomear "Maturidade Geral" → "Maturidade Operacional" apenas na descrição da síntese (manter "Maturidade Geral" onde representar a média global do diagnóstico, se houver essa distinção no doc).
- Mencionar ajustes visuais do PPT: "Em desenv." e coluna "LACUNAS".
- Regenerar `.docx` correspondente.

## 3. Doc detalhado (`Logica_Sistematizacao_Detalhado.md` + `.docx` mais recente)
- Mesmos ajustes do executivo, com mais profundidade:
  - Tabela completa da nova escala Geral com cores atualizadas (`Redução substancial` #2D7A4F, `Avanço significativo` #1A6B3C, `Avanço parcial` #C8860A).
  - Descrição algorítmica do bloco em `calcEixo6`: cálculo de `pctBruto` → nível bruto → aplicação de `tetoPorMeta` / `pisoPorMeta` quando `nVsMetaValido > 0` → `nivel` final.
  - Atualização da seção de exportação PPT: encurtamento "Em desenv." na síntese, reordenação (Índice de Redução de Gaps em primeiro, "Maturidade Operacional" como rótulo da síntese) e coluna "LACUNAS" no slide de coalizão.
- Regenerar `.docx` (próxima versão, ex.: `Sistematizacao_ER_Logica_v4.docx`).

## Entrega
Três `<presentation-artifact>` no fim: JSON v7, executivo `.docx`, detalhado `.docx` v4. Os `.md` ficam atualizados em `/mnt/documents/` como fontes.

## Escopo
Somente arquivos em `/mnt/documents/`. Nenhuma alteração em `public/Ferramenta_ER.html` ou no app React.
