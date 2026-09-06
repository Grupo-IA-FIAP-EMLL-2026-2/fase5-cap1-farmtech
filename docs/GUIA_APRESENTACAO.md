# Guia para apresentar o projeto ao grupo

**FarmTech Solutions — Fase 5** · Preparado em 06/09/2026 · Prazo: **08/09, 23:59**

Este guia serve para o coordenador conduzir a reunião de alinhamento: mostrar o que já existe,
explicar os resultados e apresentar as frentes disponíveis para que cada um escolha a sua.

**Tempo sugerido: 25 minutos** — 15 de apresentação, 10 de escolha das frentes.

---

## Antes da reunião

Deixe aberto:

1. `notebooks/farmtech_desenvolvimento.ipynb` (já executado, com as saídas)
2. Este guia
3. `docs/CONTINUIDADE_EQUIPE.md`

E rode uma vez para conferir que o ambiente está funcionando:

```powershell
.\.venv\Scripts\Activate.ps1
jupyter lab notebooks\farmtech_desenvolvimento.ipynb
```

---

# Parte 1 — O que já está pronto (5 min)

## Fale assim

> "Adiantei a parte de análise e a primeira comparação dos modelos. Tudo o que vou mostrar foi
> executado de verdade, com as saídas salvas — não é um plano, é código rodando. O que falta está
> marcado como pendente em todo lugar."

## Quadro honesto da situação

| Item | Situação | Evidência |
| --- | --- | --- |
| Ambiente Python e dependências | ✅ **testado** | `requirements.txt` com versões que rodaram |
| Contrato de dados e protocolo | ✅ **implementado e verificado** | 6 verificações automáticas passando |
| Análise exploratória | ✅ **executada** | 12 figuras geradas |
| Clusterização | ✅ **executada** | k escolhido por estabilidade |
| Outliers | ✅ **executada** | 2 critérios independentes |
| **Cinco regressores + 2 referências** | ✅ **executados** | `docs/resultados_modelos.csv` |
| Documentação para o grupo | ✅ **escrita** | 5 documentos |
| Teste final | ⏸️ **travado de propósito** | Seção 9, esperando decisão |
| Entrega 2 (AWS) | ❌ **não iniciada** | — |
| Ir Além 1 e 2 | ❌ **não iniciados** | — |
| Vídeos | ❌ **nenhum gravado** | — |

**Frase para não passar impressão errada:** *"A Entrega 1 está em primeira versão completa. A
Entrega 2 e os dois extras não têm uma linha escrita ainda."*

---

# Parte 2 — Os três achados que valem explicar (7 min)

Estes são os pontos que fazem o trabalho ter qualidade. Se o grupo entender só isto, já é suficiente.

## Achado 1 — A base tem 156 linhas, mas só 39 cenários climáticos

**Mostre:** seção 3.1 do notebook.

> "Descobri uma coisa que muda o desenho todo. As quatro variáveis de clima formam só 39
> combinações diferentes, e cada uma aparece exatamente 4 vezes — uma para cada cultura. Ou seja: a
> mesma condição de clima, medida em quatro culturas.
>
> Isso **não** é duplicata, não dá para apagar: os rendimentos são diferentes porque as culturas são
> diferentes. Mas significa que a gente tem 39 observações de clima independentes, não 156.
>
> A consequência prática: se dividíssemos as linhas ao acaso entre treino e teste, o modelo veria no
> treino exatamente o mesmo clima que encontraria no teste, mudando só a cultura. A nota do modelo
> ia ficar ótima e não ia querer dizer nada — ele estaria decorando, não generalizando.
>
> Por isso a divisão é **por cenário**: cada combinação de clima fica inteira de um lado só."

**Números:** 124 linhas / 31 cenários para desenvolver, 32 linhas / 8 cenários reservados.

## Achado 2 — A cultura explica 98,8% do rendimento

**Mostre:** Figura 1 (escala linear vs. log) e a saída do eta² na seção 5.2.

> "Olhem o gráfico da esquerda. O dendê é tão maior que as outras três culturas viram uma linha
> achatada embaixo. Um dendê rende umas vinte vezes mais que um cacau.
>
> Isso tem uma consequência que quase derrubou a análise: **saber só a cultura já explica 98,8% da
> variação do rendimento**. Eu criei um modelo bobo, que ignora o clima completamente e só chuta a
> média da cultura. Ele tira **R² de 0,987**.
>
> Então se a gente escrever no relatório 'nosso modelo tem R² de 0,99', isso é verdade e é inútil —
> significa só que ele aprendeu a diferenciar quatro culturas, coisa que a coluna já dava de graça.
>
> Por isso comparei tudo **contra esse modelo bobo** e olhei as métricas **dentro de cada cultura**.
> É aí que a pergunta de verdade aparece: o clima acrescenta alguma coisa?"

