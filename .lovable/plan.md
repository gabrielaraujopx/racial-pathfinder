## Objetivo
Resolver os 2 alertas de segurança de severidade alta nas dependências.

## Vulnerabilidades
1. **react-router-dom 6.30.1** — XSS via Open Redirects (GHSA-2w69-qvjg-hvjx)
2. **recharts 2.15.4** — Code Injection via lodash `_.template` (GHSA-r5fr-rjxr-66jc)

## Ações
1. Atualizar `react-router-dom` para a versão mais recente da linha 6.x que corrige o advisory (≥ 6.31 / 7.x conforme disponível) via `bun add react-router-dom@latest`.
2. Atualizar `recharts` para a versão mais recente (≥ 2.15.5 / 3.x) que remove a dependência vulnerável de lodash, via `bun add recharts@latest`.
3. Rodar `bun install` para regenerar o lockfile.
4. Verificar build (typecheck/Vite) e abrir a app para confirmar que rotas e gráficos (Recharts) continuam funcionando.
5. Marcar os findings como `fixed` no scanner.

## Escopo
Apenas `package.json` / `bun.lockb`. Sem alterações em `public/Ferramenta_ER.html`, JSON de dados ou documentos em `/mnt/documents/`.

## Observação
Se o major bump do `recharts` (2.x → 3.x) introduzir breaking changes nos componentes usados em `src/`, fico na maior 2.x corrigida. Idem para `react-router-dom`.
