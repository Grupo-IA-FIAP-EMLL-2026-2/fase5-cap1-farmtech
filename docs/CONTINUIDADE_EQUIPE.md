# Continuidade do trabalho — instruções por frente

**FarmTech Solutions — Fase 5**
Atualizado em 06/09/2026 (revisado) · Prazo: **08/09/2026, 23:59 (Brasília)** · Meta interna: **08/09, 22h**

---

## Como ler este documento

A divisão de responsabilidades **ainda não foi definida**. O coordenador vai apresentar o que já
existe e, depois disso, os integrantes escolhem suas frentes. Por isso este documento está
organizado **por frente de trabalho**, não por pessoa. Onde aparece "responsável a definir", é
porque a escolha ainda não foi feita.

| Frente | Escopo | Responsável |
| --- | --- | --- |
| **ML** | Revisão dos regressores, fechamento do modelo, teste final, classificador demonstrativo | *a definir* |
| **IoT / simulação** | Protótipo Wokwi, dois sensores virtuais, MQTT, receptor Python | *a definir* |
| **AWS / documentação** | Estimativas nas duas regiões, evidências, justificativa, README | *a definir* |
| **Vídeos** | Quatro vídeos de até 5 minutos | *a definir* (um por entrega) |
| **Coordenação** | Unidades com a FIAP, nomes/RMs, integração, submissão | Coordenador |

---

## ⚠️ Duas mudanças de escopo já confirmadas

**1. Sem ESP32 físico.** O grupo confirmou em 06/09 que não terá o hardware até a entrega. As duas
opções de "Ir Além" passam a ser **protótipos simulados no Wokwi**. Consequência que precisa estar
escrita em todo material: os requisitos de **ESP32 real, coleta física e validação agronômica
permanecem NÃO ATENDIDOS**. Uma simulação funcionando demonstra o software e a comunicação — não
cumpre o requisito de hardware. Detalhes em [`AJUSTE_IR_ALEM_SIMULADO.md`](AJUSTE_IR_ALEM_SIMULADO.md).

**2. As duas entregas obrigatórias seguem com escopo completo.** A Entrega 1 (ML) está executada e
já passou por uma revisão independente, cujas correções foram aplicadas. Da Entrega 2 (AWS) existe a
documentação e o checklist; faltam as cotações.

---

## Antes de qualquer coisa: preparar o ambiente

Testado em Windows 11 com Python 3.12.10, em 06/09/2026.

```powershell
# 1. Na raiz do projeto
cd C:\Users\<seu-usuario>\...\fase5-cap1-farmtech

# 2. Criar o ambiente virtual (só na primeira vez)
python -m venv .venv

# 3. Ativar
.\.venv\Scripts\Activate.ps1

# 4. Instalar as dependências nas versões testadas
python -m pip install --upgrade pip
python -m pip install -r requirements.txt

# 5. Abrir o notebook
jupyter lab notebooks\farmtech_desenvolvimento.ipynb
```

> Se o PowerShell bloquear a ativação, rode uma vez:
> `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`

**Conferência rápida de que está tudo certo** (deve imprimir o hash e `124 / 32`):

```powershell
.\.venv\Scripts\python.exe -c "import hashlib,pandas as pd;from pathlib import Path;print(hashlib.sha256(Path('data/crop_yield.csv').read_bytes()).hexdigest().upper())"
```

Esperado: `07B3335F497E08E705B5835EE334426AC16CB24732C1BFAA994254D39F771B1B`

---

# Frente ML — *responsável a definir*

## O que já está pronto e testado

Tudo na Entrega 1 abaixo já foi **implementado e executado**, não é plano:

| Item | Situação |
| --- | --- |
| Contrato de dados e protocolo agrupado | ✅ implementado e verificado |
| EDA completa com 12 figuras | ✅ executado |
| Clusterização com escolha justificada de k | ✅ executado |
| Investigação de outliers | ✅ executado |
| **Cinco regressores + duas referências**, em validação aninhada | ✅ executado |
| Verificação de sensibilidade com log do alvo | ✅ executado |
| Diagnóstico do mau desempenho do KNN | ✅ executado |
| Avaliação no teste reservado | ⏸️ **travada, esperando você** |

