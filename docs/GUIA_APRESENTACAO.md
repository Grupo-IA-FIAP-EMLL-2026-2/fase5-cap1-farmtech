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
| Outliers | ✅ **executada** | 2 critérios, IQR e z robusto |
| **Cinco regressores + 2 referências** | ✅ **executados** | validação aninhada, `docs/resultados_modelos.csv` |
| Diagnóstico do KNN | ✅ **executado** | composição das vizinhanças, seção 8.4 |
| Documentação para o grupo | ✅ **escrita** | 6 documentos |
| Revisão independente | ✅ **aplicada** | `docs/REVISAO_E_AJUSTES_ETAPA_1.md` |
| Teste final | ⏸️ **travado de propósito** | Seção 9, esperando decisão |
| Entrega 2 (AWS) | ⏳ **documentada, não cotada** | checklist em `docs/aws/README.md` |
| Ir Além 1 e 2 | ⏳ **especificados, não implementados** | `ir_alem/README.md` |
| Vídeos | ❌ **nenhum gravado** | — |

**Situação para apresentar ao grupo:** *"O núcleo técnico da Entrega 1 está executado e revisado;
a escolha do modelo e o teste final continuam pendentes. Da Entrega 2 e dos dois extras existe a
documentação — arquitetura, contrato
de mensagens, checklist da AWS —, mas as cotações e o código ainda não foram feitos."*

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
> diferentes. Mas significa que a base descreve 39 condições de clima diferentes, não 156.
>
> Um cuidado de linguagem: eu falo em '39 cenários distintos', não em '39 observações
> independentes'. Como a base não tem data nem local, não dá para saber se esses cenários são
> independentes entre si — podem ser anos seguidos da mesma região, por exemplo.
>
> A consequência prática: se dividíssemos as linhas ao acaso entre treino e teste, o modelo veria no
> treino condições climáticas também presentes no teste. Isso avaliaria uma situação diferente
> da que queremos estudar: prever rendimento em cenários climáticos ainda não vistos.
>
> Por isso a divisão é **por cenário**: cada combinação de clima fica inteira de um lado só."

**Números:** 124 linhas / 31 cenários para desenvolver, 32 linhas / 8 cenários reservados.

## Achado 2 — A cultura responde por 98,8% da variação do rendimento

**Mostre:** Figura 1 (escala linear vs. log) e a saída do eta² na seção 5.2.

> "Olhem o gráfico da esquerda. O dendê é tão maior que as outras três culturas viram uma linha
> achatada embaixo. Um dendê rende umas vinte vezes mais que um cacau.
>
> Isso tem uma consequência que quase derrubou a análise: **a diferença entre as culturas responde
> por 98,8% da variação do rendimento** — é uma decomposição estatística da variância, não uma
> afirmação de que a cultura *causa* isso. Eu criei um modelo bobo, que ignora o clima
> completamente e só chuta a média da cultura. Ele tira **R² de 0,987**.
>
> Então se a gente escrever no relatório 'nosso modelo tem R² de 0,99', isso é verdade mas não prova
> o que a gente quer: sozinho, esse número não diz se o modelo aprendeu algo sobre clima ou só
> aprendeu a diferenciar quatro culturas, coisa que a coluna já dava de graça.
>
> Por isso comparei tudo **contra esse modelo bobo** e olhei as métricas **dentro de cada cultura**.
> É aí que a pergunta de verdade aparece: o clima acrescenta alguma coisa?"

## Achado 3 — A associação com o clima aparece mais em arroz e borracha, em sentidos opostos

**Mostre:** Figura 3 (painel b) e Figura 7.

> "E a resposta é: sim, mas a associação aparece de forma clara em duas das quatro culturas.
>
> O arroz rende mais quando está quente e o ar está úmido — correlação de +0,70 com umidade
> específica e +0,61 com temperatura. A borracha faz o contrário: −0,43 e −0,41. Em cacau e dendê as
> correlações ficaram fracas.
>
> Cheguei nisso por **três caminhos complementares** — as correlações, os grupos de clima e o modelo
> — e os três apontaram para o mesmo lugar. Isso mostra que a análise é internamente coerente. Mas
> atenção: são três leituras **dos mesmos 124 dados**, não três confirmações independentes. Elas
> compartilham as mesmas limitações.
>
> E duas coisas que eu **não** posso afirmar. Primeira: que o clima não afeta cacau e dendê — com 31
> cenários e uma faixa de temperatura de só 1,2 grau, um efeito real e pequeno passaria batido.
> Segunda: qualquer explicação de causa. A base não tem solo, adubação, manejo, variedade, nem
> sequer o local ou o ano — então nem dá para dizer se o arroz era irrigado. O que eu tenho é a
> associação nos números, e paro aí."

