# FarmTech Solutions — Fase 5

**Previsão de rendimento de safra com Machine Learning e estimativa de custos em nuvem AWS**

Projeto do PBL da Fase 5 — FIAP, curso de Inteligência Artificial.
Uma fazenda de médio porte (200 hectares) produz quatro culturas. A partir de dados de clima e
rendimento, o trabalho explora tendências por clusterização, investiga cenários discrepantes e
compara cinco algoritmos de regressão para prever a produtividade.

> ### 🚧 Estado do projeto: em desenvolvimento
> A Entrega 1 (ML) está **executada, revisada e com o teste reservado avaliado** (seção 9 do
> notebook). A Entrega 2 (AWS) tem a cotação nas duas regiões concluída, com evidências e
> justificativa. Os protótipos do "Ir Além" têm arquitetura e contrato de mensagens especificados,
> mas **ainda não foram implementados**. Dois dos quatro vídeos já foram publicados (Entregas 1 e 2);
> os dois do "Ir Além" ainda faltam.
> Pendências detalhadas em [`docs/CONTINUIDADE_EQUIPE.md`](docs/CONTINUIDADE_EQUIPE.md).

---

## Sumário

- [Equipe](#equipe)
- [Entrega 1 — Machine Learning](#entrega-1--machine-learning)
- [Entrega 2 — Computação em nuvem (AWS)](#entrega-2--computação-em-nuvem-aws)
- [Ir Além](#ir-além)
- [Vídeos](#vídeos)
- [Instalação e execução](#instalação-e-execução)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Pendências](#pendências)

---

## Equipe

| Integrante | Nome completo | RM |
| --- | --- | --- |
| Coordenador | Lucas Carvalho Cordeiro | RM570388 |
| Larissa | Larissa da Silva Marcelino | RM571790 | 
| Elton | Elton Dias | RM572530 |
| Matheus | Matheus Fontes | RM570457 |

---

## Entrega 1 — Machine Learning

### 👉 [Abrir o notebook: `notebooks/MatheusFontes_rm570457_pbl_fase4.ipynb`](notebooks/MatheusFontes_rm570457_pbl_fase4.ipynb)

**Toda a solução, os resultados e as interpretações estão no notebook.** Ele é o relatório do
trabalho: análise exploratória, clusterização, investigação de outliers, a comparação dos cinco
regressores e a discussão de limitações — com todas as células executadas e as saídas preservadas.
Este README não repete esse conteúdo; ele apenas conduz o leitor até lá.

### O que o notebook contém

| Seção | Conteúdo |
| --- | --- |
| 1–2 | Ambiente, carga da base e verificação de integridade por hash |
| 3 | Contrato de dados, identificador de cenário climático e a dúvida de unidades |
| 4 | Divisão treino/teste agrupada e validação cruzada, com verificações de vazamento |
| 5 | Análise exploratória: distribuições, correlações e multicolinearidade |
| 6 | Clusterização dos cenários climáticos, com escolha justificada do número de grupos |
| 7 | Investigação de outliers climáticos e de rendimento |
| 8 | Comparação dos cinco regressores em validação cruzada aninhada |
| 9 | Avaliação no teste reservado — **executado**, ver seção 9.1 |
| 10 | Conclusões, limitações e pendências |

### A base de dados

`data/crop_yield.csv` — 156 registros, 6 colunas, preservado sem alterações
(SHA-256 `07B3335F...F771B1B`). Quatro culturas (*Cocoa, beans*, *Oil palm fruit*, *Rice, paddy*,
*Rubber, natural*), quatro variáveis climáticas e o rendimento como alvo.

> ⚠️ **Dúvida de unidades, em aberto.** O enunciado declara o rendimento em t/ha, mas os valores vão
> de 5.249 a 203.399 — incompatível com essa leitura. O mesmo ocorre com a precipitação. **Os dados
> não foram convertidos** e as métricas são reportadas na unidade original do arquivo. A confirmação
> com a FIAP está pendente; a seção 3.3 do notebook detalha o que muda em cada cenário de
> esclarecimento.

### Resultado em uma linha

Entre os cinco algoritmos e as duas referências avaliados em validação cruzada aninhada, **a Floresta
Aleatória é a única alternativa com erro menor que o de uma referência que usa apenas a cultura, nas
quatro culturas.** A discussão completa — incluindo por que o R² global não basta para essa
comparação e por que a vantagem ainda não está estatisticamente estabelecida — está na
[seção 8 do notebook](notebooks/MatheusFontes_rm570457_pbl_fase4.ipynb). No teste reservado (seção
9.1), essa vantagem se confirma em arroz e borracha, mas **não em dendê**, que generaliza mal fora do
desenvolvimento — leitura honesta que o notebook mantém, sem reabrir a escolha do modelo.

![Comparação dos modelos](docs/figuras/fig10_comparacao_modelos.png)

Tabelas exportadas: [`docs/resultados_modelos.csv`](docs/resultados_modelos.csv) e
[`docs/resultados_por_cultura.csv`](docs/resultados_por_cultura.csv).
Protocolo de dados e avaliação: [`docs/CONTRATO_DADOS.md`](docs/CONTRATO_DADOS.md).

---

## Entrega 2 — Computação em nuvem (AWS)

As estimativas foram realizadas na AWS Pricing Calculator em 08/09/2026. A
especificação, o checklist e os critérios de evidência estão em
[`docs/aws/README.md`](docs/aws/README.md).

Esta seção comparará o custo de uma máquina Linux On-Demand (100%) entre **São Paulo (`sa-east-1`)**
e **Norte da Virgínia (`us-east-1`)**, com a configuração exigida pelo enunciado:

| Requisito | Valor |
| --- | --- |
| CPUs | 2 |
| Memória | 1 GiB |
| Rede | até 5 Gigabit |
| Armazenamento | 50 GB |

O enunciado faz duas perguntas distintas, que serão respondidas separadamente: **qual é a opção mais
barata** e **qual seria escolhida** considerando acesso rápido aos dados dos sensores e a restrição
legal de armazenamento no exterior.

| Item | Situação |
| --- | --- |
| Cotação São Paulo | ✅ concluída |
| Cotação Norte da Virgínia | ✅ concluída |
| Capturas da calculadora | ✅ concluídas |
| Tabela comparativa de custos | ✅ concluída |
| Justificativa técnica | ✅ concluída |

### Premissas da cotação

As duas regiões foram comparadas com uma instância EC2 `t4g.micro`, Linux,
instância compartilhada, On-Demand, uso contínuo de 730 h/mês e volume EBS
`gp3` de 50 GB. Não foram incluídos snapshots, transferência de dados
adicional, Elastic IP, monitoramento detalhado, IOPS adicionais ou throughput
adicional.

A `t4g.micro` atende aos requisitos do enunciado: 2 vCPU, 1 GiB de memória e
rede de até 5 Gbps. Ela usa processador AWS Graviton (arquitetura ARM); para o
escopo demonstrativo, a aplicação deve ser implantada com dependências
compatíveis com essa arquitetura.

O enunciado usa o termo “HD”, mas os volumes HDD `st1` e `sc1` exigem no mínimo
125 GiB. Por isso, foi adotado EBS `gp3` de 50 GB como armazenamento de uso
geral, sem provisionamento adicional de desempenho.

### Comparação de custos mensais

| Componente | São Paulo (`sa-east-1`) | Norte da Virgínia (`us-east-1`) |
| --- | ---: | ---: |
| Instância EC2 `t4g.micro` | US$ 9,78 | US$ 6,13 |
| Armazenamento EBS `gp3`, 50 GB | US$ 7,60 | US$ 4,00 |
| **Total mensal** | **US$ 17,38** | **US$ 10,13** |

O custo conjunto das duas estimativas é US$ 27,51/mês. São Paulo custa
US$ 7,25 a mais por mês, ou aproximadamente 71,6% acima do custo da Virgínia
do Norte.

Evidências da calculadora: [resumo das duas regiões](docs/aws/calculadora_resumo_duas_regioes.jpg)
e [custos mensais por região](docs/aws/calculadora_custos_por_regiao.jpg).

### Decisão de região

**Solução mais barata:** Norte da Virgínia, com custo mensal estimado de
US$ 10,13.

**Região escolhida para o cenário do projeto:** São Paulo. Embora tenha custo
superior, os dados dos sensores e a API do cenário estão no Brasil; manter a
aplicação na região brasileira reduz a distância de rede e favorece o acesso
rápido aos dados. A escolha também respeita a restrição de armazenamento no
exterior proposta pelo enunciado. Essa é uma premissa do exercício, não uma
afirmação de proibição legal universal.

---

## Ir Além

> ### ⚠️ Aviso obrigatório sobre o escopo dos extras
>
> **Este projeto apresenta um protótipo com ESP32 e sensores simulados no Wokwi. O grupo não
> dispunha de hardware físico. Os testes demonstram o funcionamento do software e da comunicação
> implementada; não constituem coleta física nem validação agronômica. Os requisitos de ESP32 real
> e coleta física do enunciado permanecem não atendidos.**
>
> O enunciado admite avaliar extras incompletos, mas a forma de avaliação dessa simulação ainda não
> foi confirmada com a FIAP. As duas opções **permanecem no escopo** do trabalho.

Arquitetura especificada para as duas opções:

```
ESP32 virtual (Wokwi) + 2 sensores virtuais distintos
        ↓  Wi-Fi simulado
   Broker MQTT público na internet
        ↓
 Assinante Python no computador  →  CSV / SQLite  →  classificador  →  tela
```

O gateway público do Wokwi permite conexões de saída à internet, mas **não** acessa a rede local do
computador — por isso a comunicação passa por um broker MQTT, e não por HTTP direto ao `localhost`.

| Opção | Situação |
| --- | --- |
| **Ir Além 1** — coleta e comunicação | ⏳ arquitetura e contrato de mensagens especificados; **implementação pendente** |
| **Ir Além 2** — classificação de saúde | ⏳ restrições e protocolo de rotulagem definidos; **implementação pendente** |

Especificação completa, contrato de mensagens e restrições de rotulagem em
[`ir_alem/README.md`](ir_alem/README.md).

---

## Vídeos

Quatro vídeos de até 5 minutos, publicados como **não listados** no YouTube.

| # | Vídeo | Link | Situação |
| --- | --- | --- | --- |
| 1 | Entrega 1 — Machine Learning | [Assistir](https://youtu.be/xpcw7nJHQtU) | ✅ Publicado |
| 2 | Entrega 2 — Comparação AWS | [Assistir](https://youtu.be/6Z77Y1DQH4A) | ✅ Publicado |
| 3 | Ir Além 1 — Coleta e comunicação | ⏳ *pendente* | Depende do protótipo |
| 4 | Ir Além 2 — Classificação de saúde | ⏳ *pendente* | Depende do classificador |

> ⏳ **PENDENTE:** os vídeos 3 e 4 ainda não foram gravados. Os links
> serão inseridos quando existirem — nenhuma URL foi inventada aqui.
>
> Nos vídeos 3 e 4 é obrigatório declarar que o hardware é simulado.

---

## Instalação e execução

Ambiente testado em **Windows 11 com Python 3.13.5**, em 08/09/2026.

```powershell
# 1. Clonar e entrar na pasta do projeto
git clone https://github.com/Grupo-IA-FIAP-EMLL-2026-2/fase5-cap1-farmtech.git
cd fase5-cap1-farmtech

# 2. Criar o ambiente virtual
python -m venv .venv

# 3. Ativar
.\.venv\Scripts\Activate.ps1

# 4. Instalar as dependências nas versões testadas
python -m pip install --upgrade pip
python -m pip install -r requirements.txt

# 5. Abrir o notebook
jupyter lab notebooks\MatheusFontes_rm570457_pbl_fase4.ipynb
```

No Jupyter, use **Kernel → Restart Kernel and Run All Cells** para executar do zero.
Tempo aproximado: **cerca de 1 minuto e 20 segundos**.

Para executar sem abrir a interface:

```powershell
.\.venv\Scripts\python.exe -m jupyter nbconvert --to notebook --execute --inplace notebooks\MatheusFontes_rm570457_pbl_fase4.ipynb
```

> Se o PowerShell bloquear a ativação do ambiente, rode uma vez na sessão:
> `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`

O notebook localiza a raiz do projeto sozinho e usa apenas caminhos relativos, então funciona tanto
executado de `notebooks/` quanto da raiz. A execução é determinística: rodar de novo produz os
mesmos números.

---

## Estrutura do repositório

```text
fase5-cap1-farmtech/
├── README.md                          # este arquivo
├── PLANEJAMENTO.md                    # cronograma e acordos do grupo
├── requirements.txt                   # versões efetivamente testadas
├── .gitignore
│
├── data/
│   └── crop_yield.csv                 # base original, preservada e verificada por hash
│
├── notebooks/
│   └── MatheusFontes_rm570457_pbl_fase4.ipynb # ⭐ Entrega 1, executado com as saídas salvas
│
├── docs/
│   ├── CONTRATO_DADOS.md              # protocolo de dados e avaliação (leitura obrigatória)
│   ├── CONTINUIDADE_EQUIPE.md         # tarefas por frente de trabalho
│   ├── GUIA_APRESENTACAO.md           # roteiro para apresentar o projeto ao grupo
│   ├── AJUSTE_IR_ALEM_SIMULADO.md     # mudança de escopo dos extras
│   ├── REVISAO_E_AJUSTES_ETAPA_1.md   # revisão independente e correções aplicadas
│   ├── ENUNCIADO.md                   # cópia do enunciado
│   ├── protocolo_divisao.json         # divisão exata, gerada pelo notebook
│   ├── resultados_modelos.csv         # métricas gerais exportadas pelo notebook
│   ├── resultados_por_cultura.csv     # métricas por cultura exportadas pelo notebook
│   ├── figuras/                       # 12 figuras geradas na execução
│   └── aws/                           # ⏳ checklist pronto; estimativas pendentes
│
└── ir_alem/
    └── README.md                      # arquitetura e contrato de mensagens (implementação pendente)
```

---

## Pendências

Organizadas por frente. Os responsáveis serão definidos pelo grupo.

| Frente | Pendência |
| --- | --- |
| **Coordenação** | Confirmar unidades de `Yield` e `Precipitation` com a FIAP |
| **Coordenação** | Disponibilizar o repositório como público e conferir o acesso antes da entrega |
| **ML** | Classificador demonstrativo do Ir Além 2 |
| **IoT / simulação** | Circuito Wokwi com dois sensores virtuais distintos |
| **IoT / simulação** | Firmware com Wi-Fi e publicação MQTT |
| **IoT / simulação** | Receptor Python e registro das sessões |
| **Vídeos** | Gravar e publicar os quatro vídeos |

Detalhamento, critérios de pronto e dependências em
[`docs/CONTINUIDADE_EQUIPE.md`](docs/CONTINUIDADE_EQUIPE.md).

---

### Requisitos do enunciado ainda **não atendidos**

Registrados de forma explícita, para não haver dúvida:

- ❌ **ESP32 real** — o grupo não dispõe do hardware; o protótipo será simulado no Wokwi.
- ❌ **Coleta física de dados por sensores** — os dados dos extras serão simulados.
- ❌ **Validação agronômica da saúde das plantas** — não há observação real de plantas.

Esses itens dizem respeito apenas às entregas extras ("Ir Além"), que não valem nota e **permanecem
no escopo do trabalho**. As duas entregas obrigatórias seguem com escopo completo.