## Arquivos para abrir

1. **[`docs/CONTRATO_DADOS.md`](CONTRATO_DADOS.md)** — comece por aqui e siga todas as regras.
2. **[`notebooks/farmtech_desenvolvimento.ipynb`](../notebooks/farmtech_desenvolvimento.ipynb)** —
   seções 8 e 9 são as suas.
3. **`docs/resultados_modelos.csv`** e **`docs/resultados_por_cultura.csv`** — tabelas exportadas.
4. **`docs/protocolo_divisao.json`** — a divisão exata, para você reproduzir.

## Resultado atual, em uma tabela

Previsões da **validação cruzada aninhada** (5 partições externas × 3 internas), 124 linhas do
desenvolvimento:

| Alternativa | MAE | RMSE | R² | MAPE | Culturas em que supera a referência |
| --- | --- | --- | --- | --- | --- |
| **3. Floresta Aleatória** | **3.597** | 6.804 | 0,990 | 11,2% | **4 de 4** |
| *Ref. média da cultura* | *4.772* | *7.846* | *0,987* | *14,2%* | *referência* |
| 2. Árvore de Decisão | 4.800 | 9.364 | 0,982 | 13,0% | 2 de 4 (arroz, borracha) |
| 1. Regressão Linear | 5.025 | 7.915 | 0,987 | 19,5% | 2 de 4 (dendê, arroz) |
| 4. KNN | 6.759 | 15.723 | 0,948 | 35,4% | 0 de 4 |
| 5. SVR (RBF) | 7.365 | 11.001 | 0,975 | 39,6% | 0 de 4 |
| *Ref. média global (Dummy)* | *58.898* | *69.146* | *−0,000* | *340,8%* | — |

**Leia esta tabela com cuidado:** o R² alto não basta. A referência que só prevê a média da cultura,
sem olhar clima nenhum, já chega a R² = 0,987. O que separa as alternativas é o desempenho **dentro
de cada cultura**, comparado ao da referência. A métrica adotada é o **MAE dentro da cultura**:

| Alternativa (MAE) | Cacau | Dendê | Arroz | Borracha |
| --- | --- | --- | --- | --- |
| **Floresta Aleatória** | **1.113** | **9.418** | **2.508** | **1.348** |
| *Ref. média da cultura* | *1.298* | *12.288* | *3.884* | *1.616* |
| Árvore de Decisão | 1.342 | 13.549 | **2.933** | **1.376** |
| Regressão Linear | 2.215 | **11.973** | **3.385** | 2.526 |
| SVR (RBF) | 5.456 | 13.541 | 5.181 | 5.281 |
| KNN | 5.956 | 12.684 | 5.359 | 3.036 |

⚠️ **Atenção a uma armadilha de leitura:** um R² negativo **não** significa perder para a
referência. O zero do R² é a média da cultura no conjunto avaliado, enquanto a referência usa médias
aprendidas no treino de cada partição — por isso a própria referência tem R² negativo. A regressão
linear, por exemplo, tem R² = −0,013 no dendê e **supera** a referência, cujo R² ali é −0,039.

## Suas tarefas

### Tarefa ML-1 — Revisar a seção 8 (prioridade máxima)

Você precisa **entender e conseguir explicar** o código antes de assumi-lo. Confira em especial:

- [ ] O pipeline não vaza: escalonamento e codificação são reajustados em cada partição.
- [ ] O SVR usa `TransformedTargetRegressor` para escalar `y` **dentro** da partição.
- [ ] Todas as sete alternativas usam a mesma lista `PARTICOES` no nível externo.
- [ ] A busca de hiperparâmetros roda nas partições **internas**, formadas só com o treino externo.
- [ ] As comparações com a referência são geradas por código, não digitadas à mão.

