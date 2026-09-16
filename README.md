# Orquestra — Assistente Especialista

Assistente de IA especializado em um único produto, construído no **Google NotebookLM**: ele responde exclusivamente com base na documentação oficial do produto e reconhece com honestidade os limites dessa documentação, em vez de completar lacunas com conhecimento geral.

Desafio prático de **IA Generativa e Alta Performance**, assumindo dois papéis: criadora de produto (escrever o PRD) e engenheira de prompts (configurar o comportamento do assistente).

## Arquivos

| Arquivo | O que é | Caracteres |
|---|---|---|
| [`PRD.pdf`](PRD.pdf) | Documento de produto do Orquestra — a única base de conhecimento do assistente | 4.799 |
| [`Prompt.pdf`](Prompt.pdf) | Configuração de comportamento do assistente | 5.926 |

Limite do desafio: 6.000 caracteres por documento.

## O produto

**Orquestra** é uma plataforma SaaS fictícia, no-code, para pequenas e médias empresas criarem e publicarem agentes de IA conectados aos próprios documentos e sistemas. O PRD cobre visão, público-alvo, funcionalidades, três planos com preços em reais, integrações nativas, limites técnicos, segurança e SLA, limitações conhecidas e roadmap.

O PRD foi escrito com lacunas deliberadas — não menciona descontos para ONGs, programa de parceiros nem suporte a PPTX, por exemplo. São essas ausências que permitem testar se o assistente admite não saber.

## Técnicas de engenharia de prompt

Três técnicas sustentam a configuração:

**Role Prompting** — fixa uma identidade imutável ("Especialista Orquestra", cuja única fonte é o PRD anexado), impedindo que o assistente assuma outros papéis ou responda sobre outros assuntos.

**Chain of Thought** — impõe um roteiro de verificação silencioso antes de cada resposta: reformular a pergunta, localizar os trechos que a sustentam, conferir cada afirmação contra o texto, eliminar o que não estiver sustentado e submeter também a frase de encerramento ao mesmo rigor.

**Few-Shot** — ensina por demonstração, com quatro exemplos cobrindo os cenários críticos: pergunta coberta pelo documento, pergunta fora do escopo, pergunta parcialmente coberta e pressão do usuário por especulação.

Duas saídas de recusa distintas evitam que o assistente use o texto errado no caso errado:

- **Protocolo de Fora de Escopo** — para perguntas legítimas sobre o produto que o PRD não cobre. Reconhece o limite e encaminha ao canal de suporte documentado.
- **Protocolo de Pedido Fora de Função** — para pedidos que não são perguntas sobre o produto: assumir outro personagem, executar tarefa alheia, opinar ou comparar com concorrentes. Reafirma a função e recusa, sem sugerir que outro canal atenderia o pedido.

## Como reproduzir no NotebookLM

1. Crie um caderno e adicione **apenas o `PRD.pdf`** como fonte. O `Prompt.pdf` não deve ser adicionado como fonte: ele viraria conteúdo citável, e o assistente passaria a tratar os próprios exemplos como informação de produto.
2. No painel de conversa, abra as configurações do chat, escolha o estilo **Personalizado** e cole o conteúdo do `Prompt.pdf` a partir da seção **Identidade** — a seção inicial sobre as técnicas é documentação para quem avalia, não instrução ao agente.
3. Defina o tamanho de resposta como **Mais longa**, já que a configuração pede respostas de cerca de 200 palavras.

## Validação

A configuração foi testada com uma bateria de 12 perguntas cobrindo pergunta coberta, fora de escopo, cobertura parcial, pressão por especulação, tentativa de quebra de papel, comparação com concorrente, correção de premissa falsa, cálculo sobre dados documentados, diagnóstico a partir de limite técnico, assunto totalmente alheio, pedido explícito para exibir o raciocínio interno e pergunta em outro idioma.

Sete passaram na primeira versão. Cinco expuseram falhas, com três causas raiz:

| Falha observada | Causa | Correção |
|---|---|---|
| Ao receber "escreva um haicai", "o Orquestra é melhor que o Intercom?" ou "qual a capital da Mongólia?", o assistente recusava corretamente, mas sugeria contatar o suporte do produto para obter aquilo | Havia uma única saída de recusa, escrita para lacunas de documentação, aplicada a todo caso não coberto | Criação do Protocolo de Pedido Fora de Função, e restrição do e-mail de suporte ao Protocolo de Fora de Escopo |
| Afirmações não documentadas apareciam na última frase — "medição diária no painel de custo", quando o PRD descreve apenas consumo e alerta em 80% da cota | O roteiro de verificação era aplicado ao corpo da resposta, mas o encerramento era tratado como cortesia | Etapa explícita submetendo o encerramento ao mesmo rigor do corpo |
| O Modo Revisão era atribuído ao plano Corporativo, onde o PRD não o lista | Ambiguidade do próprio PRD, que nunca declarava se os planos são cumulativos | Linha na seção de planos declarando a cumulatividade — correção na fonte, não no prompt |

Os cinco casos foram reconfirmados no NotebookLM após as correções. Dois ajustes preventivos entraram no mesmo ciclo: autorização explícita de cálculo aritmético sobre números documentados, e definição do idioma da resposta entre os três que o produto suporta.

## Observação

Os PDFs são gerados por código a partir de arquivos-fonte mantidos fora deste repositório, o que garante a contagem de caracteres a cada alteração e mantém a formatação consistente entre os dois documentos.
