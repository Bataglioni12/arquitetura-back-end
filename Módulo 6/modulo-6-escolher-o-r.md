# Módulo 6 — Escolher o R de cada aplicação do portfólio

Recomendação à diretoria. Fatos do caso aparecem como fatos; o que é hipótese está marcado como **hipótese**.

## Resumo para comparar as três decisões

| Aplicação | R escolhido | Dois fatos que o sustentam | Alternativa descartada |
|---|---|---|---|
| Sistema de faturamento | **Rehost** | Ninguém da equipe domina a linguagem; funciona e não muda há quatro anos | Retire (quase escolhido) |
| Portal de agendamento | **Replatform** | Satura no pico de segunda e derruba requisições; a equipe conhece o código e o banco é padrão de mercado | Refactor (adiado, não descartado) |
| Controle de estoque da farmácia | **Retire** | O ERP já cobre 3 das 5 funções; as outras 2 são relatórios impressos uma vez por mês | Rehost |

## 1. Sistema de faturamento

Escolho **rehost**: mover o sistema como está, dos dois servidores físicos para máquinas virtuais na nuvem, sem alterar o código. Dois fatos sustentam isso. Primeiro, ninguém da equipe atual domina a linguagem em que ele foi escrito, então qualquer mudança de código seria feita às cegas. Segundo, ele funciona e não recebe mudança funcional há quatro anos: não há problema de comportamento a corrigir, só um problema de lugar, causado pela restrição de que o data center precisa ser esvaziado em dezoito meses.

O R que quase escolhi foi **retire**, porque um sistema estável e parado há quatro anos parece candidato natural a desligamento. Ele é desqualificado por dois fatos: a auditoria externa exige que o histórico continue consultável por mais dois anos (vinte e quatro meses, mais que os dezoito do prazo), e nenhum outro sistema sabe produzir os relatórios que ele produz. Desligar agora tiraria da diretoria uma obrigação de auditoria. Retire fica como decisão futura, para depois de encerrada a janela de auditoria, e o rehost não impede isso.

**Hipótese a validar cedo:** que uma pilha de dezoito anos, hoje em hardware físico, roda estável em máquina virtual (dependências de sistema operacional antigo, licenças presas a hardware, drivers). O caso não informa isso, e é o principal risco de prazo desta decisão.

## 2. Portal de agendamento

Escolho **replatform**: levar o monolito para um ambiente de execução na nuvem com escala automática e banco relacional gerenciado, com alteração mínima de código. Os fatos que sustentam: o problema declarado é saturação no pico de segunda de manhã, que é um problema de capacidade; e a equipe conhece o código e o banco é padrão de mercado, o que torna a troca de plataforma barata e previsível para uma equipe de cinco pessoas que também cuida das outras duas aplicações.

**Por que replatform e refactor dão resultados diferentes no pico.** Replatform muda *onde e como* o sistema roda: com escala automática, sobe mais cópias do monolito inteiro quando a demanda chega. Isso resolve o pico se o monolito aceitar várias cópias ao mesmo tempo (**hipótese**: sessões e estado em memória podem impedir isso e precisam ser testados). O custo é escalar tudo junto, inclusive as partes que não estão sob pressão, e o gargalo pode estar no banco, não na aplicação. Refactor muda *como o código é organizado* sem mudar o comportamento externo: permitiria isolar o caminho quente (a consulta de horários) e escalar só ele, com resultado mais fino, mas com muito mais esforço e risco durante o horário de atendimento.

**A publicação de seis horas** é resolvida pelo **refactor**, não pelo replatform. Seis horas é propriedade do artefato publicado e do processo de publicação, e trocar o lugar onde o mesmo artefato roda não muda o que precisa ser publicado. Um replatform bem feito pode encurtar a parte causada pelo ambiente manual (**hipótese**: o caso não diz por que leva seis horas), mas a redução estrutural exige quebrar o monolito em partes publicáveis separadamente. Aceito isso como risco: como a publicação é mensal, de madrugada, e o sintoma que derruba requisições em horário de atendimento é o pico, o replatform ataca primeiro o problema mais caro, e o refactor entra depois, se a medição mostrar que a publicação ainda pesa.

