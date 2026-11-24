# Estimativa de ganho: manual × automatizado

**Tarefa medida:** atendimento de uma ocorrência em rodovia, desde a chegada da ficha de campo até as equipes serem acionadas.

> **Aviso importante.** Os tempos do processo manual são **hipóteses do grupo**, não medições feitas em campo. Os tempos do processo automatizado vêm do desenho do fluxo (a parte sem IA foi medida; a parte com IA depende do computador). Por isso o resultado é uma **estimativa**. O cálculo está em `ganho.py`: ajuste os números em `CENARIOS` e rode `python ganho.py` para refazer a tabela.

## 1. Como lemos o problema

Uma ocorrência tem sempre as mesmas cinco tarefas de escritório, e todas dependem de uma pessoa: ler a ficha, digitar, decidir a prioridade, redigir o relatório e avisar cada órgão. O fluxo automatizado tira da pessoa a leitura, a digitação, a classificação, a redação e o envio. Ela fica só com o que exige julgamento: **revisar e aprovar**.

Por isso o ganho não vem de "a IA ser rápida". Vem de a pessoa deixar de fazer quatro das cinco tarefas.

## 2. Processo manual (hoje)

| Etapa | O que a pessoa faz | Conservador | Esperado |
|---|---|---|---|
| Ler a ficha ou o relato | Entender o que aconteceu: tipo, local, vítimas, bloqueio | 2 min | 3 min |
| Digitar no sistema | Passar os dados da ficha para o registro | 3 min | 4 min |
| Classificar a prioridade | Consultar a tabela de prioridades e decidir | 1 min | 2 min |
| Redigir o relatório | Escrever o resumo da ocorrência | 3 min | 4 min |
| Acionar os destinos | Ligar ou enviar mensagem para SAMU, PRF, Bombeiros etc. | 3 min | 5 min |
| **Total por ocorrência** | | **12 min** | **18 min** |

- **Conservador** supõe um coordenador experiente, com ficha clara e tabela de prioridades de cabeça.
- **Esperado** supõe o dia a dia: ficha mal preenchida, consulta à tabela, mais de um órgão para avisar.

## 3. Processo automatizado (Raly Campo)

| Etapa | Quem faz | Tempo | De onde vem o número |
|---|---|---|---|
| Enviar a ficha (ou falar o relato) | Agente de campo | 0,5 min | Anexar o PDF ou ditar por voz |
| Ler, extrair, aplicar as regras R1–R5 e redigir o relatório | IA + regras | 0,5 min | A parte sem IA mede cerca de 2 ms. O modelo local (Ollama) depende do computador, então 0,5 min é uma folga |
| Revisar e aprovar | Administrador | 2 min | Ler o relatório pronto e decidir. É o único trabalho humano que sobra |
| Ação final: chamado, despachos e relatório | Sistema | 0 min | Automática depois da aprovação |
| **Subtotal (caso normal)** | | **3 min** | |
| Exceções E1, E2 e E3 | Administrador | +2 min por caso | Ver a seção 4 |

## 4. Como as exceções entram na conta

Nem toda ocorrência é "normal". Quando o fluxo cai numa exceção, o administrador gasta mais tempo:

| Exceção | O que acontece | Custo extra |
|---|---|---|
| **E1** — dado faltando | O sistema pede o dado (por exemplo o km) e reavalia | tempo de pedir e esperar a resposta |
| **E2** — revisão reforçada | Confiança baixa, conflito entre campos ou IA fora do ar. A aprovação exige justificativa | tempo de conferir e escrever a justificativa |
| **E3** — duplicata | O mesmo acidente foi relatado duas vezes. O novo relato é vinculado ao primeiro | quase nenhum |

Assumimos que **20% a 25% dos casos** caem numa exceção e que cada um custa **2 minutos a mais**. A conta fica:

> tempo automatizado = 3 min + (% de exceções × 2 min)

## 5. Resultado

| Cenário | Manual | Automatizado | Ganho por ocorrência | Ocorrências por dia | Horas por dia (manual → auto) | Horas poupadas por dia |
|---|---|---|---|---|---|---|
| Conservador | 12 min | 3,5 min | 8,5 min (71%) | 20 | 4,0 → 1,2 | 2,8 |
| Esperado | 18 min | 3,4 min | 14,6 min (81%) | 30 | 9,0 → 1,7 | 7,3 |

**Conta do cenário conservador, passo a passo**

1. Manual: 2 + 3 + 1 + 3 + 3 = **12 min**.
2. Automatizado: 0,5 + 0,5 + 2 = 3 min, mais 25% × 2 min = 0,5 min de exceções, total **3,5 min**.
3. Ganho: 12 − 3,5 = **8,5 min**, ou 8,5 ÷ 12 = **71%**.
4. Por dia: 20 ocorrências × 12 min = 240 min (4,0 h) no manual contra 20 × 3,5 = 70 min (1,2 h) no automatizado. Sobram **2,8 h** por dia.

O cenário esperado usa 20% de exceções: 3 + 0,2 × 2 = 3,4 min.

**Leia o resultado com cuidado:** "9 horas por dia" no cenário esperado é a soma do trabalho de várias pessoas ou turnos, não de uma só. O que importa é a **proporção** (de 71% a 81% menos tempo por ocorrência) e a ordem de grandeza.

## 6. O que o ganho não mede

- **Padronização.** A mesma regra (R1–R5) vale para todo agente, e a prioridade é sempre explicável.
- **Rastreabilidade.** Toda ação fica na trilha de auditoria: quem, quando e por quê.
- **Segurança.** Nada sai sem aprovação humana, e as exceções exigem justificativa.
- **Menos erro de digitação.** A pessoa não copia mais os dados à mão.

## 7. Limites da estimativa

- Os tempos manuais não foram cronometrados. É a maior fonte de incerteza.
- A IA pode errar. Por isso existem a verificação cruzada, a confiança mínima e a aprovação humana. O ganho vem de tirar a digitação e a redação do coordenador, não de tirar o julgamento dele.
- Os despachos para SAMU, PRF e Bombeiros são **simulados** nesta entrega. O tempo real de acionar um órgão depende de integração externa.
- A taxa de exceções (20% a 25%) é uma suposição. Se for maior, o ganho cai.

## 8. Como validar com dados reais (próximo passo)

1. Cronometrar **10 ocorrências** no processo manual, etapa por etapa.
2. Cronometrar **10 ocorrências** no Raly Campo, incluindo as que caírem em exceção.
3. Contar quantas caíram em E1, E2 e E3 para trocar a taxa suposta pela real.
4. Substituir os números em `CENARIOS` no `ganho.py`, rodar `python ganho.py` e atualizar esta tabela.