**O que já foi corrigido e por quê.** Uma versão anterior fazia a busca de hiperparâmetros nas
mesmas partições usadas para reportar as métricas. Isso deixava os números otimistas: a árvore de
decisão aparecia com MAE 4.340, **abaixo** da referência, e com a validação aninhada passou a 4.800,
**acima** dela. A ordem entre alternativas mudou — vale a pena olhar isso antes de confiar em
qualquer comparação apertada.

**Ponto que ainda exige atenção:** o aninhamento removeu o viés da escolha de *hiperparâmetros*, mas
escolher a alternativa vencedora olhando estas mesmas métricas continua sendo uma seleção. Só o
teste reservado resolve isso, e ele é usado uma vez, depois de você fechar a escolha.

**Critério de pronto:** você consegue explicar, sem consultar o notebook, por que o R² global de
0,99 não é suficiente para dizer que os modelos entenderam o clima.

### Tarefa ML-2 — Avaliar a recomendação do log do alvo

A seção 8.5 já testou transformar o alvo em log, **com o mesmo desenho aninhado**:

| Modelo | MAE (escala original) | MAE (com log) | Culturas superadas por MAE | Por R² |
| --- | --- | --- | --- | --- |
| Regressão Linear | 5.025 | **4.571** | **4 de 4** | 2 de 4 |
| SVR | 7.365 | 7.161 | 0 de 4 | 0 de 4 |
| KNN | 6.759 | 6.611 | 0 de 4 | 0 de 4 |

**Aqui as duas métricas discordam, e a decisão é sua.** A regressão linear com log tem MAE menor que
o da referência nas quatro culturas, mas R² maior em apenas duas — porque o log reduz os erros
típicos e mantém alguns erros grandes, que o R² penaliza mais. Nenhuma das variantes alcança a
Floresta Aleatória (MAE 3.597).

**Decisões suas:** (a) incorporar o log à comparação principal ou mantê-lo como verificação de
sensibilidade; (b) qual métrica governa a escolha final.

Caminhos ainda não testados, se sobrar tempo: reduzir a mistura de culturas diagnosticada na seção
8.4 (ampliando o peso das colunas de cultura no espaço de distâncias, ou ajustando um modelo por
cultura), e fazer um diagnóstico próprio do SVR — o da seção 8.4 explica o KNN, não o SVR.

**Critério de pronto:** decisão registrada em markdown no notebook, com justificativa e métrica
declarada.

### Tarefa ML-3 — Fechar a escolha e liberar o teste final

Depois de ML-1 e ML-2, e **só depois**:

1. Registre a escolha em `ALTERNATIVA_FINAL`, na seção 9. O catálogo `CONFIGURACOES_FINAIS` já traz
   as cinco alternativas e as três variantes com log, cada uma com pipeline e grade prontos.
   Enquanto `ALTERNATIVA_FINAL` for `None`, a avaliação falha de propósito, com mensagem explícita.
2. Confirme com o grupo que nenhuma outra alternativa será comparada depois.
3. Mude `EXECUTAR_TESTE_FINAL = False` para `True`.
4. Execute o notebook inteiro: `Kernel > Restart Kernel and Run All Cells`.
5. Escreva a interpretação do resultado.

> ⚠️ **O que invalida o teste.** Reexecutar o notebook e obter os mesmos números não invalida nada —
> o procedimento é determinístico, e o professor precisa conseguir rodar tudo na correção. O que
> invalida é **olhar o resultado do teste e então mudar de modelo**. Se isso acontecer, declare no
> notebook em vez de esconder.

**Critério de pronto:** seção 9 executada com a alternativa escolhida registrada, métricas gerais e
por cultura, e um parágrafo comparando o desempenho no teste com o da validação.

### Tarefa ML-4 — Classificador demonstrativo (Ir Além 2)

Depende da frente IoT ter dados. Ver [`ir_alem/README.md`](../ir_alem/README.md).

**Restrições que não podem ser violadas:**

- ❌ **Não** derivar "saúde" do rendimento acima da mediana do `crop_yield.csv`. São problemas
  diferentes, e isso seria inventar um rótulo.
