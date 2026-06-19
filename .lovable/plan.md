## Reformulação do Eixo 2 — Indicadores da Visão de Sucesso

### 1. Mudança conceitual

Hoje o Eixo 2 mede a qualidade de indicadores específicos de ER. Passa a medir a **maturidade operacional da coalizão em ER** a partir dos indicadores da própria visão de sucesso que **admitem recorte racial**, conectando-se diretamente ao Eixo 6 (gaps BA×PPI).

- **Eixo 2 = catálogo** de todos os indicadores da visão de sucesso com possibilidade de recorte racial (gerais racializáveis + já racializados).
- **Eixo 6 = aplicação** do recorte racial sobre indicadores escolhidos do Eixo 2.
- Lacuna entre os dois = "falha de olhar racial" e **penaliza a nota do Eixo 2**.

### 2. Novo campo no Eixo 2: Tipo do indicador

Cada indicador passa a ter um campo **Tipo**:

| Tipo | Exemplo | Esperado no Eixo 6 | Cobra penalidade? |
|---|---|---|---|
| **Geral racializável** | Crianças alfabetizadas na idade certa | Sim, para apurar gap BA×PPI | **Sim** |
| **Específico já racializado** | % diretores PPI | Sim, para monitorar evolução do gap | **Não** (já é um recorte racial por construção) |

Os "Específicos já racializados" entram no monitoramento do Eixo 6, mas **não contam no denominador da cobertura**, pois não há "lacuna de olhar racial" a cobrar.

### 3. Campos atuais (mantidos, recontextualizados)

Os três campos hoje existentes seguem como **dimensões de qualidade do indicador na visão de sucesso da coalizão**, com rótulos/tooltips reescritos para deixar claro que medem **a robustez do indicador**, não o recorte racial em si:

- **Status / Qualidade do dado** (Tem / Tem parcialmente / Não tem)
- **Resultados nos Estados** (S / P / N)
- **Resultado Nacional** (S / P / N)

O recorte racial vira **dimensão separada**, vinda automaticamente do Eixo 6.

### 4. Nova fórmula do Eixo 2

```text
nota_qualidade = média atual (Q + RSN² + RN)/3 × robustez por nº indicadores
cobertura_racial = (indicadores tipo "Geral racializável" referenciados em ≥1 item do Eixo 6)
                  ÷ (total de indicadores tipo "Geral racializável")
nota_final_eixo2 = nota_qualidade × cobertura_racial
```

Regras de borda:
- Coalizão sem nenhum "Geral racializável" → cobertura = 1 (não penaliza; todos seriam já-racializados).
- Coalizão com "Geral racializável" e zero referência no Eixo 6 → cobertura = 0 → eixo vai a Incipiente.
- Faixas de nível (Incipiente / Em desenvolvimento / Avançado / Consolidado) continuam pelas mesmas notas de corte (35 / 60 / 80).

### 5. Vínculo Eixo 6 → Eixo 2

Um indicador é considerado **com recorte racial trabalhado** quando existe pelo menos um item do Eixo 6 com `indicadorRef` apontando para ele E com no mínimo 1 ponto de série temporal preenchido (BA e PPI). Sem ponto preenchido = referência vazia = ainda em lacuna.

### 6. Sinalização visual

**No formulário do Eixo 2** — em cada card de indicador "Geral racializável" sem contrapartida válida no Eixo 6:
- Badge vermelho ⚠ **"Sem recorte racial trabalhado"** no topo do card.
- Tooltip explicando que isso reduz a nota do eixo e direcionando ao Eixo 6.
- Resumo no topo do eixo: "Cobertura racial: X/Y indicadores (Z%)".

**No PPT do Eixo 2**:
- Subtítulo do slide ganha a linha "Cobertura racial: X/Y · Penalidade aplicada: -W%".
- Coluna nova ou ícone na tabela existente marcando os indicadores em lacuna.
- Bloco textual ao final listando nominalmente os "Indicadores em lacuna de recorte racial" (apenas se houver ≥1).

### 7. Migração de dados existentes

Coalizões já cadastradas: todos os indicadores existentes recebem **Tipo = "Geral racializável"** por padrão (escolha conservadora, que cobra contrapartida). A coalizão pode reclassificar manualmente como "Específico já racializado" caso aplicável. Nenhuma perda de dado.

### 8. Documento de lógica

Atualizar a seção do Eixo 2 explicando:
- Novo escopo (indicadores da visão de sucesso com possibilidade de recorte racial).
- Definição dos dois Tipos.
- Fórmula `nota_qualidade × cobertura_racial` com exemplos numéricos.
- Como o Eixo 6 sinaliza "recorte trabalhado".
- Justificativa: por que a coalizão pode ter bons indicadores e ainda assim ficar Incipiente no Eixo 2 (= não olha para os gaps).

### 9. Detalhes técnicos (arquivo único `public/Ferramenta_ER.html`)

- `defaultIndicador` e `normalizeIndicador`: adicionar campo `tipo` com default `'geral'`.
- `calcEixoIndicadores(indicadores, eixo6)`: receber também a lista do Eixo 6 e aplicar o multiplicador de cobertura.
- Todos os callers (`refreshImpacto`, dashboard, PPT, DOCX) passam `c.eixo6` adjacente.
- Render do card do Eixo 2: novo `<select>` para Tipo + badge condicional + bloco de resumo no topo do eixo.
- PPT (`addEixoIndicadoresSlide` equivalente): subtítulo, marcação na tabela e bloco "Indicadores em lacuna".
- DOCX: mesmas adições resumidas.

### 10. Fora de escopo

- Alterar Eixos 1, 3, 4, 5.
- Mexer na fórmula interna do Eixo 6 (continua como ficou após a última refatoração).
- Recalcular automaticamente indicadores antigos como "Específicos já racializados" — fica como ação manual da coalizão.