---

# Parte 3 — Resultados dos modelos (3 min)

**Mostre:** Figura 10.

| Alternativa | MAE | R² geral | Culturas em que supera a referência (por MAE) |
| --- | --- | --- | --- |
| **3. Floresta Aleatória** | **3.597** | 0,990 | **4 de 4** |
| *Ref. média da cultura* | *4.772* | *0,987* | *referência* |
| 2. Árvore de Decisão | 4.800 | 0,982 | 2 de 4 — arroz e borracha |
| 1. Regressão Linear | 5.025 | 0,987 | 2 de 4 — dendê e arroz |
| 4. KNN | 6.759 | 0,948 | 0 de 4 |
| 5. SVR (RBF) | 7.365 | 0,975 | 0 de 4 |

> "Olhem a coluna do R² geral: todos entre 0,95 e 0,99. Parece que está tudo ótimo — inclusive o
> modelo bobo, com 0,987.
>
> Agora a última coluna, que é o que interessa: em quantas culturas cada um erra menos que o chute
> da média da cultura. **Só a Floresta Aleatória consegue nas quatro.** A árvore e a regressão
> linear conseguem em duas cada, mas em culturas diferentes. KNN e SVR não conseguem em nenhuma.
>
> Uma coisa importante que eu preciso dizer aqui: **essa tabela mudou depois da revisão.** Antes, a
> busca dos parâmetros de cada modelo usava os mesmos dados que depois mediam o resultado — isso
> deixava os números otimistas. Corrigi para uma validação aninhada, em que a busca acontece só
> dentro do treino de cada partição. A árvore de decisão, que aparecia **melhor** que o modelo bobo,
> passou a aparecer **pior**. A ordem mudou. É exatamente o tipo de erro que só aparece quando
> alguém confere.
>
> Sobre o KNN, eu levantei uma hipótese e **fui verificar** em vez de só afirmar. A hipótese era que
> ele misturava culturas ao procurar vizinhos. Fui olhar de que cultura vinha cada vizinho: em só 16
> das 124 linhas entra um vizinho de outra cultura — mas essas 16 linhas concentram **57% de todo o
> erro dele**. Nas outras 108 linhas, o erro médio foi 3.328. Isso mostra onde os erros se
> concentram; comparar esse subgrupo com a referência exigiria usar as mesmas 108 linhas.
>
> Um detalhe honesto: isso explica o KNN, cujos vizinhos dá para inspecionar. Para o SVR o mesmo
> mecanismo é plausível, mas eu **não** demonstrei — deixei registrado como tarefa em aberto."

**Ponto de honestidade a mencionar:**

> "Três ressalvas. Primeira: com 31 cenários só, as barras de erro são grandes. A Floresta é
> consistentemente melhor, mas eu **não** afirmaria que a diferença está estatisticamente provada.
>
> Segunda: mesmo com a validação aninhada, escolher o vencedor olhando essas métricas ainda é uma
> escolha. Só o teste reservado resolve isso de verdade.
>
> Terceira: o teste final **não foi rodado**. Está travado de propósito, e agora ele nem sabe qual
> modelo usar — quem pegar a frente de ML precisa registrar a escolha numa variável, senão o
> notebook para com uma mensagem de erro. Isso foi de propósito: antes ele estava com a Floresta
> fixa no código, e se a gente escolhesse outro modelo ia avaliar o errado sem avisar."

---

# Parte 4 — Duas coisas que precisam de decisão do grupo (2 min)

## 1. As unidades não batem — e isso é meu, para resolver

