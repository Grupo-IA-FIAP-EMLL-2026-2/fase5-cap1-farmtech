# Contrato de dados e protocolo de avaliação

**FarmTech Solutions — Fase 5, Entrega 1**
Versão 1.1 — 06/09/2026 (revisada após revisão independente)

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

Há, portanto, uma diferença entre a **unidade declarada no enunciado** e a **unidade efetiva do
arquivo, que não foi verificada**. Não sabemos qual é a unidade real e **não adotamos nenhuma
hipótese** — supor uma converteria um palpite em número de relatório.

### Regras enquanto a dúvida não for resolvida

1. **Não converter nada.** Os valores originais são usados como estão, em todas as análises.
2. **Reportar MAE e RMSE como "unidades de rendimento do arquivo"**, nunca como "t/ha".
3. **Incluir MAPE (erro percentual)** ao lado, por não depender da escala e ser comparável entre
   culturas de magnitudes diferentes.
4. **Não afirmar** que os 39 cenários correspondem a 39 anos, safras ou regiões. A base não tem
   coluna de data, local ou identificação de fazenda.

### O que muda quando a unidade for confirmada

Depende do que a confirmação exigir — **não é verdade que apenas o texto mude**:

| Situação | Efeito |
| --- | --- |
| A unidade é apenas **renomeada** (os números já estão certos) | Muda só o texto. |
| Os valores precisam ser **convertidos por um fator constante** | **MAE e RMSE mudam** na mesma proporção, e as escalas dos eixos dos gráficos também. **R² e MAPE não mudam**, por serem invariantes a uma multiplicação consistente de observados e previstos. A ordem entre as alternativas se mantém. |
| A correção altera os **dados usados no treino** | É preciso revisar transformações e parâmetros sensíveis à escala (SVR e a variante com log do alvo) e reexecutar a comparação. |

> **Pendência:** o coordenador deve confirmar as unidades com a FIAP. Até lá, a pendência permanece
> aberta e nenhuma unidade é presumida.

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

Isso significa que a base tem **156 linhas mas descreve só 39 cenários climáticos distintos**.

⚠️ Note a formulação: **cenários distintos**, não "observações independentes". Sem data, local ou
origem, não é possível verificar se esses 39 cenários são estatisticamente independentes entre si —
podem, por exemplo, vir de anos consecutivos da mesma região.

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

## 7. Validação cruzada **aninhada**

```python
from sklearn.model_selection import GroupKFold

# Particoes EXTERNAS - as mesmas para todas as alternativas
validacao_cruzada = GroupKFold(n_splits=5)
PARTICOES = list(validacao_cruzada.split(
    desenvolvimento, desenvolvimento[ALVO], groups=desenvolvimento["cenario"]
))
```

**Regra 4 — as partições externas são calculadas apenas sobre o desenvolvimento, e todas as
alternativas usam exatamente as mesmas.** `GroupKFold` é determinístico e não precisa de semente.

### Partições externas obtidas

| Partição | Linhas treino | Linhas validação | Cenários treino | Cenários validação |
| --- | --- | --- | --- | --- |
| 0 | 96 | 28 | 24 | 7 |
| 1 | 100 | 24 | 25 | 6 |
| 2 | 100 | 24 | 25 | 6 |
| 3 | 100 | 24 | 25 | 6 |
| 4 | 100 | 24 | 25 | 6 |

### Regra 4b — a busca de hiperparâmetros roda DENTRO do treino de cada partição

Se o mesmo conjunto escolhe os hiperparâmetros e depois mede o resultado dessa escolha, a métrica
reportada incorpora o ganho da própria seleção e sai otimista. O desenho correto é aninhado:

```
para cada uma das 5 partições EXTERNAS:
    ├── treino externo  ──► GridSearchCV com GroupKFold de 3 partições INTERNAS
    │                        (formadas só com os cenários do treino externo)
    └── validação externa ──► previsão, nunca vista pela busca
```

```python
N_PARTICOES_INTERNAS = 3

for indices_treino, indices_validacao in PARTICOES:
    X_treino, y_treino = X_dev.iloc[indices_treino], y_dev[indices_treino]
    grupos_treino = grupos_dev[indices_treino]

    particoes_internas = list(GroupKFold(n_splits=N_PARTICOES_INTERNAS).split(
        X_treino, y_treino, groups=grupos_treino))          # agrupado tambem no nivel interno

    busca = GridSearchCV(modelo, grade, cv=particoes_internas,
                         scoring="neg_mean_absolute_error", n_jobs=N_JOBS).fit(X_treino, y_treino)
    previsto[indices_validacao] = busca.best_estimator_.predict(X_dev.iloc[indices_validacao])
```

**Isso importa na prática.** Quando a busca usava as partições externas, a árvore de decisão
aparecia com MAE 4.340, abaixo da referência; com a busca aninhada, seu MAE é 4.800, **acima** da
referência. A ordem entre alternativas mudou.

Cada partição externa pode escolher hiperparâmetros diferentes — isso é esperado, e a variação entre
elas é um indicador de estabilidade que o notebook reporta.

**O que o aninhamento não resolve:** escolher a alternativa vencedora olhando estas mesmas métricas
continua sendo uma seleção. A confirmação independente depende do teste reservado (seção 10).

**Regra 4c — um único nível de paralelismo.** `N_JOBS = 1` no notebook inteiro. Com 124 linhas o
custo é baixo, e a execução sequencial evita o encadeamento de processos do joblib/loky, que no
Windows deixa rastros de encerramento no terminal.

