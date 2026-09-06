# Contrato de dados e protocolo de avaliação

**FarmTech Solutions — Fase 5, Entrega 1**
Versão 1.0 — 06/09/2026

Este documento fixa as regras que **todos os integrantes devem seguir** ao trabalhar com
`data/crop_yield.csv`. O mesmo protocolo está implementado nas seções 3 e 4 do notebook
`notebooks/farmtech_desenvolvimento.ipynb`, e o resultado da divisão fica registrado em
`docs/protocolo_divisao.json`.

O objetivo é simples: **qualquer pessoa do grupo que seguir este contrato obtém exatamente a mesma
divisão de dados e as mesmas partições de validação**, e portanto números comparáveis.

---

## 1. Arquivo de origem

| Item | Valor |
| --- | --- |
| Caminho | `data/crop_yield.csv` (relativo à raiz do projeto) |
| Tamanho | 7.606 bytes |
| SHA-256 | `07B3335F497E08E705B5835EE334426AC16CB24732C1BFAA994254D39F771B1B` |
| Dimensões | 156 linhas × 6 colunas |
| Codificação | UTF-8, separador `,`, sem quebra de linha final |

**Regra 1 — o arquivo original é imutável.** Nenhum script pode sobrescrevê-lo. Toda limpeza,
renomeação ou transformação acontece em memória, sobre uma cópia. O notebook verifica o hash no
início e no fim da execução.

Se o hash não bater, pare: o arquivo em uso não é o mesmo e nenhum resultado será comparável.

```python
import hashlib
from pathlib import Path
print(hashlib.sha256(Path("data/crop_yield.csv").read_bytes()).hexdigest().upper())
```

---

## 2. Mapeamento de nomes

| Nome original no CSV | Nome interno | Tipo | Papel | Unidade declarada |
| --- | --- | --- | --- | --- |
| `Crop` | `cultura` | categórica (4 níveis) | entrada | — |
| `Precipitation (mm day-1)` | `precipitacao` | numérica | entrada | mm/dia ⚠️ |
| `Specific Humidity at 2 Meters (g/kg)` | `umidade_especifica` | numérica | entrada | g/kg |
| `Relative Humidity at 2 Meters (%)` | `umidade_relativa` | numérica | entrada | % |
| `Temperature at 2 Meters (C)` | `temperatura` | numérica | entrada | °C |
| `Yield` | `rendimento` | numérica | **alvo** | t/ha ⚠️ |

```python
RENOMEAR = {
    "Crop": "cultura",
    "Precipitation (mm day-1)": "precipitacao",
    "Specific Humidity at 2 Meters (g/kg)": "umidade_especifica",
    "Relative Humidity at 2 Meters (%)": "umidade_relativa",
    "Temperature at 2 Meters (C)": "temperatura",
    "Yield": "rendimento",
}
VAR_CLIMA = ["precipitacao", "umidade_especifica", "umidade_relativa", "temperatura"]
VAR_CATEGORICA = ["cultura"]
ALVO = "rendimento"
```

**Culturas presentes** (39 registros cada): `Cocoa, beans`, `Oil palm fruit`, `Rice, paddy`,
`Rubber, natural`.

---

## 3. ⚠️ Dúvida de unidades — pendência aberta

Este é o ponto mais importante do contrato e precisa ser lido por todos.

### O problema

| Coluna | Unidade declarada no enunciado | Faixa real no CSV | Compatível? |
| --- | --- | --- | --- |
| `Yield` | toneladas por hectare | 5.249 a 203.399 | **Não.** 200 mil t/ha é agronomicamente impossível |
| `Precipitation (mm day-1)` | milímetros por dia | 1.934 a 3.086 | **Não.** 3.000 mm/dia é impossível; a faixa corresponde a um acumulado anual |

### Hipótese (não confirmada)

`Yield` estaria em **hectogramas por hectare (hg/ha)**, unidade usada em bases do tipo FAOSTAT.
Nesse caso 203.399 hg/ha ≈ 20,3 t/ha para o dendê, valor plausível. A precipitação seria um
**acumulado anual** em mm.

**Isto é uma leitura nossa dos números, não uma informação verificada.** Está registrada aqui como
hipótese justamente para não ser confundida com fato.

### Regras enquanto a dúvida não for resolvida

1. **Não converter nada.** Os valores originais são usados como estão, em todas as análises.
2. **Reportar MAE e RMSE como "unidades de rendimento do arquivo"**, nunca como "t/ha".
3. **Incluir MAPE (erro percentual)** ao lado, por ser independente da unidade e comparável entre
   culturas de magnitudes diferentes.
4. **Não afirmar** que os 39 cenários correspondem a 39 anos, safras ou regiões. A base não tem
   coluna de data, local ou identificação de fazenda.

> **Pendência:** o coordenador deve confirmar as unidades com a FIAP. Quando confirmadas, só o
> **texto** das métricas muda — nenhum resultado numérico é afetado.