> "O enunciado diz que o rendimento está em toneladas por hectare. Só que os valores vão de 5 mil a
> 203 mil. Essa escala levanta uma dúvida sobre a unidade declarada.
>
> A precipitação exige a mesma conferência: a coluna diz 'mm por dia' e os valores vão até 3.086.
> Precisamos confirmar a unidade e o período a que esses números se referem.
>
> Eu **não** vou chutar qual é a unidade certa. Qualquer palpite meu viraria número no relatório, e
> a gente não sabe.
>
> Então **não converti nada**. Deixei os valores originais e reporto o erro na unidade do arquivo,
> mais o erro percentual. Eu confirmo com a FIAP.
>
> E uma correção que a revisão me apontou: eu tinha dito que, quando confirmasse, só o texto mudaria.
> Isso está errado. Se a resposta for que os valores precisam ser convertidos — divididos por dez
> mil, por exemplo —, o **MAE e o RMSE mudam** na mesma proporção, e as escalas dos gráficos também.
> O que **não** muda são o R² e o erro percentual, porque eles não dependem da escala. A ordem entre
> os modelos também se mantém. Está tudo detalhado na seção 3.3 do notebook."

## 2. Não temos ESP32 — o que muda

> "Confirmamos que não vamos ter o ESP32 físico até a entrega. Então os dois 'Ir Além' passam a ser
> **protótipo simulado no Wokwi**.
>
> Preciso que fique claro para todo mundo: **isso não cumpre o requisito do enunciado**, que pede
> ESP32 real e coleta física. A simulação demonstra o software e a comunicação, que tem valor, mas
> não é a mesma coisa. Isso está escrito em todo material, e vai ter que ser falado nos vídeos
> também.
>
> **As duas entregas obrigatórias continuam com escopo completo.** Os dois extras permanecem no
> plano como protótipos simulados, com os requisitos físicos não atendidos claramente declarados."

---

# Parte 5 — Escolha das frentes (10 min)

> "Agora vocês escolhem. São três frentes, mais os vídeos. Deixei instruções detalhadas para cada
> uma em `docs/CONTINUIDADE_EQUIPE.md`, com o que já funciona, os comandos e o critério de pronto."

## Frente ML

**Trabalho:** revisar os cinco modelos, decidir sobre a transformação log, fechar a escolha e rodar
o teste final. Depois, o classificador do Ir Além 2.

**Já pronto:** tudo executado, com os resultados na mesa. É revisão e fechamento, não construção
do zero.

**Exige:** entender o notebook a ponto de explicar.

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
| 0:30–1:30 | EDA: a cultura responde por 98,8% da variação — o R² sozinho não basta | Figuras 1 e 3 |
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
maior risco técnico — convém acompanhá-la de perto nos alinhamentos de 12h e 18h.

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
Porque os dois critérios que apliquei — IQR e escore z robusto, dentro de cada cultura — não
marcaram nenhum rendimento nas 124 linhas de desenvolvimento. Isso é o que esses dois critérios
dizem sobre esses dados; não é uma garantia de que não exista nada estranho por outros critérios.
E mostro no notebook o que aconteceria sem separar por cultura: marcaria 27 das 31 linhas de dendê.
Não seria detectar anomalia, seria descobrir que o dendê opera numa escala maior que as outras.

**"Dá para melhorar os modelos?"**
Dá, e deixei o caminho medido: transformar o alvo em log leva a regressão linear a errar menos que o
modelo bobo nas quatro culturas — embora pelo R² ela só ganhe em duas, e essa discordância entre as
métricas é uma decisão que a frente de ML vai ter que tomar. Mas com 31 cenários o teto é baixo: o
ganho maior viria de mais dados, não de mais ajuste.

**"Por que a tabela dos modelos mudou depois da revisão?"**
Porque a busca dos hiperparâmetros usava as mesmas partições que depois mediam o resultado, o que
deixava os números otimistas. Agora a busca roda só dentro do treino de cada partição. O efeito foi
real: a árvore de decisão saiu de 4.340 (melhor que o modelo bobo) para 4.800 (pior). Nenhuma outra
posição mudou, mas essa mudou — e é por isso que vale conferir o trabalho dos outros.

---

# Encerramento

> "O núcleo técnico da Entrega 1 está executado e revisado; faltam a escolha do modelo e o teste
> final. AWS e extras têm documentação preparada, com cotações e implementação pendentes. As
> instruções de cada frente estão em `docs/CONTINUIDADE_EQUIPE.md`, com comandos
> testados e critério de pronto.
>
> Escolham as frentes, e quem pegar ML e AWS pode começar hoje mesmo — não dependem de ninguém.
> Me mandem nome completo e RM para eu renomear o notebook e publicar o repositório."
