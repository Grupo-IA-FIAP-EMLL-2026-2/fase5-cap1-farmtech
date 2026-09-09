# Entrega 2 — Estimativa de custos AWS

**Situação: ⏳ não iniciada.** Esta pasta está preparada para receber as evidências.
Responsável: *a definir* (frente AWS/documentação).

**Nenhum preço, link ou captura foi incluído aqui**, porque nenhum foi obtido ainda. Não preencha
com valores de memória ou de outra fonte: a cotação precisa vir da calculadora oficial, com data.

---

## O que o enunciado pede

Estimativa de custos **On-Demand 100%** para uma máquina **Linux**, comparando **São Paulo (BR)** e
**Norte da Virgínia (EUA)**. A máquina hospedaria a API que recebe dados dos sensores e roda o
modelo de Machine Learning.

| Requisito | Valor |
| --- | --- |
| CPUs | 2 |
| Memória | 1 GiB |
| Rede | até 5 Gigabit |
| Armazenamento | 50 GB ("HD") |

Duas perguntas a responder **separadamente**:

1. Qual é a solução **mais barata**?
2. Considerando **acesso rápido aos dados** e **restrição legal de armazenamento no exterior**,
   qual você escolheria? Justifique.

---

## Checklist

### Instância

- [ ] Pesquisar a instância compatível **antes** de cotar; registrar tipo, arquitetura e desempenho
      de rede.
- [ ] Candidatas verificadas em documentação oficial em 05/09/2026: `t3.micro`, `t3a.micro`,
      `t4g.micro` — 2 vCPUs, 1 GiB, rede até 5 Gbps em burst.
      Fontes: [T3/T3a](https://aws.amazon.com/ec2/instance-types/t3/) · [T4g](https://aws.amazon.com/ec2/instance-types/t4/)
- [ ] ⚠️ `t4g` usa **ARM** — conferir compatibilidade da aplicação e disponibilidade nas duas regiões.
- [ ] Registrar que a família T usa **créditos de CPU** e declarar a premissa de uso (e eventual
      custo de créditos excedentes).

### Armazenamento — ponto de atenção

O enunciado diz "HD", mas **50 GB não obriga o uso de SSD**:

- `st1` e `sc1` (HDD) têm **mínimo de 125 GiB** — não servem para 50 GB.
- O volume magnético de geração anterior `standard` aceita **de 1 GiB a 1 TiB** — serve.
- `gp3` (SSD) também serve.

- [ ] Conferir quais opções a calculadora oferece em cada região.
- [ ] Se adotar `gp3`, **justificar** a interpretação de "HD" como armazenamento genérico.
- [ ] Registrar o tipo de volume efetivamente cotado. **Não escolher em silêncio.**

Fonte: [tipos de volumes EBS](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)

### Premissas — idênticas nas duas regiões

- [ ] Horas mensais (ex.: 730 h = uso contínuo) — declarar qual foi usada
- [ ] Sistema operacional (Linux)
- [ ] Modalidade de compra (On-Demand 100%)
- [ ] Tipo e tamanho do volume
- [ ] Itens **incluídos** e **excluídos** da estimativa (transferência de dados, IP elástico,
      snapshots, monitoramento)

### Evidências a salvar nesta pasta

- [ ] `sp_calculadora.png` — captura da estimativa de São Paulo
- [ ] `virginia_calculadora.png` — captura da estimativa da Virgínia do Norte
- [ ] `comparacao_custos.png` — gráfico ou tabela comparativa
- [ ] `estimativa.json` ou `.csv` — exportação da calculadora, se disponível
- [ ] Data da cotação, moeda e câmbio, se aplicável

### Registro dos valores

Preencher **somente com valores efetivamente cotados**:

| Componente | São Paulo (`sa-east-1`) | Norte da Virgínia (`us-east-1`) |
| --- | --- | --- |
| Instância EC2 (mensal) | ⏳ | ⏳ |
| Armazenamento EBS 50 GB (mensal) | ⏳ | ⏳ |
| **Total mensal** | ⏳ | ⏳ |
| Diferença percentual | ⏳ | |

- Data da cotação: ⏳
- Tipo de instância: ⏳
- Tipo de volume: ⏳
- Horas/mês assumidas: ⏳
- Moeda: ⏳

---

## Justificativa técnica (peso 2,5 do barema)

Estruture a resposta em três partes:

**1. Custo.** Qual região é mais barata, com os números da sua cotação.

**2. Acesso rápido aos dados.** Argumente por distância geográfica e caminho de rede entre os
sensores (no Brasil) e a região.
⚠️ **Não invente medições de latência.** Se não mediu, não cite número em milissegundos.

**3. Restrição legal de armazenamento no exterior.** Trate como **premissa dada pelo exercício**,
não como proibição legal universal. A pergunta 2 do enunciado é uma hipótese de trabalho.

> Mantenha as respostas 1 e 2 separadas: a região mais barata pode não ser a escolhida sob as
> restrições. Essa distinção é justamente o que o barema avalia.

---

## Critério de pronto

- [ ] Duas cotações obtidas na calculadora oficial, com capturas
- [ ] Tabela comparativa preenchida com valores reais e data
- [ ] Premissas idênticas e declaradas
- [ ] Tipo de volume justificado
- [ ] As duas perguntas do enunciado respondidas separadamente
- [ ] Seção da Entrega 2 do README preenchida, com imagens
- [x] Vídeo 2 gravado, não listado, com o link no README