Documentação:
[GroupKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupKFold.html) ·
[validação aninhada](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html)

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

Numa decomposição descritiva da variância no desenvolvimento, a diferença entre as médias das
culturas responde por **98,8%** da soma de quadrados do rendimento — uma decomposição estatística
nesta amostra, não uma medida de causa. Na prática, uma referência que apenas prevê a média da
cultura, ignorando o clima, obtém **R² = 0,987**. Portanto **um R² global alto não é suficiente,
isoladamente**, para demonstrar que um modelo aprendeu algo além de distinguir quatro culturas de
escalas muito diferentes.

**Comparações obrigatórias:**

1. Contra `DummyRegressor(strategy="mean")` — piso absoluto.
2. Contra a **referência da média da cultura** — a comparação que realmente importa.
3. **MAE, R² e MAPE calculados dentro de cada cultura**, a partir das previsões de validação.

### Regra 8b — declarar a métrica de "supera a referência" e calculá-la por código

⚠️ **Um R² negativo NÃO significa perder para a referência.** O zero do R² corresponde à média da
cultura **no conjunto avaliado**, enquanto a referência do projeto usa médias aprendidas **no treino
de cada partição**. Por isso a própria referência tem R² levemente negativo (entre −0,02 e −0,15).

Exemplo real desta base: a regressão linear tem R² = −0,013 no dendê e **supera** a referência, cujo
R² ali é −0,039.

Regras práticas:

- Comparar cada alternativa **diretamente com a referência**, na mesma cultura e na mesma métrica —
  nunca contra o zero.
- **Declarar a métrica usada.** No notebook, "supera a referência" significa **MAE menor dentro da
  cultura**; o julgamento pelo R² é exibido ao lado.
- As duas métricas **podem discordar**: a regressão linear com log do alvo tem MAE menor que o da
  referência nas quatro culturas, mas R² maior em apenas duas. Quando discordam, a conclusão depende
  da métrica, e isso precisa estar dito.
- **Gerar essas indicações por código**, comparando com a linha da referência, em vez de escrever
  tabelas à mão — foi assim que uma versão anterior deste projeto afirmou incorretamente que a
  regressão linear só superava a referência no arroz.

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

**Regra 9 — feche as escolhas antes de olhar o teste, e não volte atrás depois de olhar.**

Nesta etapa o teste **não foi tocado**. Nenhuma decisão (divisão, número de clusters,
hiperparâmetros, transformação do alvo) usou o conjunto reservado.

### O que invalida o teste, e o que não invalida

| Ação | Invalida? |
| --- | --- |
| Reexecutar o notebook e obter os mesmos números, para conferir reprodutibilidade | **Não.** O procedimento é determinístico. |
| O professor executar o notebook inteiro na correção | **Não.** A execução completa precisa continuar possível na entrega. |
| Olhar o resultado do teste e **então** trocar de algoritmo, ajustar hiperparâmetros ou escolher outra transformação | **Sim.** A métrica deixa de ser independente. |
| Avaliar várias alternativas no teste e reportar a melhor | **Sim.** É seleção pelo teste. |

Ou seja: a regra **não** é "execute uma vez e nunca mais". É **não usar o resultado do teste para
escolher ou ajustar alternativas**.

### Regra 9b — a alternativa avaliada é parametrizada

A seção 9 do notebook **não fixa nenhum modelo no código da avaliação**. Ela lê:

```python
ALTERNATIVA_FINAL = None      # a frente ML registra aqui a alternativa escolhida
EXECUTAR_TESTE_FINAL = False  # trava de protocolo
```

`CONFIGURACOES_FINAIS` reúne as cinco alternativas do catálogo mais as três variantes com log do
alvo, cada uma com pipeline, transformações e grade registrados. Se `ALTERNATIVA_FINAL` continuar
`None`, a avaliação falha com mensagem explícita em vez de avaliar um modelo qualquer — o notebook
verifica isso a cada execução.

**Condições para destravar:**

1. A frente ML fecha a escolha com base **apenas** na seção 8 e registra `ALTERNATIVA_FINAL`.
2. O grupo confirma que nenhuma outra alternativa será comparada depois.
3. `EXECUTAR_TESTE_FINAL = True` e o notebook é executado.

Se, depois de ver o resultado, o grupo decidir mudar de modelo, a métrica final deixa de ser
independente e isso precisa ser declarado no relatório.

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

## 12. Resumo das regras

1. O CSV original é imutável; confira o hash.
2. Uma única semente, `SEED = 42`, no projeto inteiro.
3. `cenario` agrupa a divisão; nunca é entrada de modelo.
4. As partições externas vêm só do desenvolvimento e são as mesmas para todos; a busca de
   hiperparâmetros roda **dentro do treino de cada partição** (validação aninhada).
5. Toda transformação de entrada vive dentro do pipeline.
6. Toda transformação do alvo é aprendida só no treino.
7. Uma única função de métricas para todas as alternativas.
8. Sempre reportar métricas **por cultura**, comparar com a referência da média da cultura (nunca
   com o zero do R²) e **declarar a métrica** usada para dizer "supera a referência".
9. Feche as escolhas **antes** de olhar o teste reservado e não volte atrás depois; reexecutar o
   notebook para conferir reprodutibilidade não invalida nada.
10. A alternativa avaliada no teste é **parametrizada** (`ALTERNATIVA_FINAL`), nunca fixada no código.
