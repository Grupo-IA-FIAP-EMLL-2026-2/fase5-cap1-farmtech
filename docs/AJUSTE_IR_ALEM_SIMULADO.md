# Atualização para o Claude Code — Ir Além simulado

O grupo informou em 06/09/2026 que não tem ESP32 físico e não conseguirá adquiri-lo até a entrega de 08/09. Esta atualização substitui as premissas anteriores de hardware e coleta física.

## Orientação

1. Continue normalmente a Entrega 1 com o CSV original, EDA, clusters/outliers e cinco regressores. Preserve o protocolo agrupado, a dúvida de unidades e o teste final reservado.
2. A Entrega 2 continua sendo a comparação AWS completa na calculadora, com evidências e justificativa.
3. Planeje as duas opções de Ir Além como protótipos simulados no Wokwi. O requisito de ESP32 real continua não atendido. Não afirmar aceitação pela FIAP nem garantir pontuação dos extras.
4. Use ESP32 virtual e dois componentes sensores distintos. Registre a origem simulada das leituras nos dados, textos, interface e vídeos.
5. Arquitetura sugerida: Wokwi → broker MQTT acessível pela internet → assinante Python no computador → armazenamento → modelo demonstrativo → tela.
6. O gateway público do Wokwi acessa a internet, mas não a rede local. Não depender de HTTP direto ao localhost, de gateway pago ou de abertura de portas para concluir o protótipo.
7. O classificador deve ter uma base simulada própria. Não extrair “saúde” da mediana de rendimento do crop_yield.csv. Se os rótulos forem produzidos por regras, documentar as regras e limitar a interpretação das métricas à simulação. Não afirmar validação real da saúde de plantas.
8. Manter a documentação dos requisitos faltantes: hardware físico, coleta física e validação de saúde com observações reais.
9. Larissa, Elton e Matheus escolherão suas frentes. Use ML, IoT e AWS/documentação como rótulos provisórios, sem fixar nomes.
10. Atualize somente arquivos pertinentes à sua execução, preservando o trabalho já concluído. Informe a mudança de escopo no resumo final.

Esta orientação ajusta os extras, sem exigir que a etapa inicial implemente tudo de uma vez. Conclua o escopo da etapa 1 e deixe interfaces e tarefas restantes claras.

Referência técnica: o [Wokwi documenta MQTT e o limite de acesso à rede local pelo gateway público](https://docs.wokwi.com/guides/esp32-wifi).

## Texto para a documentação dos extras

“Este projeto apresenta um protótipo com ESP32 e sensores simulados no Wokwi. O grupo não dispunha de hardware físico. Os testes demonstram o funcionamento do software e da comunicação implementada; não constituem coleta física nem validação agronômica. Os requisitos de ESP32 real e coleta física do enunciado permanecem não atendidos.”

O enunciado admite avaliar extras incompletos, mas a forma de avaliação dessa simulação ainda não foi confirmada com a FIAP.