## Achado 3 — O clima importa, mas só para arroz e borracha, em sentidos opostos

**Mostre:** Figura 3 (painel b) e Figura 7.

> "E a resposta é: sim, mas só em duas das quatro culturas.
>
> O arroz rende mais quando está quente e o ar está úmido — correlação de +0,70 com umidade
> específica e +0,61 com temperatura. A borracha faz exatamente o contrário: −0,43 e −0,41.
> Cacau e dendê praticamente não respondem.
>
> E o mais interessante: cheguei nisso por **três caminhos independentes** e os três deram a mesma
> resposta — as correlações, os grupos de clima e o modelo. Quando três análises diferentes apontam
> para o mesmo lugar, dá bem mais confiança do que qualquer uma sozinha.
>
> Faz sentido agronomicamente: arroz irrigado gosta de calor e ar úmido, seringueira produz menos
> látex em condição muito quente e abafada. Mas isso é **associação, não causa** — não temos nada
> sobre solo, adubação ou manejo."

---

# Parte 3 — Resultados dos modelos (3 min)

**Mostre:** Figura 10.

| Alternativa | MAE | R² geral | R² dentro de cada cultura (Cacau/Dendê/Arroz/Borracha) |
| --- | --- | --- | --- |
| **3. Floresta Aleatória** | **3.551** | 0,991 | **+0,13 / +0,24 / +0,46 / +0,04** |
| 2. Árvore de Decisão | 4.340 | 0,985 | −0,18 / −0,26 / +0,18 / −0,51 |
| *Ref. média da cultura* | *4.772* | *0,987* | *referência* |
| 1. Regressão Linear | 5.025 | 0,987 | −2,60 / −0,01 / +0,10 / −2,71 |
| 4. KNN | 6.759 | 0,948 | −116 / −1,45 / −5,22 / −30,9 |
| 5. SVR (RBF) | 7.365 | 0,975 | −35,3 / −0,37 / −1,21 / −24,8 |

> "Olhem a coluna do R² geral: todos entre 0,95 e 0,99. Parece que está tudo ótimo.
>
> Agora a última coluna, que é o que interessa. Verde é 'melhor que chutar a média da cultura'.
> **Só a Floresta Aleatória consegue isso nas quatro culturas.** Três dos cinco algoritmos são
> **piores** do que o chute — mesmo com R² de 0,99.
>
> O KNN e o SVR quebram feio. Eu diagnostiquei o motivo: eles trabalham por distância, e como as
> culturas viram variáveis binárias padronizadas, eles acabam misturando vizinhos de culturas
> diferentes. Errar entre um cacau e um dendê custa dezenas de milhares de unidades.
>
> Já testei uma correção — transformar o alvo em log — e o ganho está medido no notebook. Quem pegar
> a frente de ML tem esse caminho pronto para seguir."

**Ponto de honestidade a mencionar:**

> "Duas ressalvas. Primeira: com 31 cenários só, as barras de erro são grandes. A Floresta é
> consistentemente melhor, mas eu **não** afirmaria que a diferença está estatisticamente provada.
> Segunda: o teste final **não foi rodado**. Está travado de propósito, esperando a gente fechar a
> escolha. Se rodar antes e depois mudar o modelo, o número final deixa de valer."

---

# Parte 4 — Duas coisas que precisam de decisão do grupo (2 min)

## 1. As unidades não batem — e isso é meu, para resolver

> "O enunciado diz que o rendimento está em toneladas por hectare. Só que os valores vão de 5 mil a
> 203 mil. Duzentas mil toneladas por hectare é impossível — o dendê, que é a cultura mais produtiva
> do mundo, faz uns 20 t/ha.
>
> A precipitação tem o mesmo problema: diz 'mm por dia' e os valores vão até 3.086. Isso é chuva de
> um ano, não de um dia.
>
> **Minha suspeita** é que o rendimento esteja em hectogramas por hectare, que é o padrão da base da
> FAO — aí 203.399 hg/ha dá 20,3 t/ha, que faz sentido. Mas isso é suposição minha.
>
> Então **não converti nada**. Deixei os valores originais e reporto o erro na unidade do arquivo,
> mais o erro percentual, que não depende da unidade. Eu confirmo com a FIAP. Quando confirmar, só o
> texto muda — nenhum número da análise é afetado."

