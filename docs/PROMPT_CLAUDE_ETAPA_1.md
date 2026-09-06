# Pedido ao Claude Code — Etapa 1: preparar o projeto para o grupo

## Contexto e objetivo

Trabalhe em C:/Users/lcordeiro/GitHub/fase5-cap1-farmtech. Leia PLANEJAMENTO.md, docs/ENUNCIADO.md e data/crop_yield.csv. Respeite eventuais instruções locais aplicáveis.

Sou o coordenador do grupo e quero adiantar minha parte e reduzir o trabalho restante dos integrantes. Prazo: 08/09/2026 às 23:59, horário de Brasília. Hoje é 06/09: não marque etapas anteriores do cronograma como concluídas sem evidência.

Escopo final: as duas entregas obrigatórias e as duas opções de Ir Além, conforme planejamento. Nesta etapa, implemente a base do projeto, EDA, clusters/outliers e a primeira comparação dos cinco regressores; prepare documentação e instruções para o restante do grupo. Execute o trabalho, não entregue somente um plano.

Responsabilidades: coordenador = EDA, clusters e integração. Larissa, Elton e Matheus escolherão entre ML, IoT e AWS/documentação; não atribua automaticamente nomes às frentes.

Atualização de 06/09: o grupo não possui ESP32 físico e não conseguirá adquiri-lo a tempo. Leia docs/AJUSTE_IR_ALEM_SIMULADO.md e o planejamento atualizado. Os extras serão protótipos simulados; os requisitos físicos permanecem não atendidos. As entregas obrigatórias seguem completas. Esta atualização substitui a premissa anterior de coleta física.

## Constatações reais do CSV — reconfirme antes de implementar

- 156 registros e 6 colunas.
- Crop: Cocoa, beans; Oil palm fruit; Rice, paddy; Rubber, natural. Cada cultura tem 39 registros.
- Quatro variáveis climáticas e alvo Yield.
- 39 combinações climáticas distintas, cada uma presente nas quatro culturas, com quatro registros por combinação.
- Não foram encontradas linhas completas duplicadas nem valores numéricos vazios.
- Precipitation (mm day-1): mínimo 1934.62 e máximo 3085.79.
- Yield: mínimo 5249 e máximo 203399.
- Não existem colunas de data, local ou identificação de fazenda. Não inferir anos nem afirmar que os 39 grupos representam 39 anos.
- SHA-256 do original: 07B3335F497E08E705B5835EE334426AC16CB24732C1BFAA994254D39F771B1B.

Há uma dúvida material sobre unidades e agregação temporal, comparando os nomes das colunas, os valores e o enunciado. Preserve o original. Não converta Yield para t/ha nem precipitação para outra escala por suposição. Distinga “unidade declarada no enunciado” de “unidade efetiva não verificada no CSV”; reporte erros em unidades originais de Yield enquanto isso não for esclarecido. Registre essa pendência para o coordenador verificar com a FIAP. Continue os experimentos com os valores originais.

## 1. Ambiente e organização

- Inspecione o ambiente Python e use um ambiente virtual local, com versões de dependências compatíveis que você efetivamente execute.
- Crie requirements.txt, .gitignore e instruções simples para instalar dependências e abrir/executar os notebooks no Windows.
- Ignore ambiente virtual, caches e credenciais. Preserve as saídas dos notebooks na entrega.
- Organize notebooks/, docs/, docs/aws/, docs/figuras/ e ir_alem/ de forma simples. Evite arquitetura desnecessária.
- Mantenha o CSV original intacto e use caminhos relativos ao projeto.
- Pode inicializar Git local se ainda não existir. Criação/publicação do repositório remoto fica pendente para o coordenador; não é necessária nesta etapa.
- Nome completo e RM ainda faltam: use notebooks/farmtech_desenvolvimento.ipynb e registre a renomeação obrigatória antes da entrega. Não invente identificadores.

## 2. Contrato de dados e avaliação

Crie docs/CONTRATO_DADOS.md e implemente o mesmo protocolo no notebook:

- Mapeamento dos nomes originais para nomes internos, variáveis de entrada, alvo, tipos, unidades declaradas e dúvidas.
- SEED = 42 e identificador determinístico de cenário calculado pelas quatro variáveis climáticas, sem usar Crop ou Yield. O identificador serve para dividir os dados, não como entrada do modelo.
- Para avaliar generalização a cenários climáticos ainda não vistos, manter cada cenário inteiro em treino ou teste. A repetição entre culturas não é duplicata a remover.
- Sugestão: GroupShuffleSplit com n_splits=1, test_size=0.20, random_state=42 para reservar o teste. Esperam-se 31 cenários/124 linhas de desenvolvimento e 8 cenários/32 linhas de teste, se a inspeção confirmar os dados.
- Usar GroupKFold de cinco partições somente no conjunto de desenvolvimento. Todas as alternativas usam as mesmas partições.
- Registrar a divisão e o hash do CSV para que a pessoa responsável por ML reproduza exatamente o protocolo. Verificar que não há grupos compartilhados entre treino/teste nem entre treino/validação.
- Ajustar imputação, codificação e escalonamento dentro dos pipelines e de cada partição. Qualquer transformação do alvo também deve ser aprendida apenas no treino.
- Não usar o teste reservado para comparar ou ajustar alternativas nesta etapa. Ele será usado no fechamento após a revisão da equipe. A seção correspondente fica claramente marcada como pendente, sem quebrar a execução do notebook.

Documentação de referência:
- GroupShuffleSplit: https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupShuffleSplit.html
- GroupKFold: https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupKFold.html

## 3. Minha parte: EDA, clusters e outliers