- ❌ **Não** apresentar dados simulados como coleta física.
- ✅ A base do classificador é **própria e simulada**, gerada pelo protótipo Wokwi.
- ✅ Se os rótulos vierem de regras artificiais, **documente as regras** e limite a interpretação:
  as métricas medem se o modelo reproduz a regra, **não** se ele diagnostica plantas.
- ✅ Separe treino e teste **por sessão simulada**, nunca por leitura individual — leituras
  consecutivas da mesma condição não são casos independentes.
- ✅ Relate matriz de confusão, precisão, recall e F1 por classe, com o número de casos.

**Critério de pronto:** classificador treinado, avaliado em sessões simuladas separadas, com uma
seção de limitações que diz claramente que não há validação agronômica.

---

# Frente IoT / simulação — *responsável a definir*

## Situação de partida

**A arquitetura e o contrato de mensagens já estão especificados** em
[`ir_alem/README.md`](../ir_alem/README.md), incluindo a restrição técnica do gateway do Wokwi e as
restrições de rotulagem. **O código ainda não foi escrito** — firmware, receptor e classificador são
tarefas suas.

## Arquitetura definida

```
ESP32 virtual (Wokwi) + 2 sensores virtuais
        ↓ Wi-Fi simulado do Wokwi
   Broker MQTT público na internet
        ↓
 Assinante Python no computador
        ↓
   CSV / SQLite  →  classificador demonstrativo  →  tela
```

