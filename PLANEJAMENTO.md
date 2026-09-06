# Planejamento — FarmTech Solutions, Fase 5

Atualizado em **06/09/2026**. Prazo informado: **08/09/2026 às 23:59**, horário de Brasília. Meta interna de envio: **08/09 às 22h**.

Integrantes: você (nome completo pendente), Larissa, Elton e Matheus. Você coordena, faz EDA/clusters/outliers e integra o notebook. **Os demais escolherão entre as três frentes abaixo; os nomes ainda não estão atribuídos.**

## Ajuste confirmado: sem ESP32 físico

O grupo informou que não possui ESP32 físico e não conseguirá comprá-lo a tempo. O cronograma deixa de depender de aquisição ou coleta física. Mantemos as duas entregas obrigatórias e planejamos desenvolver as duas opções extras como **protótipos em simulação**, com as limitações explicitadas.

O enunciado exige ESP32 real nas duas opções extras. O protótipo simulado não cumpre integralmente esses requisitos. O texto permite avaliar entregas extras incompletas, sem garantir aceitação integral ou pontuação específica. O coordenador pode consultar a FIAP sobre a avaliação da simulação; não há confirmação de aceitação.

As entregas obrigatórias de ML e AWS seguem com todos os requisitos. Os extras têm reconhecimento próprio, separado da nota dessas entregas.

## Escopo e evidências

| Entrega | O que desenvolver | Como apresentar |
| --- | --- | --- |
| 1 — Machine Learning | EDA, clusterização, outliers e cinco algoritmos de regressão | Notebook executado, métricas, conclusões, limitações e vídeo de até 5 minutos |
| 2 — AWS | Linux On-Demand 100%, São Paulo e Virgínia do Norte, 2 CPUs, 1 GiB, rede até 5 Gbps e 50 GB de armazenamento | Cotações na calculadora, evidências e justificativa no README; vídeo de até 5 minutos |
| Ir Além 1 — Protótipo simulado | ESP32 virtual, dois sensores virtuais distintos, publicação MQTT, recebimento e registro | Código, circuito, logs e vídeo identificados como simulação; requisito de ESP32 físico não atendido |
| Ir Além 2 — Protótipo simulado | Modelo demonstrativo com dados simulados, novas sessões simuladas e exibição do resultado | Explicar geração e rótulos; métricas limitadas ao cenário artificial; coleta física e validação real de saúde não atendidas |

Planejar quatro vídeos não listados no YouTube, até cinco minutos cada. Nos extras, informar a simulação no título, na demonstração e no README. Não é necessário contratar uma máquina AWS para a cotação.

## Frentes para o grupo escolher

| Frente | Responsabilidades | Evidências e apoio |
| --- | --- | --- |
| Coordenador — você | EDA, clusters, outliers, integração, revisão e submissão | Notebook consolidado; apoio ao vídeo ML e orientação dos colegas |
| ML — a escolher | Revisar/concluir os cinco regressores; desenvolver classificador demonstrativo separado | Métricas, rótulos documentados, limitações e apoio aos vídeos ML/Ir Além 2 |
| IoT — a escolher | Circuito Wokwi, dois sensores virtuais, firmware, MQTT e registro de dados simulados | Projeto reproduzível, mensagens recebidas e apoio ao vídeo Ir Além 1 |
| AWS/documentação — a escolher | Calculadora AWS, justificativa, README, figuras e apoio à visualização | Cotações verificáveis, documentos e apoio ao vídeo AWS |

Cada integrante explica sua parte. Responsáveis por apresentação, gravação e publicação dos quatro vídeos serão escolhidos pelo grupo. O coordenador adianta a estrutura e a primeira comparação dos modelos; os colegas revisam e concluem as respectivas frentes.

## Cronograma revisto — 06 a 08/09

O CSV, o ambiente e o núcleo técnico do notebook já estão preparados, executados e revisados. Faltam a escolha do modelo, o teste final, as cotações AWS, a implementação dos dois extras simulados e os vídeos. A tabela abaixo registra as metas; o estado detalhado e as instruções atuais estão em `docs/CONTINUIDADE_EQUIPE.md`.