---

## 4. Semente

```python
SEED = 42
```

**Regra 2 — uma única semente no projeto inteiro.** Todo procedimento aleatório que aceite semente
usa `SEED`. Não redefinir por seção, não testar várias sementes para escolher a que dá números
melhores.

---

## 5. Identificador de cenário climático

### Por que ele existe

As quatro variáveis climáticas formam apenas **39 combinações distintas**, e **cada combinação
aparece exatamente quatro vezes**, uma por cultura. Verificado:

```
Cenários climáticos distintos : 39
Linhas por cenário            : [4]
Culturas por cenário          : [4]
```

Isso significa que a base tem **156 linhas mas só 39 observações climáticas independentes**.

**Essas quatro linhas não são duplicatas.** Elas têm rendimentos diferentes porque descrevem
culturas diferentes sob o mesmo clima. **Não remover.**

### Como é calculado

```python
import hashlib

def identificador_cenario(linha):
    """Identificador determinístico das quatro variáveis climáticas.

    NÃO usa Crop nem Yield. Serve para AGRUPAR na divisão dos dados,
    nunca como entrada de modelo.
    """
    assinatura = "|".join(format(linha[v], ".6f") for v in VAR_CLIMA)
    return hashlib.sha256(assinatura.encode("utf-8")).hexdigest()[:12]

dados["cenario"] = dados.apply(identificador_cenario, axis=1)
```

**Regra 3 — `cenario` nunca é entrada de modelo.** Ele existe só para dividir os dados. Usá-lo como
variável preditora seria memorizar identificadores.

O arredondamento fixo em 6 casas garante o mesmo identificador em qualquer máquina.

---

## 6. Divisão treino/teste

```python
from sklearn.model_selection import GroupShuffleSplit

separador = GroupShuffleSplit(n_splits=1, test_size=0.20, random_state=SEED)
idx_desenvolvimento, idx_teste = next(separador.split(dados, groups=dados["cenario"]))

desenvolvimento = dados.iloc[idx_desenvolvimento].reset_index(drop=True)
teste = dados.iloc[idx_teste].reset_index(drop=True)
```

### Resultado esperado (confirmado na execução)

| Conjunto | Linhas | Cenários | Linhas por cultura |
| --- | --- | --- | --- |
| Desenvolvimento | **124** | **31** | 31 de cada |
| Teste reservado | **32** | **8** | 8 de cada |

Se você obtiver números diferentes, algo saiu do contrato — confira o hash do CSV e a semente.

### Por que agrupado e não aleatório

Uma divisão aleatória comum colocaria linhas do mesmo cenário climático nos dois lados. O modelo
veria no treino exatamente o clima que encontraria no teste, mudando apenas a cultura, e a métrica
mediria memorização, não generalização.

Mantendo cada cenário inteiro de um lado só, a pergunta que o teste responde passa a ser a correta:
**o modelo acerta em condições climáticas que nunca viu?**

Documentação: [GroupShuffleSplit](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupShuffleSplit.html)

---

## 7. Validação cruzada

```python
from sklearn.model_selection import GroupKFold

validacao_cruzada = GroupKFold(n_splits=5)
PARTICOES = list(validacao_cruzada.split(
    desenvolvimento, desenvolvimento[ALVO], groups=desenvolvimento["cenario"]
))
```

**Regra 4 — as partições são calculadas apenas sobre o desenvolvimento, e todas as alternativas
usam exatamente as mesmas.** `GroupKFold` é determinístico e não precisa de semente.

### Partições obtidas

| Partição | Linhas treino | Linhas validação | Cenários treino | Cenários validação |
| --- | --- | --- | --- | --- |
| 0 | 96 | 28 | 24 | 7 |
| 1 | 100 | 24 | 25 | 6 |
| 2 | 100 | 24 | 25 | 6 |
| 3 | 100 | 24 | 25 | 6 |
| 4 | 100 | 24 | 25 | 6 |

Documentação: [GroupKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupKFold.html)

---

## 8. Regras de vazamento

**Regra 5 — toda transformação vive dentro do pipeline.** Imputação, codificação e escalonamento são
ajustados apenas com dados de treino, reajustados em cada partição.

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

