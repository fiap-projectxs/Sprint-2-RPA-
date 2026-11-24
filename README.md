# Raly · AI for Robotic Process Automation — Sprint 2

**Automatizar uma tarefa de campo: atendimento de ocorrência em rodovia**

| | |
|---|---|
| **Disciplina** | AI for Robotic Process Automation |
| **Professor** | Orlando Clinio Patriarcha |
| **Projeto** | Challenge Motiva · Sprint 2 |
| **Equipe** | Raly · Turma 2TIAPR |
| **Integrantes** | Rodrigo Wolkoff Giorgi (RM 556980) · Carlos Mamede (RM 568269) |
| **Vídeo de demonstração** | https://youtu.be/08LAuHyLBMQ |

> Todos os dados desta entrega são **fictícios**. Os despachos para SAMU, PRF e Bombeiros são **simulados**. O ganho de tempo é uma **estimativa**.

---

## 1. Sobre a entrega

Esta entrega apresenta a automação de uma tarefa de campo dentro do **SmartDeskAI**, o sistema da Sprint 1: o **atendimento de uma ocorrência em rodovia** (acidente, incêndio, pane, animal na pista, buraco, produto perigoso).

Hoje esse atendimento é manual: alguém lê a ficha, digita, decide a prioridade, redige o relatório e liga para cada órgão. Estimamos de 12 a 18 minutos por ocorrência. Com a automação, o agente de campo envia a ficha (PDF, TXT ou foto) ou dita o relato, e o sistema faz o restante até a aprovação humana.

Esta entrega reúne o **relatório**, o **vídeo**, a **estimativa de ganho**, as **fichas de exemplo** e as **capturas de tela** da execução real. **O código-fonte não faz parte desta entrega.**

## 2. Conteúdo

| Item | O que é |
|---|---|
| `RPA_Sprint2_Automatizar_Tarefa_de_Campo.pdf` | Relatório completo: processo antes e depois, regras, uso da IA, aprovação humana, demonstração, ganho e limitações |
| Vídeo (link acima) | Demonstração do fluxo no site, com os diagramas AS-IS e TO-BE |
| `ganho.md` | Estimativa de ganho de tempo, com cálculo passo a passo |
| `fichas/` | As 5 fichas em PDF usadas na demonstração |
| `capturas/` | Capturas de tela da execução real, com o Ollama `qwen2.5:3b` |

## 3. Como o fluxo funciona

```
Documento (PDF, TXT, foto) ou relato de voz
        │
        ▼
 1. Documento recebido
        │
        ▼
 2. Leitura da IA (Ollama, modelo local qwen2.5:3b)
        │   extrai tipo, rodovia, km, sentido, vítimas, via bloqueada, veículos
        │
        ├── E1  falta rodovia, km ou tipo ........ o sistema pergunta ao agente
        ├── E3  mesmo evento já relatado ......... vincula à ocorrência original
        ▼
 3. Regras de decisão R1–R5 (prioridade, destinos e prazo)
        │
        ├── E2  conflito, confiança baixa ou IA fora do ar ... revisão reforçada
        ▼
 4. Aprovação humana (somente o administrador)
        │   justificativa obrigatória se houver E2 ou se mudar a prioridade
        │   rejeição exige motivo
        ▼
 5. Ação final: chamado real no SmartDeskAI (SD-AAAA-NNNN, canal "Campo"),
    encaminhamento à equipe, despachos simulados e relatório final
        │
        ▼
 Trilha de auditoria em todas as etapas
```

O relatório operacional é redigido pela IA. A prioridade, os destinos e o prazo vêm das regras, e cada decisão cita a regra usada. **Nada é enviado antes da aprovação humana.**

### Regras de decisão

| Regra | Condição | Prioridade | Prazo de resposta |
|---|---|---|---|
| R1 | Vítima grave ou fatal, fogo ou produto perigoso | P1 · Crítica | 5 min |
| R2 | Vítima leve ou via totalmente bloqueada | P2 · Alta | 15 min |
| R3 | Acidente com situação das vítimas desconhecida (na dúvida, sobe) | P2 · Alta | 15 min |
| R4 | Acidente, bloqueio parcial ou animal na pista | no mínimo P3 · Média | 40 min |
| R5 | Pane, buraco ou ocorrência sem risco imediato | P4 · Baixa | 120 min |

Quando mais de uma regra se aplica, vale a prioridade mais alta.

### Exceções tratadas

| Exceção | Gatilho | O que o sistema faz |
|---|---|---|
| **E1** — dados faltando | Falta rodovia, km ou tipo | Pergunta o dado que falta ("Em que quilômetro aconteceu?") e reavalia quando ele chega |
| **E2** — revisão reforçada | Conflito entre os campos e o texto, confiança abaixo de 60%, ou IA fora do ar | Exige justificativa para aprovar |
| **E3** — duplicata | O mesmo evento é relatado de novo | Anexa a informação à ocorrência original e não abre chamado novo |