| Quando | Trabalho em paralelo | Marco esperado |
| --- | --- | --- |
| **06/09 — restante da tarde** | Coordenador/Claude avançam na estrutura e notebook; grupo escolhe frentes; IoT inicia Wokwi e teste MQTT; AWS inicia cotações; ML define cenários/rótulos demonstrativos | Base executável inicial, responsabilidades definidas e prova de comunicação do simulador |
| **06/09 — noite, até 22h** | EDA/clusters e cinco regressores em primeira versão; IoT registra mensagens simuladas; ML prepara classificador demonstrativo; AWS organiza evidências | Notebook inicial e protótipo em primeira versão, com pendências registradas |
| **07/09 — manhã** | Integrar/revisar notebook; executar validação dos modelos conforme contrato; testar sensores virtuais → MQTT → receptor → modelo → tela; fechar AWS | Entregas obrigatórias verificáveis e fluxo simulado reproduzível |
| **07/09 — tarde** | Corrigir erros, revisar conclusões, preparar figuras e roteiros; conferir que toda evidência simulada está identificada | Versão candidata pronta, com requisitos atendidos e não atendidos explicitados |
| **07/09 — noite** | Gravar e publicar os quatro vídeos como não listados | Links no README, todos com até 5 minutos |
| **08/09 — até 15h** | Revisão cruzada: notebook do zero, instruções, links públicos, vídeos, custos e limitações | Pendências finais identificadas |
| **08/09 — até 20h** | Correções, eventual regravação e fechamento | Versão final aprovada pelo grupo |
| **08/09 — até 22h** | Envio no portal e registro do comprovante | Entrega antes das 23:59; nenhum commit posterior ao envio |

A simulação permanece no escopo dos extras. Se a comunicação externa falhar, registrar a falha e distinguir testes locais de transmissão pelo Wokwi; não apresentar testes locais como evidência de rede funcionando.

## Arquitetura proposta para os extras

**ESP32 e dois sensores no Wokwi → Wi-Fi virtual → broker MQTT acessível pela internet → assinante Python no computador → CSV/SQLite → classificador demonstrativo → resultado na tela.**