pipeline = Pipeline([
    ("preparacao", ColumnTransformer([
        ("categorica", OneHotEncoder(handle_unknown="ignore"), VAR_CATEGORICA),
        ("numerica", StandardScaler(), VAR_CLIMA),
    ])),
    ("modelo", ...),
])
```

**Regra 6 — transformação do alvo também é aprendida só no treino.** Use
`TransformedTargetRegressor`, nunca escale `y` fora da partição:

```python
from sklearn.compose import TransformedTargetRegressor
# correto: o escalonador de y é reajustado em cada partição
modelo = TransformedTargetRegressor(regressor=SVR(), transformer=StandardScaler())
```

### Verificações obrigatórias (implementadas no notebook)

| # | Verificação | Situação |
| --- | --- | --- |
| a | Nenhum cenário compartilhado entre desenvolvimento e teste | ✅ 0 em comum |
| b | Nenhum cenário compartilhado entre treino e validação | ✅ `[0, 0, 0, 0, 0]` |
| c | Desenvolvimento + teste = 156 linhas | ✅ 124 + 32 |
| d | Cada linha validada exatamente uma vez | ✅ |
| e | As 4 culturas em toda partição de validação | ✅ |
| f | CSV original preservado (SHA-256) | ✅ |

---

## 9. Métricas

**Regra 7 — uma única função de métricas para todas as alternativas.**

```python
def calcular_metricas(observado, previsto):
    return {
        "MAE":    mean_absolute_error(observado, previsto),
        "RMSE":   float(np.sqrt(mean_squared_error(observado, previsto))),
        "R2":     r2_score(observado, previsto),
        "MAPE_%": float(np.mean(np.abs((observado - previsto) / observado)) * 100),
    }
```

### Regra 8 — sempre reportar as métricas por cultura, não só o total

Este é o ponto metodológico central da Entrega 1.

A cultura sozinha explica **98,8%** da variância do rendimento. Uma referência que apenas prevê a
média da cultura, ignorando completamente o clima, obtém **R² = 0,987**. Portanto **um R² global
alto não demonstra nada**: mostra apenas que o modelo distingue quatro culturas de escalas muito
diferentes.

**Comparações obrigatórias:**

1. Contra `DummyRegressor(strategy="mean")` — piso absoluto.
2. Contra a **referência da média da cultura** — a comparação que realmente importa.
3. **R², MAE e MAPE calculados dentro de cada cultura**, a partir das previsões de validação.

Interpretação do R² dentro da cultura:

| Valor | Significado |
| --- | --- |
| **> 0** | o modelo prevê melhor do que a média da própria cultura → **o clima acrescenta informação** |
| ≈ 0 | equivale a prever a média da cultura → o clima não acrescentou nada |
| < 0 | pior do que prever a média da cultura |

### Referência da média da cultura

```python
class MediaDaCultura(BaseEstimator, RegressorMixin):
    """Prevê a média da cultura, aprendida SOMENTE no treino de cada partição."""

    def fit(self, X, y):
        serie = pd.Series(np.asarray(y), index=np.asarray(X["cultura"]))
        self.medias_ = serie.groupby(level=0).mean().to_dict()
        self.media_global_ = float(np.mean(y))
        return self

    def predict(self, X):
        return np.array([self.medias_.get(c, self.media_global_) for c in X["cultura"]])
```

---

## 10. Uso do conjunto de teste

**Regra 9 — o teste reservado é usado uma única vez, no fechamento.**

Nesta etapa ele **não foi tocado**. Nenhuma decisão (divisão, número de clusters, hiperparâmetros,
transformação do alvo) usou o teste.

A seção 9 do notebook está implementada e travada por `EXECUTAR_TESTE_FINAL = False`. Ela roda sem
erro e imprime o estado pendente, sem avaliar nada.

**Condições para destravar:**

1. A frente ML fecha a escolha de algoritmo e hiperparâmetros com base **apenas** na seção 8.
2. O grupo confirma que nenhuma outra alternativa será testada.
3. `EXECUTAR_TESTE_FINAL = True` e o notebook é executado inteiro **uma vez**.

Se depois de ver o resultado do teste alguém quiser mudar o modelo, a métrica final deixa de ser
independente e precisa ser declarada como tal.

Cenários reservados (não usar em nenhuma análise até o fechamento):

```
154c73461399  1dccfe0180f0  4f37a2d386b6  b11b23c4a13e
bbac82ed07b9  d19ed8808b8c  dcbf3da4c6bb  f3dacef6642f
```

---

## 11. Versões do ambiente

Versões efetivamente usadas na execução de 06/09/2026 (ver `requirements.txt`):

| Pacote | Versão |
| --- | --- |
| Python | 3.12.10 |
| numpy | 2.3.3 |
| pandas | 2.3.2 |
| scikit-learn | 1.7.1 |
| matplotlib | 3.10.6 |
| seaborn | 0.13.2 |

Divergências de versão de `scikit-learn` podem alterar levemente resultados de `RandomForest` e
`SVR`. Se os números não baterem, confira a versão antes de investigar o código.

---

## 12. Resumo das nove regras

1. O CSV original é imutável; confira o hash.
2. Uma única semente, `SEED = 42`, no projeto inteiro.
3. `cenario` agrupa a divisão; nunca é entrada de modelo.
4. As partições vêm só do desenvolvimento e são as mesmas para todos.
5. Toda transformação de entrada vive dentro do pipeline.
6. Toda transformação do alvo é aprendida só no treino.
7. Uma única função de métricas para todas as alternativas.
8. Sempre reportar métricas **por cultura** e comparar com a referência da média da cultura.
9. O teste reservado é usado **uma vez**, no fechamento.