## 4. Fichas de exemplo e resultado observado

Resultados da execução real, como mostram as capturas.

| Ficha | Cenário | Resultado |
|---|---|---|
| `fichas/ficha_01_acidente_com_vitima.pdf` | Acidente na BR-116 km 215,5, vítimas leves, bloqueio parcial, 2 veículos | **P2 Alta** (R2 e R4). Aciona SAMU, PRF, Concessionária (sinalização) e Guincho, resposta em até 15 min. Aguarda aprovação. |
| `fichas/ficha_02_pane_simples.pdf` | Pane na SP-070 km 42 | **P4 Baixa** (R5). Gera o chamado SD-2026-0001, encaminhado ao CCO. |
| `fichas/ficha_03_produto_perigoso.pdf` | Produto perigoso na BR-381 km 512, vítimas graves, via totalmente bloqueada | **P1 Crítica** (R1 e R2). Aprovada: chamado SD-2026-0002 atribuído à Equipe de Meio Ambiente, 6 despachos simulados e relatório gerado. |
| `fichas/ficha_04_sem_quilometro.pdf` | Animal na BR-101, sem o quilômetro | **E1**. O sistema pergunta o km antes de continuar. |
| `fichas/ficha_05_conflito_de_dados.pdf` | Acidente na BR-040 km 88: a extração diz "sem vítimas", mas o texto cita feridos | **E2**. Decisão sugerida P3 Média (R4), com justificativa obrigatória para aprovar. |
| Dois relatos de voz da mesma batida (BR-116 km 215) | Mesmo acidente da ficha 01, relatado duas vezes | **E3**. Os dois relatos foram vinculados à ocorrência original e nenhum chamado novo foi aberto. |

## 5. Capturas de tela

Os nomes abaixo seguem a ordem da demonstração. Todas vêm da execução real com o Ollama (`qwen2.5:3b`, confiança de 90% nas leituras mostradas).

| Arquivo | O que mostra |
|---|---|
| `capturas/01_lista.png` | Menu Campo: formulário de nova ocorrência (relato, **Ditar**, **Anexar ficha**, **Analisar com IA**) e a lista de ocorrências com status |
| `capturas/02_aprovacao_p2.png` | Ficha 01: decisão P2 Alta, relatório da IA, campo de justificativa e botões **Aprovar e executar ação final** e **Rejeitar** |
| `capturas/03_excecao_e1.png` | Ficha 04: "Faltam dados (E1)" com o campo para complementar o km |
| `capturas/04_excecao_e2.png` | Ficha 05: conflito detectado e justificativa obrigatória |
| `capturas/05_excecao_e3.png` | Relato de voz vinculado à ocorrência original (duplicata) |
| `capturas/06_executada_p1.png` | Ficha 03: ação final executada, chamado SD-2026-0002 aberto e 6 despachos simulados |
| `capturas/07_fila_chamados.png` | Central de **Solicitações**: os chamados SD-2026-0001 e SD-2026-0002, canal "Campo", status "Encaminhada" |

## 6. Estimativa de ganho

Resumo de `ganho.md` (os tempos manuais são **hipóteses do grupo**, não medições em campo):

| Cenário | Manual | Automatizado | Ganho por ocorrência |
|---|---|---|---|
| Conservador | 12 min | 3,5 min | 8,5 min (71%) |
| Esperado | 18 min | 3,4 min | 14,6 min (81%) |

O ganho vem de a pessoa deixar de ler, digitar, classificar, redigir e avisar cada órgão. Ela fica só com o que exige julgamento: **revisar e aprovar**. As exceções entram na conta com 20% a 25% dos casos e 2 minutos a mais cada.

## 7. Critérios da disciplina e onde estão

| Critério | Peso | Onde ver |
|---|---|---|
| Fluxo ponta a ponta | 40% | Seções 2 e 3 do relatório; vídeo; capturas 01 a 07 |
| Uso de IA | 25% | Seção 5 do relatório: leitura do documento, extração, relatório operacional e salvaguardas contra erro |
| Aprovação humana e exceções | 20% | Seções 4 e 6 do relatório; capturas 02 a 05 |
| Documentação e ganho | 15% | Este README, o relatório e `ganho.md` |

## 8. Limitações

- Dados fictícios.
- Os despachos para SAMU, PRF e Bombeiros são arquivos simulados. O chamado e o encaminhamento dentro do SmartDeskAI são reais.
- O ganho é uma estimativa. Os tempos manuais não foram cronometrados.
- A IA pode errar. Por isso existem a verificação cruzada dos campos, a confiança mínima e a aprovação humana obrigatória.
- Ficha em foto depende de OCR (`pytesseract`) para ser lida.
- O código-fonte não faz parte desta entrega.