## 2. Não temos ESP32 — o que muda

> "Confirmamos que não vamos ter o ESP32 físico até a entrega. Então os dois 'Ir Além' passam a ser
> **protótipo simulado no Wokwi**.
>
> Preciso que fique claro para todo mundo: **isso não cumpre o requisito do enunciado**, que pede
> ESP32 real e coleta física. A simulação demonstra o software e a comunicação, que tem valor, mas
> não é a mesma coisa. Isso está escrito em todo material, e vai ter que ser falado nos vídeos
> também.
>
> A boa notícia: **as duas entregas obrigatórias não são afetadas.** Os extras não valem nota. Se
> apertar o tempo, os extras são a primeira coisa a cortar."

---

# Parte 5 — Escolha das frentes (10 min)

> "Agora vocês escolhem. São três frentes, mais os vídeos. Deixei instruções detalhadas para cada
> uma em `docs/CONTINUIDADE_EQUIPE.md`, com o que já funciona, os comandos e o critério de pronto."

## Frente ML

**Trabalho:** revisar os cinco modelos, decidir sobre a transformação log, fechar a escolha e rodar
o teste final. Depois, o classificador do Ir Além 2.

**Já pronto:** tudo executado, com os resultados na mesa. É revisão e fechamento, não construção
do zero.

**Exige:** entender o notebook a ponto de explicar. Quem pegar essa frente grava o vídeo 1.

**Começa:** imediatamente, não depende de ninguém.

## Frente IoT / simulação

**Trabalho:** montar o circuito no Wokwi com dois sensores virtuais **distintos**, firmware com
Wi-Fi e MQTT, e o receptor Python que grava as mensagens.

**Já pronto:** só a especificação — a arquitetura e o contrato de mensagens estão em
`ir_alem/README.md`. **O código é todo por fazer.**

**Atenção técnica já resolvida:** o Wokwi não alcança o `localhost` do nosso computador. Por isso a
arquitetura passa por um broker MQTT público — o ESP32 virtual publica, o Python assina. Se tentar
HTTP direto, não funciona.

**Cuidado:** dois sensores **distintos** significa dois componentes. Temperatura e umidade do mesmo
DHT22 conta como um só.

**É a frente de maior risco** — é o caminho crítico do cronograma.

## Frente AWS / documentação

**Trabalho:** cotar a máquina nas duas regiões na calculadora oficial, salvar as capturas, escrever
a justificativa e consolidar o README.

**Já pronto:** o README está estruturado com marcadores de pendência, e `docs/aws/README.md` tem o
checklist completo, incluindo as armadilhas.

**Duas armadilhas já mapeadas:**
1. O enunciado diz "HD", mas 50 GB **não** obriga SSD — os volumes HDD `st1`/`sc1` têm mínimo de
   125 GiB, e o magnético `standard` aceita 50 GB. Escolha e **justifique**.
2. São **duas perguntas**: a mais barata, e a que você escolheria sob a restrição legal. Responda
   separado — pode ser a mesma resposta ou não.

**Começa:** imediatamente, não depende de ninguém. É a frente mais autocontida.

## Vídeos

Quatro vídeos, até 5 minutos, não listados no YouTube. Cada um depende da sua frente estar pronta.
Quem grava ainda não está definido — pode ser quem fez a frente ou não.

---

# Referência rápida — o que cada vídeo precisa ter

| # | Vídeo | Conteúdo obrigatório | Depende de |
| --- | --- | --- | --- |
| 1 | **Entrega 1 — ML** | Base, EDA, clusters, outliers, os cinco modelos, métricas, funcionamento e conclusões | Teste final fechado |
| 2 | **Entrega 2 — AWS** | Calculadora em uso, configuração, as duas regiões, valores e justificativa da escolha | Cotação pronta |
| 3 | **Ir Além 1** | Circuito, dois sensores, Wi-Fi, dados chegando ao serviço — **declarando que é simulação** | Receptor funcionando |
| 4 | **Ir Além 2** | Cultura, dados, regra dos rótulos, treino, validação e a classificação aparecendo na tela | Classificador pronto |

