# Ir Além — protótipo simulado no Wokwi

**FarmTech Solutions — Fase 5**
Especificação preparada em 06/09/2026 · Situação: **nada implementado ainda**

---

## ⚠️ Aviso obrigatório — leia antes de tudo

> **Este projeto apresenta um protótipo com ESP32 e sensores simulados no Wokwi. O grupo não
> dispunha de hardware físico. Os testes demonstram o funcionamento do software e da comunicação
> implementada; não constituem coleta física nem validação agronômica. Os requisitos de ESP32 real
> e coleta física do enunciado permanecem não atendidos.**

Este texto deve aparecer, com este sentido, no README principal, nesta pasta, na interface de
visualização e ser dito em voz alta nos vídeos 3 e 4.

O enunciado admite avaliar entregas extras incompletas, mas **a forma de avaliação desta simulação
ainda não foi confirmada com a FIAP**. Não afirme aceitação nem pontuação garantida.

### Requisitos do enunciado não atendidos

| Requisito | Situação | Motivo |
| --- | --- | --- |
| ESP32 real com Wi-Fi funcional | ❌ não atendido | Sem hardware disponível até a entrega |
| Coleta de dados por sensores físicos | ❌ não atendido | Leituras geradas por simulação |
| Validação do modelo com dados reais em tempo real | ❌ não atendido | Sessões simuladas, não medições de plantas |
| Código-fonte comentado | ⏳ a fazer | — |
| Figura da arquitetura | ⏳ a fazer | — |
| Comunicação com um serviço (MQTT) | ⏳ a fazer | Tecnicamente viável no simulador |

**O que a simulação demonstra de fato:** a lógica do firmware, o formato das mensagens, a
comunicação por rede e o encadeamento até a exibição do resultado. Isso tem valor técnico — mas não
é o mesmo que cumprir o requisito de hardware.

---

## Situação atual

**Nada foi implementado.** Esta pasta contém apenas a especificação da arquitetura e o contrato de
mensagens, preparados para quem assumir a frente de IoT/simulação começar sem ter que decidir tudo
do zero.

Responsável: *a definir*.

---

## Arquitetura proposta

```
┌──────────────────────────────────────┐
│  Wokwi (navegador)                   │
│                                      │
│   ESP32 virtual                      │
│    ├── Sensor A (a definir)          │
│    └── Sensor B (a definir)          │
│         │                            │
│         │ Wi-Fi virtual              │
│         │ (rede "Wokwi-GUEST")       │
└─────────┼────────────────────────────┘
          │
          │ MQTT publish  (saída para a internet)
          ▼
┌──────────────────────────────────────┐
│  Broker MQTT público na internet     │
│  tópico: farmtech/<sessao>/leituras  │
└─────────┼────────────────────────────┘
          │
          │ MQTT subscribe
          ▼
┌──────────────────────────────────────┐
│  Computador do grupo (Python)        │
│                                      │
│   assinante  →  CSV / SQLite         │
│                     │                │
│                     ▼                │
│         classificador demonstrativo  │
│                     │                │
│                     ▼                │
│        "Saudável" / "Não saudável"   │
│         (rótulo demonstrativo)       │
└──────────────────────────────────────┘
```

### Por que MQTT, e não HTTP direto

Esta é a restrição técnica que determina a arquitetura:

O gateway público do Wokwi permite que o ESP32 virtual faça conexões **de saída** para a internet,
mas **não** alcança a rede local do computador de quem está simulando. Um `POST` para
`http://localhost:5000` **não vai funcionar**, e depender disso trava o protótipo.

A solução é usar um intermediário público: o ESP32 virtual **publica** no broker, e o assinante
Python no computador **assina** o mesmo broker. Ambos fazem conexões de saída, e nenhuma porta
precisa ser aberta.