**Restrição técnica importante, já verificada:** o gateway público do Wokwi permite conexões de
**saída** para a internet, mas **não** alcança a rede local do seu computador. Por isso o ESP32
virtual **não** consegue fazer HTTP direto para `localhost`. A solução é o broker MQTT público: o
Wokwi publica, o seu Python assina o mesmo broker.
Fonte: [documentação do Wokwi](https://docs.wokwi.com/guides/esp32-wifi).

## Suas tarefas

### Tarefa IoT-1 — Montar o circuito no Wokwi

- [ ] Escolher **dois componentes sensores distintos** suportados pelo simulador.
      ⚠️ Duas variáveis de um único sensor (por exemplo, temperatura e umidade do DHT22)
      **não** contam como dois sensores. Precisa ser dois componentes.
- [ ] Justificar por escrito a relação de cada sensor com o contexto agrícola do projeto.
- [ ] Salvar o projeto e exportar o diagrama para `docs/figuras/`.

**Critério de pronto:** projeto Wokwi salvo, link registrado no README, dois componentes distintos
visíveis no diagrama, justificativa escrita.

### Tarefa IoT-2 — Firmware com Wi-Fi e MQTT

- [ ] Conectar ao Wi-Fi virtual (`Wokwi-GUEST`).
- [ ] Publicar leituras no broker seguindo **exatamente** o contrato de mensagens do
      `ir_alem/README.md` (inclusive o campo `origem: "simulado"`).
- [ ] Comentar o código linha a linha — é critério de avaliação do enunciado.
- [ ] Tratar reconexão quando a rede cair.
- [ ] Credenciais em arquivo separado, fora do repositório (já coberto pelo `.gitignore`).

**Critério de pronto:** mensagens chegando ao broker, com log de horário e conteúdo.

### Tarefa IoT-3 — Receptor Python

- [ ] Assinar o tópico, validar o formato das mensagens e gravar em CSV ou SQLite.
- [ ] Registrar o horário de recepção e a origem simulada em **todo** registro.
- [ ] Guardar os dados brutos, sem sobrescrever.

**Critério de pronto:** arquivo de dados com sessões identificadas, pronto para a frente ML.

### Tarefa IoT-4 — Gerar as sessões para o classificador

Combine com a frente ML antes de gerar:

- [ ] Cultura de referência definida.
- [ ] Condições que representam cada classe, e a **regra** que gera o rótulo, escritas antes.
- [ ] Sessões separadas para treino e para validação.
- [ ] Diversidade suficiente: muitas leituras da mesma condição não equivalem a muitos casos.

> ⚠️ **Se a comunicação externa falhar**, registre a falha honestamente e distinga testes locais de
> transmissão real pelo Wokwi. Não apresente um teste local como prova de que a rede funcionou.

---

# Frente AWS / documentação — *responsável a definir*

## Situação de partida

A **documentação da frente já existe**: `docs/aws/README.md` traz a especificação exigida, o
checklist completo, as instâncias candidatas já verificadas e as duas armadilhas mapeadas. A seção
da Entrega 2 no README também está estruturada, com os campos a preencher.

**O que falta é a execução:** as cotações na calculadora oficial, as capturas e a redação da
justificativa. Não há preço, captura nem link neste repositório — e nenhum deve ser inventado.

## Especificação exigida pelo enunciado

| Requisito | Valor |
| --- | --- |
| CPUs | 2 |
| Memória | 1 GiB |
| Rede | até 5 Gbps |
| Armazenamento | 50 GB ("HD") |
| Sistema | Linux |
| Modalidade | On-Demand 100% |
| Regiões | São Paulo (`sa-east-1`) **e** Norte da Virgínia (`us-east-1`) |

## Suas tarefas

### Tarefa AWS-1 — Cotar nas duas regiões

- [ ] Pesquisar a instância compatível **antes** de cotar. Candidatas verificadas em 05/09:
      `t3.micro`, `t3a.micro`, `t4g.micro` (2 vCPUs, 1 GiB, rede até 5 Gbps em burst).
      ⚠️ `t4g` é ARM — confira compatibilidade e disponibilidade nas duas regiões.
- [ ] Registrar que essas famílias usam **créditos de CPU** e declarar a premissa de uso.
- [ ] Usar **premissas idênticas** nas duas regiões: horas mensais, SO, modalidade, armazenamento.
- [ ] Registrar data da cotação, moeda, preço de cada componente e total mensal.

**Ponto de atenção sobre o "HD":** o enunciado diz "HD", mas 50 GB não obriga SSD.
`st1` e `sc1` têm mínimo de 125 GiB, enquanto o volume magnético `standard` aceita de 1 GiB a 1 TiB.
Se optar por `gp3`, **justifique** a interpretação de "HD" como armazenamento genérico e registre o
tipo efetivamente cotado. Não escolha em silêncio.
Fonte: [tipos de volumes EBS](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html).

**Critério de pronto:** duas estimativas salvas, capturas em `docs/aws/`, planilha ou tabela com os
componentes e o total de cada região.

### Tarefa AWS-2 — Justificativa técnica (peso 2,5 do barema)

O enunciado faz **duas perguntas diferentes**. Responda as duas separadamente:

1. **Qual é a mais barata?** → resposta puramente de preço.
2. **E considerando acesso rápido aos dados e restrição legal de armazenamento no exterior?**
   → aqui entram latência e a restrição dada pelo exercício.

⚠️ Trate a restrição legal como **premissa do enunciado**, não como proibição legal universal.
Não invente medições de latência: argumente por distância geográfica e caminho de rede.

**Critério de pronto:** as duas perguntas respondidas em separado, com números da sua própria
cotação.

### Tarefa AWS-3 — Consolidar o README

O README já tem a estrutura montada, com marcadores `> ⏳ PENDENTE` onde falta conteúdo. Preencha
as seções de AWS e revise o conjunto. **Não invente URLs, preços ou capturas.**

---

# Frente de vídeos — *responsável a definir*

Quatro vídeos, **não listados** no YouTube, de até 5 minutos cada (mire em 4min30). Os links vão no
README. Quem grava cada um ainda não foi definido.

| Vídeo | Conteúdo indispensável | Depende de |
| --- | --- | --- |
| **1. Entrega 1 (ML)** | Base, EDA, clusters, outliers, cinco modelos, métricas e conclusões | ML-3 concluída |
| **2. Entrega 2 (AWS)** | Calculadora, configuração, duas regiões, valores e justificativa | AWS-2 concluída |
| **3. Ir Além 1** | Circuito Wokwi, dois sensores, Wi-Fi, MQTT, dados chegando — **dizendo que é simulação** | IoT-3 concluída |
| **4. Ir Além 2** | Cultura, dados simulados, regra dos rótulos, treino, validação, classificação na tela | ML-4 concluída |

> ⚠️ Nos vídeos 3 e 4 é **obrigatório** dizer em voz alta que o hardware é simulado e que o
> requisito de ESP32 físico não foi atendido. Apresentar simulação como coleta real seria
> informação falsa numa entrega acadêmica.

Roteiro sugerido para o vídeo 1, já que o material existe:

1. (30s) Base: 156 linhas, 4 culturas, mas só 39 cenários climáticos.
2. (60s) EDA: a cultura responde por 98,8% da variação — mostrar Figura 1 e Figura 3.
3. (60s) Clusters: três regimes, escolha por estabilidade — Figuras 5 e 6.
4. (30s) Outliers: nenhum rendimento marcado pelos dois critérios; contraexemplo da Figura 9.
5. (75s) Modelos: Figura 10, a comparação contra a referência da média da cultura, em validação
   aninhada.
6. (30s) Limitações e o que ficou pendente.

---

# Frente de coordenação

| # | Tarefa | Bloqueia |
| --- | --- | --- |
| 1 | Confirmar com a FIAP as unidades de `Yield` e `Precipitation` | Só o texto das métricas |
| 2 | Reunir nomes completos e RMs dos quatro integrantes | Nomeação do notebook e README |
| 3 | **Renomear** `farmtech_desenvolvimento.ipynb` → `NomeCompleto_rmXXXXX_pbl_fase4.ipynb` | Barema |
| 4 | Confirmar no portal a inconsistência "Fase 5" com sufixo `pbl_fase4.ipynb` | Nomeação |
| 5 | Criar e publicar o repositório remoto público | Entrega |
| 6 | Integrar o material das frentes no notebook e no README | Entrega |
| 7 | Submeter no portal e guardar comprovante — **sem commits depois** | Prazo |

---

## Dependências entre as tarefas

```
ML-1 (revisar) ──► ML-2 (log do alvo) ──► ML-3 (teste final) ──► Vídeo 1
                                                    │
                                                    └──► Integração final

IoT-1 (Wokwi) ──► IoT-2 (MQTT) ──► IoT-3 (receptor) ──► Vídeo 3
                                          │
                                          └──► IoT-4 (sessões) ──► ML-4 ──► Vídeo 4

AWS-1 (cotar) ──► AWS-2 (justificar) ──► AWS-3 (README) ──► Vídeo 2

Coordenação-2 (nomes/RMs) ──► Coordenação-3 (renomear) ──► Coordenação-7 (submeter)
```

**Caminho crítico:** `IoT-1 → IoT-2 → IoT-3 → IoT-4 → ML-4 → Vídeo 4`. É a cadeia mais longa e a de
maior risco técnico — convém acompanhá-la de perto nos alinhamentos.

**Frentes que podem começar imediatamente:** AWS e ML não dependem de ninguém.

---

## O que NÃO pode ser afirmado em nenhum material

Lista de verificação antes de publicar qualquer texto, vídeo ou figura:

- ❌ "Rendimento em toneladas por hectare" — a unidade não foi confirmada.
- ❌ "Os 39 cenários são 39 anos" — a base não tem coluna de data.
- ❌ "O clima causa o rendimento" — há associação, não causalidade demonstrada.
- ❌ "Modelo X é o melhor" com base no teste — o teste ainda não foi executado.
- ❌ "Os 39 cenários são observações independentes" — não há como verificar, sem data ou origem.
- ❌ "O clima não afeta o cacau e o dendê" — as associações foram fracas *nesta amostra*, o que não
  demonstra ausência de efeito.
- ❌ "Coletamos dados dos sensores" — os dados são simulados no Wokwi.
- ❌ "Validamos a saúde das plantas" — não há observação agronômica real.
- ❌ "O ESP32 foi testado" — não há hardware físico.
- ❌ Qualquer preço, URL ou captura da AWS que não tenha sido efetivamente obtido.