**Roteiro pronto para o vídeo 1** (o material já existe):

| Tempo | Conteúdo | O que mostrar |
| --- | --- | --- |
| 0:00–0:30 | A base: 156 linhas, 4 culturas, mas só 39 cenários de clima | Seção 3.1 |
| 0:30–1:30 | EDA: a cultura explica 98,8% — o R² alto é armadilha | Figuras 1 e 3 |
| 1:30–2:30 | Clusters: três regimes, escolhidos por estabilidade | Figuras 5 e 6 |
| 2:30–3:00 | Outliers: nenhum rendimento atípico; e o contraexemplo | Figura 9 |
| 3:00–4:15 | Modelos: só a Floresta supera a referência | Figura 10 |
| 4:15–4:45 | Limitações e pendências | Seção 10 |

> Nos vídeos 3 e 4 é **obrigatório** dizer que o hardware é simulado. Apresentar simulação como
> coleta real seria informação falsa numa entrega acadêmica.

---

# Referência rápida — dependências entre as tarefas

```
ML-1 revisar ──► ML-2 log do alvo ──► ML-3 teste final ──► Vídeo 1
                                            │
                                            └──► Integração final ──► SUBMISSÃO

IoT-1 Wokwi ──► IoT-2 MQTT ──► IoT-3 receptor ──► Vídeo 3
                                     │
                                     └──► IoT-4 sessões ──► ML-4 classificador ──► Vídeo 4

AWS-1 cotar ──► AWS-2 justificar ──► AWS-3 README ──► Vídeo 2

Coordenação: nomes/RMs ──► renomear notebook ──► publicar repositório ──► submeter
```

**Caminho crítico:** `IoT-1 → IoT-2 → IoT-3 → IoT-4 → ML-4 → Vídeo 4`. É a cadeia mais longa e a de
maior risco técnico. Se atrasar, o Ir Além 2 é o primeiro item a sacrificar.

**Independentes, podem começar já:** ML e AWS.

**Só eu bloqueio:** nomes e RMs (para renomear o notebook) e a publicação do repositório.

---

# Perguntas que provavelmente vão fazer

**"Por que o notebook não tem meu nome no arquivo?"**
Porque ainda não tenho os nomes completos e os RMs. O enunciado exige
`NomeCompleto_rmXXXXX_pbl_fase4.ipynb`. Me passem hoje que eu renomeio — está na minha lista.

**"O R² de 0,99 não é ótimo?"**
É enganoso. A referência que só chuta a média da cultura, sem olhar clima nenhum, também tira 0,987.
O R² alto mede a distinção entre culturas, não entendimento do clima. Por isso a comparação é dentro
de cada cultura.

**"Podemos rodar o teste para ver como ficou?"**
Podemos, mas só uma vez, e depois não dá para mudar o modelo. Se rodarmos agora e depois trocarmos
alguma coisa por causa do resultado, o número final vira propaganda, não avaliação. Melhor fechar a
escolha primeiro.

**"A simulação do Wokwi conta como entrega?"**
O enunciado diz que avalia extras incompletos, mas não confirmamos com a FIAP como a simulação será
avaliada. Não vou prometer pontuação. O que fazemos é entregar bem feito e declarar exatamente o que
foi e o que não foi atendido.

**"Por que não removeu os outliers?"**
Porque não existem. Testei dois critérios dentro de cada cultura e nenhum rendimento ficou fora.
E mostro no notebook o que aconteceria com o critério aplicado errado, sem separar por cultura:
marcaria 27 linhas como anômalas — o dendê inteiro. Não seria detectar anomalia, seria descobrir que
o dendê rende mais.

**"Dá para melhorar os modelos?"**
Dá, e deixei o caminho medido: transformar o alvo em log já melhora bastante a regressão linear.
Mas com 31 cenários, o teto é baixo. O ganho maior seria mais dados, não mais ajuste.

---

# Encerramento

> "Resumindo: a Entrega 1 está em primeira versão completa e executada. A Entrega 2 e os dois extras
> não começaram. As instruções de cada frente estão em `docs/CONTINUIDADE_EQUIPE.md`, com comandos
> testados e critério de pronto.
>
> Escolham as frentes, e quem pegar ML e AWS pode começar hoje mesmo — não dependem de ninguém.
> Me mandem nome completo e RM para eu renomear o notebook e publicar o repositório."