Fonte: [documentação do Wokwi sobre ESP32 e Wi-Fi](https://docs.wokwi.com/guides/esp32-wifi).

> Alternativas caso o broker escolhido esteja indisponível: qualquer broker MQTT público, ou um
> broker Mosquitto hospedado em serviço gratuito. **Não** dependa de gateway pago do Wokwi nem de
> abertura de portas no roteador.
>
> **Se a comunicação externa falhar:** registre a falha honestamente e distinga testes locais de
> transmissão real pelo Wokwi. Um teste local não é evidência de que a rede funcionou.

---

## Escolha dos sensores — *a definir*

### Regra que não pode ser violada

O enunciado exige **pelo menos dois sensores distintos**. Isso significa **dois componentes
diferentes**, não duas variáveis do mesmo componente.

- ❌ Temperatura **e** umidade lidas de um único DHT22 → conta como **um** sensor.
- ✅ Um DHT22 **mais** um sensor de umidade de solo → conta como **dois**.

### Critérios para a escolha

1. O componente precisa existir no simulador Wokwi.
2. A variável medida precisa fazer sentido no contexto agrícola da FarmTech.
3. A justificativa técnica é critério de avaliação — escreva-a antes de programar.

Candidatos a avaliar (confirmar disponibilidade no simulador antes de fixar): sensores de
temperatura e umidade do ar, sensores de umidade de solo, sensores de luminosidade, potenciômetros
usados como simulação analógica de uma grandeza contínua.

> ⏳ **PENDENTE:** escolher os dois componentes, verificar no simulador e escrever a justificativa.

---

## Contrato de mensagens — **proposta a confirmar**

> Esta é uma **proposta inicial**. Ela deve ser revisada e confirmada depois que os sensores forem
> escolhidos, porque os campos `sensor`, `variavel` e `unidade` dependem dessa escolha.
> Uma vez confirmada, **não mude o formato no meio da coleta** — mudar o contrato invalida os
> registros já gravados.

### Formato: JSON, uma mensagem por leitura

```json
{
  "horario": "2026-09-07T14:32:05Z",
  "dispositivo": "esp32-wokwi-01",
  "sensor": "<id do componente>",
  "variavel": "<grandeza medida>",
  "valor": 0.0,
  "unidade": "<unidade>",
  "cultura_referencia": "<a definir>",
  "sessao": "sessao-001",
  "origem": "simulado",
  "rotulo": null,
  "proveniencia_rotulo": null
}
```

### Descrição dos campos

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `horario` | string ISO-8601 UTC | sim | Momento da leitura. Se o ESP32 virtual não tiver relógio sincronizado, use o tempo desde o início e **registre isso**; o assinante grava também o horário de recepção. |
| `dispositivo` | string | sim | Identificador do ESP32 virtual. |
| `sensor` | string | sim | Identificador do **componente**. Cada sensor publica com o seu próprio identificador. |
| `variavel` | string | sim | Grandeza medida (ex.: `temperatura_ar`, `umidade_solo`). |
| `valor` | número | sim | Valor lido. |
| `unidade` | string | sim | Unidade da grandeza (ex.: `C`, `%`). |
| `cultura_referencia` | string | sim | Cultura de referência da simulação. |
| `sessao` | string | sim | **Chave de agrupamento.** Separa treino e validação — leituras da mesma sessão nunca podem cair nos dois lados. |
| `origem` | string | sim | Sempre `"simulado"`. Este campo é o que impede que os dados sejam confundidos com coleta real depois. |
| `rotulo` | string ou null | não | `"saudavel"` / `"nao_saudavel"`, quando houver. |
| `proveniencia_rotulo` | string ou null | não | **Como o rótulo foi obtido.** Para regra artificial, descreva a regra. Nunca deixe vazio quando houver rótulo. |

### Campos que o assinante Python acrescenta ao gravar

| Campo | Descrição |
| --- | --- |
| `horario_recepcao` | Horário em que a mensagem chegou ao computador. |
| `topico` | Tópico MQTT de origem. |
| `bruto` | Mensagem original, preservada como recebida. |

**Regra:** os dados brutos nunca são sobrescritos. Limpeza e transformação geram arquivos novos.

---

## Rótulos do classificador — restrições

O Ir Além 2 pede um classificador "Saudável" / "Não saudável". Como não há observação real de
plantas, o rótulo será **demonstrativo**. As restrições abaixo existem para que o trabalho seja
honesto sobre o que mede:

### O que não pode ser feito

- ❌ **Derivar "saúde" do rendimento acima da mediana do `crop_yield.csv`.** São problemas
  diferentes, com bases diferentes. Isso seria inventar um rótulo e apresentá-lo como diagnóstico.
- ❌ **Apresentar dados simulados como coleta física.**
- ❌ **Afirmar validação agronômica** de qualquer natureza.

### O que deve ser feito

- ✅ O classificador tem **base própria e simulada**, gerada pelo protótipo.
- ✅ Se os rótulos vierem de uma regra artificial, **a regra é documentada** e as métricas são
  interpretadas apenas como "o modelo reproduz a regra que geramos" — nunca como capacidade de
  diagnosticar plantas.
- ✅ Definir as condições de cada classe **antes** de gerar os dados, não depois de olhar os
  resultados.
- ✅ Separar treino e validação **por sessão**. Leituras consecutivas da mesma condição não são
  casos independentes: cem leituras de uma mesma situação valem aproximadamente um caso.
- ✅ Relatar matriz de confusão, precisão, recall e F1 **por classe**, com o número de casos.
- ✅ Na interface, usar "classificação simulada". Se exibir os rótulos "Saudável"/"Não saudável"
  exigidos pelo enunciado, manter visível que são demonstrativos.

---

## Estrutura de arquivos planejada

```text
ir_alem/
├── README.md                    # este arquivo
├── firmware/                    # ⏳ código do ESP32 virtual (C/C++ ou MicroPython)
│   ├── main.ino  ou  main.py
│   ├── config_local.h.exemplo   # exemplo SEM credenciais
│   └── diagram.json             # circuito do Wokwi
├── receptor/                    # ⏳ assinante MQTT em Python
│   └── assinante.py
├── modelo/                      # ⏳ classificador demonstrativo
│   └── treinar.py
└── dados/                       # ⏳ sessões simuladas gravadas
    └── .gitkeep
```

> ⚠️ **Credenciais nunca vão para o repositório.** Wi-Fi e broker ficam em `config_local.h` ou
> `config_local.py`, já cobertos pelo `.gitignore`. Versione apenas um arquivo de exemplo, sem
> senhas. Também não envie informações pessoais por um broker público — qualquer pessoa pode assinar
> um tópico público.

---

## Tarefas, na ordem

| # | Tarefa | Critério de pronto |
| --- | --- | --- |
| 1 | Escolher dois componentes sensores distintos e justificar | Justificativa escrita, componentes confirmados no simulador |
| 2 | Montar o circuito no Wokwi e exportar o diagrama | Projeto salvo, figura em `docs/figuras/` |
| 3 | Confirmar o contrato de mensagens com os campos reais | Este documento atualizado |
| 4 | Firmware: Wi-Fi, leitura dos dois sensores, publicação MQTT | Mensagens chegando ao broker, com log |
| 5 | Assinante Python: validação, gravação, horário de recepção | Arquivo de dados com sessões identificadas |
| 6 | Definir as classes e a regra de rotulagem (com a frente ML) | Regra escrita **antes** de gerar os dados |
| 7 | Gerar sessões separadas para treino e validação | Sessões distintas, diversidade verificada |
| 8 | Treinar e avaliar o classificador demonstrativo | Matriz de confusão e métricas por classe |
| 9 | Exibir o resultado na tela | Demonstração funcionando, com aviso de simulação |
| 10 | Figura da arquitetura correspondendo ao que foi implementado | Figura em `docs/figuras/` |

**Ordem de dependência:** o Ir Além 1 (tarefas 1 a 5) funciona sozinho e já demonstra coleta e
comunicação; o Ir Além 2 depende dele. Ambos permanecem no escopo do trabalho. Qualquer decisão
sobre prioridades ou redução de escopo cabe ao grupo.
