## Objetivo

Restaurar integralmente o registro da coalizão **Alfabetização** no arquivo `public/dados_teste_variabilidade.json` para que fique idêntico ao que estava no JSON original enviado pelo usuário (`ER_dados_2026-06-24.json`), mantendo todas as demais 6 coalizões com as variações que já produzimos.

## Verificação feita

- Ambos arquivos são listas com 7 coalizões na mesma ordem; índice `[1]` é Alfabetização nos dois.
- Comparação por hash confirma divergência (orig 6616 chars vs atual 6514 chars) — ou seja, a coalizão foi alterada na rodada anterior, contrariando a instrução de preservar.

## Passos

1. Carregar `/mnt/user-uploads/ER_dados_2026-06-24.json` (original) e `public/dados_teste_variabilidade.json` (atual).
2. Substituir `atual[1]` pelo objeto completo `original[1]` (Alfabetização), sem tocar nos índices 0, 2, 3, 4, 5, 6.
3. Salvar de volta em `public/dados_teste_variabilidade.json` preservando formatação (UTF-8, `ensure_ascii=false`, indentação compatível com o arquivo).
4. Validar:
   - hash de `novo[1]` == hash de `original[1]`
   - hashes de `novo[i]` (i≠1) == hashes do `atual[i]` anterior à substituição
5. Copiar o JSON resultante para `/mnt/documents/dados_teste_variabilidade.json` e disponibilizar para download.

## Fora de escopo

Nenhuma alteração em `Ferramenta_ER.html` ou em qualquer outra coalizão.
