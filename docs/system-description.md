# Kaelle — Descrição do sistema

## Contexto

Kaelle é uma plataforma de videoconferência na qual agentes de IA participam das reuniões como participantes ativos. O sistema diferencia a participação de um agente de uma simples resposta automática por meio de um protocolo social: o agente solicita a palavra, respeita a autoridade concedida pelo host e pode recorrer ao chat quando a fala não é autorizada.

## Escopo

Esta atividade documenta uma visão arquitetural do sistema atual. O objetivo não é reproduzir o código-fonte, mas produzir uma representação versionável e revisável que possa servir de contexto para futuras evoluções assistidas por IA.

### Dentro do escopo

- autenticação e dados por usuário;
- criação e configuração de agentes;
- criação e participação em salas;
- áudio e vídeo em tempo real;
- dispatch explícito de agentes;
- execução do worker de voz;
- protocolo social e governança;
- transcrição/memória de sessões;
- convites temporários;
- interoperabilidade MCP/A2A.

### Fora do escopo

- código-fonte da aplicação real;
- credenciais e segredos;
- configuração operacional completa dos provedores;
- funcionalidades futuras ainda não decididas;
- detalhamento de todos os módulos internos.

## Fronteiras

### Aplicação Kaelle

A aplicação web concentra a experiência do usuário, autenticação, sessões, configuração dos agentes, regras do protocolo social, governança e chamadas de integração.

### LiveKit

O LiveKit Cloud fornece a infraestrutura de comunicação em tempo real e as salas. O aplicativo utiliza o SDK de servidor para tokens, gerenciamento da sala e dispatch dos agentes.

### Worker de agentes

O worker Python é um processo persistente separado. Recebe o job e metadados da sessão/persona, entra na sala e executa o pipeline conversacional.

### Serviços de IA

OpenAI fornece STT e LLM. ElevenLabs fornece TTS. Tavus pode fornecer avatar de vídeo quando configurado.

### Persistência

O backend utiliza Postgres, autenticação, realtime e RLS. A transcrição possui uma ponte autenticada entre o worker e a aplicação; há também suporte opcional a persistência direta conforme configuração do ambiente.

## Restrições relevantes

- O worker de voz precisa de um processo/container persistente compatível com o runtime do LiveKit; não é tratado como Cloudflare Worker.
- O dispatch do agente é explícito; o worker registra o nome do agente e a aplicação solicita sua entrada na sala.
- Credenciais do worker não devem ser tratadas como se fossem automaticamente compartilhadas com o backend.
- Avatar é opcional e não deve ser considerado requisito do pipeline básico de voz.
- Algumas capacidades ainda estão em evolução e devem permanecer documentadas como lacunas, em vez de serem convertidas em decisões por inferência.

## Lacunas conhecidas

- transcrição ponta a ponta ainda possui integração pendente no fluxo da sala;
- reconexão automática e alguns aspectos operacionais do worker ainda estão em backlog;
- controles adicionais de segurança, métricas e testes de integração ainda precisam ser consolidados;
- parte da governança e da interoperabilidade continua em evolução.

## Regra para futuros agentes

Quando este documento não determinar uma decisão, o agente de desenvolvimento deve tratar o item como lacuna e pedir confirmação. Não deve escolher arbitrariamente um provedor, uma topologia, uma política de governança ou um comportamento de negócio apenas porque a alternativa parece tecnicamente conveniente.