O gateway público padrão do Wokwi permite conexões de saída à internet, inclusive MQTT, mas não acesso direto à rede local do computador. Por isso, o receptor local assina as mensagens do mesmo broker; o ESP32 virtual não tenta enviar HTTP diretamente a localhost. Confirmado em 06/09 na [documentação oficial do Wokwi](https://docs.wokwi.com/guides/esp32-wifi).

- Escolher dois componentes sensores distintos que o simulador suporte e justificar sua relação com o contexto agrícola. Duas variáveis de um único sensor não equivalem a dois sensores distintos.
- A comunicação pela internet pode funcionar de fato, mas hardware, Wi-Fi e medições continuam simulados.
- Registrar explicitamente a origem simulada em dados, figuras, textos e vídeos. Não enviar informações pessoais ou credenciais por um broker público.
- Ir Além 1 demonstra coleta virtual e comunicação. Ir Além 2 acrescenta a classificação demonstrativa.
- Não comprar plano, contratar infraestrutura ou expor portas locais como dependência deste planejamento. Verificar a disponibilidade do broker escolhido durante a implementação.

## Dados e classificação simulada

- Manter a base original crop_yield.csv exclusiva da Entrega 1. Não transformar sua mediana de rendimento em diagnóstico de saúde.
- Definir cultura de referência, variáveis e cenários simulados. Registrar sensor, variável, valor, unidade, origem, sessão e regra de rotulagem.
- Se usar regras para criar rótulos artificiais, documentá-las como hipóteses do exercício. Treinar e testar sobre a mesma regra demonstra sua reprodução, não comprova capacidade de diagnosticar plantas reais.
- Separar sessões simuladas para treino e teste e relatar as métricas apenas nesse contexto. Não chamar essas sessões de coleta de campo.
- Usar na interface “classificação simulada” e, se exibir os nomes exigidos “Saudável”/“Não saudável”, manter visível que são rótulos demonstrativos sem validação agronômica real.
- Conservar no relatório a lista do que falta para cumprir integralmente a opção 2: sensores físicos, observação/rotulagem justificada e avaliação com novas coletas físicas.
- Não marcar hardware físico nem validação real como concluídos.

## Acordos antes de programar

1. Obter o `crop_yield.csv` original do portal e conferir nomes, unidades, tamanho e distribuição dos dados. Não inferir resultados antes disso.
2. Definir cultura e condições ambientais como possíveis entradas e rendimento como alvo, após inspeção do CSV.
3. A pessoa responsável por ML define a divisão treino/teste e o protocolo de validação antes de escolhas orientadas por desempenho. Se houver estrutura temporal ou agrupamentos, adequar a divisão para evitar vazamento. Todos os regressores usam a mesma divisão e as mesmas partições de validação.
4. Ajustar imputação, codificação e escalonamento apenas no treino, dentro dos pipelines. Usar as mesmas divisões e métricas para todos os regressores.
5. A exploração descritiva e a clusterização não devem orientar ajustes usando o teste reservado. Não remover outliers automaticamente: investigar se são erros ou cenários agrícolas válidos.
6. Explicitar se rendimento entra na clusterização. Isso pode servir à análise descritiva, mas não pode se transformar em uma entrada do modelo preditivo que depende do alvo desconhecido.
7. Definir versões das dependências e caminhos relativos para executar o notebook em outra máquina. Usar uma constante `SEED = 42` nos procedimentos aleatórios que aceitem semente, sem redefinições nas seções.
8. Manter uma única função de métricas para todos os regressores. A divisão pode ser reproduzida por código centralizado e versão fixa dos dados; salvar índices é útil quando necessário, não uma exigência do enunciado.
9. Para a clusterização descritiva, documentar o conjunto usado para ajustar as transformações. A regra de ajustar apenas no treino se aplica à avaliação preditiva; não é uma proibição de escalonar os dados de uma análise descritiva independente.
10. Revisar qualquer código produzido com IA: o autor precisa executar, entender e explicar sua parte. As conclusões devem apontar evidências reais das saídas; não inserir métricas de exemplo ou explicações causais não demonstradas.

## Proposta técnica inicial, sujeita ao CSV

- Regressão: LinearRegression, DecisionTreeRegressor, RandomForestRegressor, KNeighborsRegressor e SVR. Conferir aderência aos algoritmos ensinados na disciplina. Um DummyRegressor adicional serve como referência e não substitui os cinco.
- Comparação: MAE e RMSE nas unidades originais de Yield, até esclarecer a divergência de unidades, além de R²; validação cruzada no treino e avaliação final no teste reservado. Evitar escolher modelos repetidamente pelo teste.
- Clusterização: começar com um método interpretável como K-Means, com escalonamento e avaliação da separação e utilidade dos grupos. Considerar outro método se o formato dos dados justificar. Não assumir que número de clusters deve ser igual ao número de culturas.
- Outliers: combinar gráficos e investigação dos registros com um critério quantitativo justificado. Registrar limitações e eventual efeito nos resultados.
- Não prometer acurácia, número de grupos ou melhor algoritmo antes dos experimentos.

## Cuidados específicos da AWS

- Pesquisar a instância e o armazenamento compatíveis antes de cotar; registrar tipo, arquitetura, desempenho de rede e eventuais limites.
- Candidatas verificadas em documentação oficial em 05/09: `t3.micro`, `t3a.micro` e `t4g.micro`, com 2 vCPUs, 1 GiB e rede de até 5 Gbps em burst. T4g usa ARM; conferir compatibilidade da aplicação e disponibilidade/preço nas duas regiões. As famílias usam créditos de CPU; registrar a premissa de uso e eventual custo adicional. Fontes: [AWS T3/T3a](https://aws.amazon.com/ec2/instance-types/t3/) e [AWS T4g](https://aws.amazon.com/ec2/instance-types/t4/).
- Usar premissas iguais nas duas regiões: horas mensais, sistema operacional, modalidade de compra e armazenamento. Explicitar itens incluídos e excluídos.
- Registrar data, moeda, preços dos componentes e total mensal, acompanhados das evidências da calculadora.
- O enunciado usa “HD”: conferir compatibilidade do tipo de volume escolhido com a capacidade de 50 GB. Explicar qualquer interpretação em vez de escolher silenciosamente um tipo incompatível.
- Atenção à distinção: `st1` e `sc1` têm mínimo de 125 GiB, mas o volume magnético de geração anterior `standard` admite de 1 GiB a 1 TiB. Portanto, não afirmar que 50 GB obrigam SSD. Conferir as opções da calculadora; se adotar `gp3`, justificar a interpretação de “HD” como armazenamento e registrar a unidade efetivamente cotada. Fonte: [tipos de volumes EBS](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html).
- Separar a resposta sobre menor custo da escolha sob a restrição de armazenamento no exterior dada pelo exercício. Não tratar essa hipótese como uma proibição legal universal.
- Fundamentar latência sem inventar medições e verificar preços atuais quando a cotação for feita.

## Integração e uso de IA

- Cada integrante pode usar IA, mas deve executar e explicar o que recebeu. Salvar evidências dos resultados e das limitações.
- O coordenador é o único integrador do notebook final. Combinar arquivos separados para desenvolvimento e revisão.
- A pessoa de ML revisa os modelos e o contrato de avaliação antes do teste final reservado.
- A pessoa de IoT mantém firmware e receptor; a pessoa de AWS/documentação consolida README e figuras. Acordar os campos das mensagens antes da integração.
- Preservar caminhos relativos, dependências documentadas e saídas dos notebooks. Reiniciar o kernel e executar tudo antes de fechar a entrega.
- Não sobrescrever trabalhos de outra sessão em andamento. Comunicar a atualização do escopo ao Claude Code.

## Vídeos

| Vídeo | Conteúdo mínimo |
| --- | --- |
| Entrega 1 | Base, EDA, clusters/outliers, cinco modelos, métricas e conclusões |
| Entrega 2 | Calculadora AWS, configurações, valores das duas regiões e justificativa |
| Ir Além 1 — simulação | Circuito Wokwi, dois sensores virtuais, mensagens MQTT e recebimento; declarar ausência de ESP32 físico |
| Ir Além 2 — simulação | Dados/rótulos artificiais, treinamento, avaliação em novas sessões simuladas, resultado na tela e limites de validade |

Meta de duração: 4min30s; máximo: 5 minutos por vídeo. Definir no grupo quem apresenta, grava e publica cada um. Os quatro links ficam no README.

## Checklist de entrega

- [ ] Notebook executado, com saídas, comentários, explicações, conclusões e limitações.
- [ ] EDA, clusters, outliers e cinco regressores com avaliação coerente.
- [ ] Nome final do notebook com nome completo, RM e sufixo solicitado pbl_fase4.ipynb.
- [ ] AWS com configuração exigida, duas cotações, capturas e justificativa.
- [ ] Protótipo Wokwi com dois sensores virtuais e código documentado.
- [ ] Comunicação MQTT e registro de dados demonstrados, ou falhas explicitadas.
- [ ] Classificador demonstrativo com origem dos dados, rótulos e limites documentados.
- [ ] Requisitos não atendidos dos extras listados: hardware físico, coleta física e validação real de saúde.
- [ ] Quatro vídeos identificados corretamente, não listados, com até cinco minutos e links acessíveis.
- [ ] README, diagramas e instruções correspondem ao que efetivamente funciona.
- [ ] Nomes/RMs preenchidos, responsabilidades escolhidas e repositório público.
- [ ] Envio no portal e comprovante; nenhum commit após o envio.

## Pendências de coordenação

- Confirmar escolhas das três frentes e responsáveis pelos vídeos.
- Consultar a FIAP sobre como será avaliada a simulação dos extras; registrar a resposta, se houver. Não há autorização para enviar mensagens em nome do grupo nesta tarefa.
- Esclarecer as unidades de precipitação/Yield e a nomenclatura pbl_fase4.ipynb.
- Completar nomes/RMs e confirmar o responsável pelo envio.
