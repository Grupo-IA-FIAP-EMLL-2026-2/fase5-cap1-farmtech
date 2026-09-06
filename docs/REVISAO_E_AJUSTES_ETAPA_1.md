# Revisão da Etapa 1 e pedido de ajustes ao Claude Code

Data: 06/09/2026. Base revisada: commit local e053a3e.

## O que foi confirmado

- Notebook com 74 células, das quais 41 são de código, e 12 figuras.
- Reexecução independente em uma cópia temporária: 42,7 segundos, sem erro de célula.
- A tabela resultados_modelos.csv reproduzida é idêntica, byte a byte, à salva no projeto.
- CSV original preservado; 156 registros, 39 cenários distintos; divisão de 124/32 linhas e 31/8 cenários.
- Cinco algoritmos e duas referências implementados.
- EXECUTAR_TESTE_FINAL permanece False; o teste reservado não foi avaliado nesta revisão.
- AWS, simulação, classificador extra e vídeos ainda precisam ser implementados/concluídos. As especificações desses itens já existem.
- Houve avisos de ambiente e rastros de erro no encerramento do gerenciador de processos joblib/loky no Windows. Eles não impediram a execução nem mudaram a tabela; resolver antes de gravar uma demonstração que inclua o terminal.

A implementação é uma primeira versão reproduzível. A narrativa e alguns detalhes da avaliação precisam dos ajustes abaixo antes de apresentar conclusões como definitivas.

## Pedido ao Claude Code

Aplique as correções abaixo preservando o CSV, a divisão registrada, a semente e o trabalho existente. Mantenha o teste final desativado. Atualize o notebook, as figuras/tabelas afetadas e toda documentação que repetir conclusões. As frentes e os vídeos continuam com responsável a definir; os extras continuam no escopo como protótipos simulados.

As referências a células usam índices começando em zero, como no JSON do notebook, e também indicam a seção correspondente.

### 1. Corrigir a avaliação e identificar suas limitações

Nas células 55 e 65 (seções 8 e 8.4), GridSearchCV escolhe parâmetros usando PARTICOES em todo o desenvolvimento e depois as mesmas partições são usadas para reportar previsões e métricas da alternativa escolhida. Isso é útil para seleção inicial, mas não constitui avaliação independente dessa seleção.

A limitação já está mencionada no item 7 da seção 10.3, porém outras frases dizem que nenhuma informação da validação influencia a escolha e qualificam o viés como “leve” sem quantificá-lo.

Para melhorar a comparação, faça validação agrupada aninhada: preserve as cinco partições externas e realize a busca dentro do treino de cada partição, com grupos também no nível interno. Uma busca interna pequena de três partições é suficiente como proposta inicial, respeitando o número de grupos disponível. Calcule previsões externas para os cinco algoritmos e referências. Após escolher a alternativa, pode ajustar seus parâmetros no desenvolvimento completo para a futura avaliação final.

Evite paralelismo aninhado excessivo: escolha um nível de paralelismo ou torne n_jobs configurável. Aplique a mesma disciplina à comparação com transformação logarítmica.

Recalcule tabelas e textos que dependam dessas métricas, sem antecipar que a classificação dos modelos permanecerá igual. A avaliação externa de cada alternativa também não elimina todo o viés de escolher o vencedor por essas mesmas métricas; a confirmação final continua reservada.

Referência: [scikit-learn — validação aninhada e seleção de hiperparâmetros](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html).

### 2. O teste final precisa usar a alternativa realmente escolhida

Na célula 70, a chamada está fixada em avaliar_no_teste_reservado(floresta_final). Se a equipe escolher outro algoritmo ou a transformação logarítmica, apenas ligar EXECUTAR_TESTE_FINAL ainda avaliará a Floresta Aleatória.

Crie uma seleção explícita da alternativa final, com pipeline, transformações e parâmetros registrados, e faça a seção 9 usar essa configuração. Inclua uma verificação que falhe claramente se a escolha não estiver definida. Mantenha EXECUTAR_TESTE_FINAL = False nesta revisão.

Corrija também a explicação de “rodar uma única vez”: reexecutar exatamente o mesmo procedimento para verificar reprodutibilidade ou permitir a correção do professor não invalida o teste. O problema é usar seus resultados para ajustar ou escolher alternativas. A execução do notebook inteiro precisa continuar possível na entrega.

### 3. Corrigir a comparação com a referência por cultura

A célula 60 informa que a regressão linear supera a referência “só no arroz”. A própria tabela salva mostra também ganho no dendê:

- Regressão linear: R² = -0,013 e MAE = 11.973 no dendê.
- Referência por cultura: R² = -0,039 e MAE = 12.288 no dendê.

Logo, R² negativo não significa automaticamente perder para a referência que foi treinada em outra partição. O zero usa a média do conjunto avaliado; a referência do projeto usa médias aprendidas nos treinos.

Compare cada alternativa diretamente com a referência na mesma cultura e métrica. Gere essas indicações programaticamente para evitar tabelas manuais incoerentes. Se a avaliação mudar, use os resultados novos.

Defina também a métrica da frase “supera a referência”: a árvore, por exemplo, melhora MAE no dendê e na borracha, embora seu R² seja pior nesses dois casos.

Referência: [scikit-learn — definição de R²](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.r2_score.html).

### 4. Corrigir a tabela e a interpretação da escolha de clusters

Na célula 36, a linha “5+” informa estabilidade ARI menor ou igual a 0,654 e silhueta menor ou igual a 0,331. A saída da célula 34 contém:

- k=7: ARI 0,819.
- k=8: ARI 0,758 e silhueta 0,332.

Corrija os números e a afirmação de queda conjunta para todos os valores. O k=3 continua tendo a maior estabilidade medida (0,923) nessa execução, então o erro não elimina automaticamente a escolha.

Não afirmar que um grupo de dois pontos sempre aumenta a silhueta “por construção” ou que não pode representar um regime. Descrever o tamanho pequeno como razão de cautela e preferência por uma solução mais estável nesta amostra.

### 5. Reduzir conclusões que os dados não demonstram

Revisar especialmente células 12, 24, 28, 32, 42, 48, 52, 60 e 71; README; guia de apresentação; contrato e continuidade.

- Usar “39 cenários distintos”. Sem datas e origem detalhada, a independência estatística desses cenários não foi demonstrada.
- Trocar “o clima só age/importa em duas culturas” por “as associações lineares e as diferenças médias entre os grupos ficaram mais evidentes em arroz e borracha nesta amostra”. Isso não prova ausência de relação nas outras culturas nem efeito causal.
- Trocar “três análises independentes” por “três análises complementares sobre os mesmos dados”.
- Descrever o R² global como insuficiente isoladamente para avaliar ganho além da cultura, não como inútil ou irrelevante.
- Os 98,8% referem-se à decomposição descritiva da variância no desenvolvimento; não são porcentagem da produção causada pela cultura.
- Trocar “não existem outliers” por “os dois critérios aplicados não identificaram outliers de rendimento nas 124 linhas de desenvolvimento, avaliadas por cultura”.
- O IQR global marcou 27 das 31 linhas de dendê do desenvolvimento, não “o dendê inteiro”.
- IsolationForest usou contamination=0.10: explicar a escolha desse limiar, sem tratar a quantidade marcada como descoberta independente de uma taxa real de anomalias.
- Valores climáticos extremos não comprovam erros, mas a ausência de erros também não foi validada externamente. Manter os registros por falta de justificativa de exclusão.
- A quarta componente principal é pequena, não exatamente nula. Multicolinearidade pode afetar estabilidade e previsões; não afirmar que a qualidade preditiva “não é afetada”.
- Não inserir explicações agronômicas específicas sem fonte pertinente nem assumir cultivo irrigado, local ou ano ausentes do CSV.

### 6. A explicação para KNN/SVR precisa corresponder ao código

Na célula 60 e no guia, o texto afirma que as colunas one-hot da cultura foram padronizadas. Na célula 54, somente as variáveis numéricas passam por StandardScaler; as categóricas passam por OneHotEncoder.

Corrija essa divergência. A mistura entre culturas no espaço de distâncias pode ser investigada, mas não afirmar causa comprovada sem diagnóstico. Se mantiver essa hipótese, verificar a composição dos vizinhos do KNN ou comparar configurações pertinentes no desenvolvimento. O comportamento do SVR não é demonstrado apenas por observar os vizinhos do KNN.

A importância por permutação da célula 63 é calculada nos dados usados para ajustar a floresta: rotulá-la como diagnóstico no treino. Considerar a correlação entre as variáveis ao interpretar sua ordem; não apresentá-la como evidência independente de generalização ou causalidade.

### 7. Corrigir o tratamento de unidades

O guia (linhas 168–169), o contrato e a tabela de pendências da célula 71 dizem que confirmar as unidades só muda o texto e nenhum número. Isso é incorreto se houver conversão.

Se valores observados e previsões forem convertidos pelo mesmo fator, MAE/RMSE e escalas dos gráficos também mudam. R² e MAPE são invariantes a essa conversão multiplicativa consistente. Se houver mudança nos dados usados para treinar, revisar transformações e parâmetros sensíveis à escala.

Preserve o original e mantenha as unidades como pendência até existir confirmação. Não substituir a pendência por hipótese sobre FAOSTAT, ano ou unidade presumida.

### 8. Respeitar as escolhas ainda abertas do grupo

- GUIA_APRESENTACAO.md, linha 199: remover “Quem pegar essa frente grava o vídeo 1”. Responsáveis pelos vídeos ainda serão escolhidos.
- Remover instruções de cortar/sacrificar os extras no guia e em CONTINUIDADE_EQUIPE.md. O usuário pediu mantê-los em simulação; qualquer mudança de escopo cabe ao grupo.
- Corrigir “AWS e extras não têm uma linha escrita”: já há especificações/documentação; faltam cotações e implementação, respectivamente.
- Enxugar a parte analítica do README para uma introdução e links ao notebook. O enunciado pede evitar repetir ali o relatório inteiro. Manter no README a comparação AWS exigida, o estado dos extras e os vídeos.

## Critérios de conclusão desta revisão

1. Notebook reexecutado do zero, com teste final desativado e dados originais preservados.
2. Se a validação mudar, tabelas, figuras, CSV e textos atualizados a partir dos novos resultados.
3. Comparações por cultura calculadas contra a referência real e com métrica explícita.
4. Teste final parametrizado para usar a alternativa escolhida, sem avaliar o conjunto reservado agora.
5. Narrativa limitada às evidências, sem causalidade ou independência presumidas.
6. Responsáveis a definir e escopo dos extras simulados preservados.
7. Investigar os avisos de encerramento do joblib/loky no Windows; se necessário, usar n_jobs=1 para essa base pequena e confirmar a execução.
8. Entregar resumo do que mudou, resultados novos e pendências por frente.

Este arquivo registra a revisão e as correções solicitadas. Ele não afirma que as correções já foram aplicadas.
