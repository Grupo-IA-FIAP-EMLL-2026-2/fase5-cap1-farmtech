# Roteiro — Vídeo Entrega 1 (Machine Learning)

**Duração alvo:** até 5 minutos
**Narração:** Matheus
**Formato:** gravação de tela rolando pelo notebook + narração por cima

> ✅ **Confirmado em 08/09/2026:** `ALTERNATIVA_FINAL` está definida como **Floresta Aleatória** e a
> Seção 9 já tem o teste reservado executado (ver Seção 9.1 do notebook). Os valores abaixo já foram
> preenchidos com o que apareceu de fato na saída das células — confira na tela antes de gravar, pois
> o notebook pode ter sido reexecutado depois deste roteiro.

---

## 0:00 – 0:20 | Abertura

**Fala:**
> "Neste vídeo, vamos demonstrar o notebook de Machine Learning do projeto FarmTech: como os dados foram explorados, os modelos comparados, e o resultado final."

**Mostrar:** capa do notebook ou título da Seção 1

---

## 0:20 – 1:00 | Contexto e dados

**Fala:**
> "Trabalhamos com 156 registros de quatro culturas — cacau, dendê, arroz e borracha — e quatro variáveis climáticas, prevendo o rendimento da safra. Os dados foram verificados por hash para garantir que não foram alterados durante o projeto."

**Mostrar (gravação de tela):** rolar pela Seção 1-2 (carga e verificação de integridade) e Seção 3 (contrato de dados)

---

## 1:00 – 1:40 | Análise exploratória e clusterização

**Fala:**
> "Na análise exploratória, olhamos distribuições, correlações e multicolinearidade entre as variáveis. Em seguida, agrupamos os cenários climáticos por semelhança, o que nos ajudou a identificar padrões e também alguns cenários fora do padrão — os outliers."

**Mostrar:** rolar pela Seção 5 (gráficos de EDA) e Seção 6 (clusters), pausando alguns segundos em cada gráfico

---

## 1:40 – 2:10 | Outliers

**Fala:**
> "Identificamos cenários climáticos e de rendimento fora do padrão esperado, sem removê-los indevidamente — cada ponto foi investigado antes de qualquer decisão."

**Mostrar:** Seção 7 (outliers)

---

## 2:10 – 3:10 | Comparação dos 5 modelos

**Fala:**
> "Comparamos cinco algoritmos de regressão em validação cruzada aninhada, sempre contra duas referências: uma que só usa a média geral, e outra mais rigorosa, que usa a média de cada cultura. A **Floresta Aleatória** foi a única alternativa que superou a referência por cultura nas quatro culturas avaliadas, com MAE de 3.597 contra 4.772 da referência."

**Mostrar:** Seção 8 (tabela/gráfico comparando os 5 modelos — Figura 10), destacando a linha da Floresta Aleatória

---

## 3:10 – 4:00 | Teste final (a parte mais importante)

**Fala:**
> "Com o modelo escolhido — Floresta Aleatória — fechado antes de olhar os dados reservados, executamos o teste final sobre os 32 registros que nunca haviam sido vistos. O resultado geral foi: MAE de 5.049,9, RMSE de 9.494,1, R² de 0,984 e MAPE de 9,5%."

**Mostrar:** Seção 9 rolando, com a saída/resultado visível na tela (isso é o que mais precisa aparecer gravado de verdade — não pode ser só falado)

---

## 4:00 – 4:40 | Conclusões e limitações

**Fala:**
> "Por cultura, o teste confirmou a vantagem da Floresta Aleatória em arroz e borracha, mas revelou uma generalização ruim no dendê — um resultado que mantivemos registrado como está, sem voltar atrás na escolha do modelo, porque essa é justamente a garantia que o teste reservado deveria oferecer. Entre as demais limitações: a unidade declarada no enunciado para rendimento e precipitação não é compatível com os valores do arquivo, então reportamos os resultados na unidade original, sem conversão. Além disso, os 39 cenários climáticos podem não ser totalmente independentes entre si, já que a base não tem informação de data ou localização."

**Mostrar:** Seção 9.1 (leitura do resultado no teste) e Seção 10 (conclusões)

---

## 4:40 – 5:00 | Encerramento

**Fala:**
> "Esse é o núcleo de Machine Learning do FarmTech: dados explorados com cuidado, modelos comparados de forma rigorosa, e um resultado final validado sem viés de seleção — inclusive quando esse resultado não é uniformemente bom. Obrigado!"

---

## Checklist antes de gravar

- [x] Confirmar `ALTERNATIVA_FINAL` preenchido no notebook (Floresta Aleatória, não `None`)
- [x] Confirmar que a Seção 9 já tem saída executada
- [x] Preencher os campos de valores acima com os resultados vistos na tela
- [ ] Gravar tela rolando pelas seções na ordem do roteiro (não precisa ler código linha a linha, só mostrar que existe e que rodou)
- [ ] Narração até 5 minutos
- [ ] Publicar como "não listado" no YouTube
- [ ] Colar o link na tabela de Vídeos do README