Produza uma primeira versão completa, executada e explicada:

- Inventário da base, qualidade, estatísticas e dicionário das colunas.
- Antes de explorações orientadas pelo alvo, fixar a divisão. Usar o desenvolvimento para as análises que orientarão escolhas preditivas; identificar claramente o conjunto usado em cada análise.
- Distribuição de Yield por cultura, relações entre variáveis climáticas, correlações e gráficos legíveis. Distinguir diferenças entre culturas de efeitos associados ao clima.
- Analisar cenários climáticos únicos para evitar contar quatro vezes o mesmo cenário na escolha dos clusters. Justificar escalonamento, método e número de grupos, sem presumir quatro clusters.
- Relacionar os grupos encontrados aos rendimentos por cultura, informando tamanho dos grupos, padrões, limitações e ausência de evidência causal.
- Não obter clusters dominados por diferenças brutas de escala de Yield entre culturas e interpretá-los como descoberta climática.
- Investigar outliers climáticos e de rendimento dentro das culturas. Não classificar automaticamente o rendimento de uma cultura inteira como anomalia nem excluir observações sem justificativa.
- Explicações em português baseadas nos resultados realmente produzidos. Não usar conclusões genéricas ou valores fictícios.

## 4. Adiantar os cinco modelos para a pessoa responsável por ML

Implementar e executar uma comparação inicial com cinco algoritmos distintos: regressão linear, árvore de decisão, floresta aleatória, KNN e SVR. Conferir aderência ao material da disciplina quando disponível.

- Acrescentar DummyRegressor global e uma referência que prediz pela média da cultura aprendida apenas no treino de cada partição. Essas referências não substituem os cinco algoritmos.
- Comparar usando o mesmo protocolo agrupado, com MAE, RMSE e R². Apresentar resultados gerais e por cultura a partir das previsões de validação.
- A comparação por cultura e com a referência baseada em cultura deve mostrar se as variáveis climáticas acrescentam valor, além de apenas distinguir culturas.
- Usar escalas apropriadas nos modelos sensíveis a elas, incluindo o alvo do SVR se necessário, sem ajustar transformações fora do treino.
- Preferir uma busca pequena e justificada no desenvolvimento a uma otimização extensa. Não escolher uma semente ou divisão porque produz números melhores.
- Registrar parâmetros, versões, médias/variação da validação e limitações da pequena quantidade de cenários.
- Entregar resultado inicial para revisão. Não declarar vencedor definitivo com base no teste nem atribuir t/ha às métricas sem verificar as unidades.

## 5. Preparar a continuidade dos colegas

Crie docs/CONTINUIDADE_EQUIPE.md, com instruções por integrante contendo: arquivos para abrir, o que já funciona, comandos para reproduzir, tarefas restantes e critério de pronto.

- Frente ML (a escolher): revisar os cinco modelos, métricas e conclusões; fechar escolhas antes do teste final; preparar o classificador demonstrativo separado com dados simulados e limitações explícitas.
- Frente IoT (a escolher): montar ESP32 e dois sensores virtuais distintos no Wokwi, definir MQTT e registro das mensagens simuladas; não presumir acesso direto do Wokwi à rede local nem afirmar teste físico.
- Frente AWS/documentação (a escolher): preparar estimativas oficiais nas duas regiões, capturas, justificativa, figuras e vídeo; consolidar README.
- Coordenador: conferir dúvidas de unidades com a FIAP, preencher nomes/RMs, revisar notebook, integrar material e preparar vídeo/submissão.

Prepare o README com seções claras para notebook, instalação, equipe, AWS, as duas opções de Ir Além e quatro vídeos. Identifique pendências em texto; não invente URLs, preços, capturas ou resultados.

Prepare ir_alem/README.md com a arquitetura Wokwi → broker MQTT na internet → assinante Python local, e um contrato de mensagens: horário, dispositivo, sensor, variável, valor, unidade, cultura de referência/sessão, origem simulada e proveniência dos rótulos. Escolher componentes suportados pelo simulador e registrar que o circuito não foi validado fisicamente.

O classificador será demonstrativo. Não transformar rendimento acima da mediana em “saúde”, nem apresentar dados simulados como coleta física. Se os rótulos vierem de regras artificiais, avaliar apenas a reprodução dessas regras; não alegar diagnóstico agronômico. Nesta etapa, preparar as interfaces/instruções e distinguir protótipo simulado de cumprimento integral dos extras.

## 6. Verificação e entrega deste bloco

- Executar o notebook de desenvolvimento do início ao fim em kernel limpo e salvar as saídas.
- Verificar preservação do CSV, separação dos grupos, execução das cinco alternativas e consistência dos textos com as métricas.
- Conferir visualmente gráficos e tabelas para evitar rótulos cortados ou informação ilegível.
- Confirmar que as instruções de execução correspondem ao ambiente testado.
- Manter teste final, AWS, protótipo e vídeos como pendentes enquanto não houver evidências de conclusão. Registrar hardware/coleta físicos e validação real de saúde como requisitos não atendidos; uma simulação funcionando não conclui esses requisitos.
- Ao terminar, informar arquivos criados, comandos usados, resultados principais, verificações realizadas e pendências por integrante.
- Explicar em linguagem simples as decisões de agrupamento, unidades e avaliação, para que eu consiga apresentar o trabalho aos colegas.

Se faltar informação pessoal, avance nas partes independentes e registre a pendência. A ausência de hardware físico já está confirmada; siga o plano de simulação sem depender de compra. Se houver problema técnico, tente corrigi-lo e relate precisamente o que não conseguiu executar.