## 3. Controle de estoque da farmácia

Escolho **retire**. Fatos: o ERP corporativo já cobre entrada de notas, contagem e inventário (três das cinco funções), e as duas restantes são relatórios impressos uma vez por mês, para quatro usuários. Manter ou mover o sistema antigo seria pagar esforço, dos cinco da equipe, por algo cuja maior parte já existe em outro lugar. Rehost é o R descartado, porque levaria para a nuvem um sistema que passa a ser redundante.

**Condição de saída.** A escolha só é executável depois de resolver as duas funções que o ERP não cobre, ou seja, os dois relatórios mensais. É preciso: (a) definir onde cada um passa a ser gerado, seja um relatório configurado no ERP, uma consulta sobre os dados dele ou uma extração para planilha; (b) rodar **um ciclo mensal completo em paralelo**, comparando o que sai do ERP com o que a farmácia imprime hoje, porque como os relatórios são mensais um erro só apareceria no fim do mês seguinte à desativação; (c) obter aceite da farmácia sobre o resultado; e (d) decidir o destino do histórico do sistema antigo (**hipótese**: pode haver exigência de guarda, que o caso não menciona). Só depois disso o sistema pode ser desligado, sem afetar o atendimento, já que o ERP continua em uso.

## 4. Ordem de execução

Como a equipe de cinco pessoas não faz as três ao mesmo tempo e o prazo é de dezoito meses, ordeno pelo risco de calendário, não pela dor do usuário.

1. **Faturamento (rehost) primeiro.** É onde há mais incerteza sobre a duração: sistema de dezoito anos, hardware físico, hipótese de compatibilidade não confirmada. Começar cedo dá tempo para descobrir surpresas, rodar em paralelo com os servidores físicos (que continuam disponíveis até o fim do prazo, servindo de retorno seguro) e acompanhar fechamentos reais. E ele precisa mudar de qualquer forma, porque a auditoria exige o histórico além do prazo do data center.
2. **Portal (replatform) em segundo.** É o maior esforço e a publicação só acontece uma vez por mês, de madrugada, então há poucas janelas para ensaiar e virar a chave sem indisponibilidade em horário de atendimento. Ele começa assim que a fase de descoberta do faturamento estiver estável, e precisa ser testado sob carga parecida com a de segunda de manhã antes da virada.
3. **Farmácia (retire) por último no esforço da equipe.** É o menor esforço e o mais previsível, e por isso o que pode absorver um atraso sem ameaçar o prazo. Mas o pedido dos dois relatórios ao time do ERP e o ciclo mensal em paralelo custam pouco da equipe e dependem do calendário do mês, então essa preparação começa no primeiro mês, em segundo plano.

Os prazos exatos dependem de estimativas que o caso não traz; a ordem acima resulta do risco de cada decisão frente aos dezoito meses.

## 5. Evidências para a primeira decisão (rehost do faturamento), seis meses depois

**Confirmaria que foi o R certo:** o sistema rodando nas máquinas virtuais há pelo menos um fechamento completo, com os relatórios produzidos idênticos aos que os servidores físicos geram sobre os mesmos dados; consulta de histórico funcionando no novo ambiente, com uma amostra aceita pela auditoria; um teste de restauração feito e bem-sucedido; e nenhum incidente atribuído à mudança de ambiente, com os servidores físicos de retorno seguro sem uso.

**Indicaria que foi o errado:** relatórios com diferença numérica que a equipe não consegue explicar; instabilidade recorrente atribuída a dependências do sistema operacional ou do hardware antigo, exigindo alterar código que ninguém domina; fechamento estourando a janela de tempo que tinha nos servidores físicos; ou a equipe gastando a maior parte do tempo em contornos para manter a pilha antiga de pé. Qualquer um desses sinais indicaria que o prazo pediria outro R (por exemplo, encerrar antes por retire com um extrato do histórico, ou reconstruir), e que a descoberta precisou vir mais cedo.
