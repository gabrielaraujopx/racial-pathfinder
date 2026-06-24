Ajustes pontuais no PPT gerado e revisão do Índice de Redução de Gaps em `public/Ferramenta_ER.html`.

## 1. Síntese: badge "Em desenvolvimento" cortando *(aprovado)*

Bloco do slide síntese (linhas ~2752–2779):
- `fontSize` base das células: 9 → 8.
- Só na célula da síntese, exibir "Em desenvolvimento" como **"Em desenv."** (maturidade e linha "Geral:" do índice).
- Margem lateral da pílula `0.04` → `0.02`; padding interno `0.05` → `0.03`.
- Manter `wrap:true` + `shrinkText:true`.

## 2. Síntese: reordenar e renomear Maturidade *(aprovado)*

Nova ordem em `dimsSint` (linhas ~2723–2750):
1. Índice de Redução de Gaps
2. Maturidade Operacional *(renomeada de "Maturidade Geral")*
3. Indicadores
4. Estratégias
5. Lacunas Identificadas

Apenas posição e `label`; cálculo intacto. Restante do app continua "Maturidade Geral".

## 3. Coluna "Lacunas Identificadas" → "LACUNAS" *(aprovado)*

Linha 2989, só no header do slide por coalizão.

## 4. Renomear níveis do Índice e aplicar teto/piso vs Meta

### 4a. Renomear níveis do Geral

| Antigo               | Novo                   |
|----------------------|------------------------|
| Reduzindo gaps       | **Redução substancial**|
| Avanço parcial       | **Avanço significativo**|
| Avanço incipiente    | **Avanço parcial**     |
| Estagnado            | Estagnado              |
| Sem avanço           | Sem avanço             |
| Gaps se abrindo      | Gaps se abrindo        |

Atualizar em todos os pontos que tratam desses strings:
- `calcEixo6` (linhas ~1407–1419): faixas de classificação por `pctBruto` passam a usar os novos nomes.
- Cores `eixoNivCor` (linhas ~1517–1523): mapear os novos nomes para as mesmas cores dos antigos.
- Render da síntese na tela (linha 2312) e `bg`/cor do slide síntese (linhas ~2882–2887): trocar strings.
- Qualquer outra referência aos antigos nomes em legendas/chips.

### 4b. Coerência Geral × vs Meta

Aplicar quando `nVsMetaValido > 0`:

| Nível vs Meta | Efeito no Geral                                |
|---------------|------------------------------------------------|
| Retrocesso    | **teto**: no máximo `Gaps se abrindo`          |
| Sem redução   | **teto**: no máximo `Estagnado`                |
| Abaixo        | **teto**: no máximo `Avanço parcial`           |
| Dentro        | **piso**: no mínimo `Avanço significativo`     |
| Acima         | **piso**: no mínimo `Redução substancial`      |

Lógica: vs Meta = `Dentro` significa cumprir uma meta mais ambiciosa que a tendência histórica, então o Geral não pode reportar status modesto; `Acima` força o topo. `Retrocesso`/`Sem redução`/`Abaixo` continuam impedindo que o Geral reporte avanço acima do equivalente. `pct` bruto não é alterado.

### Implementação em `calcEixo6` (linhas ~1396–1465)

1. Mover o cálculo de `nivelMeta` para antes da finalização do `nivel` Geral.
2. Substituir o bloco atual de teto por:

```js
const ordemGeral = ['Gaps se abrindo','Sem avanço','Estagnado',
                    'Avanço parcial','Avanço significativo','Redução substancial'];
const tetoPorMeta = {
  'Retrocesso':  'Gaps se abrindo',
  'Sem redução': 'Estagnado',
  'Abaixo':      'Avanço parcial',
};
const pisoPorMeta = {
  'Dentro': 'Avanço significativo',
  'Acima':  'Redução substancial',
};
if (nVsMetaValido > 0) {
  const teto = tetoPorMeta[nivelMeta];
  const piso = pisoPorMeta[nivelMeta];
  const idx  = ordemGeral.indexOf(nivel);
  if (teto && idx > ordemGeral.indexOf(teto)) nivel = teto;
  if (piso && idx >= 0 && idx < ordemGeral.indexOf(piso)) nivel = piso;
}
```

3. Faixas brutas por `pctBruto` (linhas 1407–1410) passam a:
```js
nivel = pctBruto >= 70 ? 'Redução substancial'
      : pctBruto >= 40 ? 'Avanço significativo'
      : pctBruto >= 20 ? 'Avanço parcial'
      :                  'Sem avanço';
```

## Validação

Gerar PPT com o JSON atual e checar:
- Síntese: nova ordem, "Maturidade Operacional", "Em desenv." cabendo na pílula.
- Slide por coalizão: header "LACUNAS".
- Coalizão com vs Meta = Retrocesso: Geral fica `Gaps se abrindo`.
- Coalizão com vs Meta = Dentro: Geral ≥ `Avanço significativo`.
- Coalizão com vs Meta = Acima: Geral = `Redução substancial`.
- Legendas/cores: novos nomes mantêm a mesma paleta dos antigos.

## Fora de escopo

Sem mudanças no JSON, sem mudanças no app React. Tudo em `public/Ferramenta_ER.html`.
