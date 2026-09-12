# Kaelle — Descrição do sistema

## 1. Contexto

Kaelle é uma plataforma de videoconferência na qual agentes de IA participam das reuniões como participantes ativos. O sistema diferencia a participação de um agente de uma simples resposta automática por meio de um protocolo social: o agente pode solicitar a palavra, respeita a autoridade concedida pelo host e pode recorrer ao chat quando a fala não é autorizada.

Esta documentação foi produzida a partir da implementação e da documentação disponíveis no sistema real, que permanece em repositório privado.

## 2. Objetivo da atividade

O objetivo é produzir uma representação arquitetural versionável, revisável e reproduzível usando **Diagrams as Code**. Os diagramas não substituem especificações detalhadas; funcionam como contexto arquitetural para pessoas e futuros agentes de desenvolvimento.

## 3. Escopo

### Dentro do escopo

- autenticação e dados por usuário;
- criação e configuração de agentes;
- criação e participação em salas;
- áudio e vídeo em tempo real;
- dispatch explícito de agentes;
- execução do worker de voz;
- protocolo social e governança;
- recuperação de contexto da sala e briefing documental;
- transcrição/memória de sessões;
- convites temporários;
- interoperabilidade MCP/A2A.

### Fora do escopo

- código-fonte da aplicação real;
- credenciais e segredos;
- configuração operacional completa dos provedores;
- funcionalidades futuras ainda não decididas;
- detalhamento de todos os módulos internos;
- descrição exaustiva do modelo de dados.

## 4. Fronteiras e responsabilidades

### Aplicação Kaelle

A aplicação web concentra a experiência do usuário, autenticação, sessões, configuração dos agentes, regras e estado de governança e chamadas de integração.

### LiveKit

O LiveKit Cloud fornece a infraestrutura de comunicação em tempo real e as salas. A aplicação utiliza o serviço para acesso às salas e dispatch dos agentes.

### Worker de agentes

O worker Python é um processo persistente separado. Recebe o job e metadados da sessão/persona, entra na sala e executa o pipeline conversacional. O código do `SocialAgent` também implementa comportamentos de silêncio, menção direta, solicitação de palavra, espera por decisão e fallback para chat.

### Serviços de IA

OpenAI fornece STT e LLM. ElevenLabs fornece TTS. Tavus pode fornecer avatar de vídeo quando configurado.

### Persistência e contexto

O backend utiliza Postgres, autenticação, realtime e RLS. O worker possui uma ponte autenticada com a aplicação para persistência de transcrição. O agente também solicita contexto da sala e briefing documental antes de continuar o processamento do turno.

## 5. Nível de abstração adotado

O diagrama estrutural utiliza uma visão **C4-inspired de containers**, mas não tenta representar cada classe, função ou módulo interno.

Uma decisão importante desta atividade foi não representar o **protocolo social** e o **dispatch** como containers independentes. Eles são tratados como responsabilidades/capacidades da aplicação e do agente, respectivamente, porque a implementação observada não justifica apresentá-los como unidades implantáveis separadas.

Da mesma forma, MCP/A2A aparecem como interfaces de integração e não como um novo subsistema interno da plataforma.

Essa escolha evita que o diagrama atribua uma fronteira física a algo que, na implementação analisada, é principalmente uma responsabilidade lógica.

## 6. Restrições relevantes

- o worker de voz precisa de um processo/container persistente compatível com o runtime do LiveKit;
- o dispatch do agente é explícito;
- credenciais do worker não devem ser tratadas como automaticamente compartilhadas com o backend;
- avatar é opcional e não faz parte do pipeline básico de voz;
- a ausência de uma decisão explícita deve ser tratada como lacuna, e não como autorização para o agente escolher uma alternativa.

## 7. Validação contra a implementação

A representação foi confrontada com o código do worker. Foram confirmados, entre outros pontos:

- o worker recebe metadados da sessão e da persona;
- o `SocialAgent` controla estados de silêncio e participação;
- antes do processamento social, o agente pode injetar contexto da sala e briefing documental no turno;
- o agente pode solicitar a palavra com uma contribuição preliminar;
- a decisão pode ser aguardada de forma assíncrona;
- quando aprovado, o agente fala e depois libera a palavra;
- quando negado ou expirado, existe fallback para chat;
- a transcrição utiliza uma ponte autenticada da aplicação, com persistência direta como fallback opcional quando configurada.

## 8. O que a IA inferiu corretamente

A geração assistida por IA identificou corretamente:

- separação entre aplicação web e worker;
- LiveKit como camada de comunicação realtime;
- dispatch explícito;
- pipeline STT → LLM → TTS;
- persistência das interações;
- governança/protocolo social como elemento central do domínio;
- avatar como integração opcional;
- MCP/A2A como fronteira de interoperabilidade.

## 9. O que foi ajustado no modelo gerado

A primeira representação tendia a tratar a Kaelle como uma videoconferência com um chatbot acoplado. A análise do sistema real mostrou que isso ocultava o mecanismo mais relevante: **a participação do agente é governada**.

Os principais ajustes foram:

1. retirar o protocolo social da condição de container independente e tratá-lo como responsabilidade de domínio;
2. retirar o dispatch da condição de container independente e tratá-lo como capacidade da aplicação;
3. manter o worker Python como unidade separada;
4. representar o dispatch explícito entre aplicação e LiveKit;
5. incluir a recuperação de contexto/briefing na jornada comportamental;
6. representar STT, LLM e TTS como integrações especializadas;
7. tratar avatar como opcional;
8. manter MCP/A2A como interfaces de integração.

## 10. Lacunas conhecidas

Para que um futuro agente de desenvolvimento possa evoluir o sistema sem inventar decisões, ainda são necessários, entre outros:

- contratos formais das APIs;
- modelo de dados completo;
- estados e transições completos do protocolo social;
- matriz de autoridade e permissões;
- regras de expiração de autorizações e convites;
- contrato dos eventos de governança;
- política de transcrição e retenção;
- requisitos de segurança, rate limiting e proteção contra replay;
- métricas e SLOs do pipeline de voz;
- estratégia completa de reconexão e recuperação do worker;
- contrato de interoperabilidade MCP/A2A;
- decisões ainda abertas no backlog.

## 11. Regra para futuros agentes

Quando este documento não determinar uma decisão, o agente de desenvolvimento deve tratar o item como **lacuna**, explicitar a dependência e solicitar confirmação. Não deve escolher arbitrariamente um provedor, uma topologia, uma política de governança ou um comportamento de negócio apenas porque a alternativa parece tecnicamente conveniente.

## 12. Relação com o sistema real

Este repositório público contém a documentação arquitetural produzida para a atividade. O código-fonte da Kaelle permanece privado.

A documentação deve ser atualizada quando uma mudança relevante alterar fronteiras, responsabilidades, integrações ou decisões arquiteturais.
