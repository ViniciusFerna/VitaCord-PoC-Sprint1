# VitaCord — Visão Geral do Projeto

---

## 1. Nome do Projeto

**VitaCord** — Sistema integrado de monitoramento contínuo de saúde com suporte à decisão clínica por IA, desenvolvido pela CarePlus.

---

**Integrantes:**

566611 Ernandes da Silva Jesus
568157 Leandro Filippini Aguiar Alves 
567772 Raul Bridi Albano
567977 Vinicius Fernandes Silva Souza
568086 Matheus Guedes Grigorio

---

## 2. Persona Escolhida

### Camila Souza, 34 anos — Analista de Marketing

Camila mora em São Paulo, trabalha em regime híbrido e tem uma rotina corrida entre reuniões, academia e cuidar do filho de 5 anos. Foi diagnosticada com hipertensão leve há dois anos e, desde então, tenta manter hábitos mais saudáveis, dorme melhor, reduziu o sal e tenta se exercitar três vezes por semana, nem sempre com sucesso.

Ela usa um smartwatch diariamente, principalmente para monitorar passos e sono, mas nunca explorou a fundo os dados de saúde que o aparelho coleta. Camila se consulta com seu médico a cada três meses, mas sente que a consulta é curta demais para transmitir tudo o que viveu naquele período, os dias de estresse intenso, as noites mal dormidas, os episódios em que sentiu o coração acelerar sem motivo aparente.

Ela gostaria de sentir que seu médico está sempre bem informado sobre seu estado de saúde, sem precisar depender apenas da sua memória durante a consulta. Por não ter conhecimento técnico sobre saúde, valoriza muito quando as informações são apresentadas de forma simples e visual.

**Objetivos:**
- Ter sua saúde monitorada de forma contínua e automática
- Garantir que seu médico tenha um histórico completo e preciso antes de cada consulta
- Entender melhor como seu estilo de vida impacta sua saúde no dia a dia

**Frustrações:**
- Esquecer de registrar sintomas importantes entre as consultas
- Sentir que a consulta não captura a realidade do seu dia a dia
- Não saber quando algo que está sentindo merece atenção médica

---

## 3. Justificativa da Persona

A paciente é a beneficiária final do sistema e, ao mesmo tempo, a principal fonte de dados que o alimenta. Ela possui uma condição de saúde que demanda acompanhamento contínuo, mas que frequentemente não encontra suporte adequado dentro do modelo tradicional de saúde. O fato de já utilizar um smartwatch no cotidiano a torna uma candidata natural para o sistema, demonstrando familiaridade prévia com tecnologia e dispositivos wearables.

O agente de IA voltado para esse perfil deve equilibrar acessibilidade e seriedade, adotando um tom amigável que facilite a interação, mas sem abrir mão da credibilidade esperada de um sistema de saúde.

---

## 4. Stack Técnica Selecionada

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Back-end | Java Spring Boot | Maturidade, segurança robusta com Spring Security e amplo ecossistema |
| Front-end | React | Flexibilidade, grande ecossistema e fácil integração com APIs REST |
| Banco relacional | MySQL | Dados estruturados: cadastros, prescrições e históricos clínicos |
| Banco de série temporal | TimescaleDB | Performance para o volume contínuo de leituras dos sensores |
| LLM | qwen3.5:9b via Ollama | Execução local, sem dependência de API externa, suporte a function calling |
| Hardware | ESP32 + sensores | Coleta de sinais vitais (temperatura corporal e batimentos cardíacos) |
| Protocolo IoT | HTTP / MQTT | MQTT para conexões instáveis e envio frequente de dados do wearable |

---

## 5. Riscos Clínicos

### 5.1 Alucinação em Contexto Médico
Modelos de linguagem podem gerar informações clinicamente incorretas com aparência de confiabilidade. Para mitigar esse risco, o agente não fornece diagnósticos nem prescrições, sua atuação se limita à coleta de dados e apresentação de padrões ao médico.

### 5.2 Viés nos Dados
Dados coletados por wearables podem ser imprecisos dependendo do tipo de sensor, posicionamento ou condição do usuário. O sistema trata os dados como indicativos, não como diagnósticos, e sempre os apresenta sob supervisão médica.

### 5.3 LGPD e Privacidade
Dados de saúde são dados sensíveis sob a Lei Geral de Proteção de Dados (Lei nº 13.709/2018). As mitigações adotadas incluem criptografia em repouso e em trânsito, controle de acesso por perfil e logs de auditoria de todas as interações com o sistema.

### 5.4 Responsabilidade sobre Prescrição
O sistema não gera prescrições de forma autônoma. Toda prescrição é criada pelo médico responsável após análise dos dados apresentados pelo sistema, garantindo que a responsabilidade clínica permaneça integralmente com o profissional habilitado.

### 5.5 Tentativas de Jailbreak
Usuários podem tentar induzir o agente a fornecer diagnósticos ou prescrições por meio de framing fictício, hipotético ou por alegação de autoridade. O system prompt contém instruções explícitas de recusa para esses cenários, com redirecionamento para o escopo correto.

---

## 6. Arquitetura Proposta

O sistema é composto por seis camadas principais:

**Entrada** — O paciente interage via interface web e os dados dos sensores são enviados pelo ESP32 via HTTP ou MQTT para o back-end.

**Roteamento** — O API Gateway (Spring Boot) autentica as requisições via Spring Security e direciona o tráfego para o agente do paciente ou para o dashboard do médico.

**Agente do Paciente** — LLM local (qwen3.5:9b via Ollama) orientado por um system prompt com identidade e restrições permanentes, com memória de conversa por turnos.

**RAG** — Antes de responder, o agente recupera contexto relevante de uma base de conhecimento vetorial contendo protocolos clínicos, bulas e diretrizes médicas.

**Guardrails** — Três camadas de proteção: detecção de red flags (emergências), filtro de jailbreak (desvios de papel) e validador de saída (bloqueio de diagnósticos e prescrições).

**Human-in-the-Loop** — O médico acessa um dashboard com os dados organizados e padrões identificados. Toda prescrição exige aprovação explícita do médico antes de ser registrada no sistema.

> O fluxograma completo da arquitetura está disponível em `/docs/arquitetura.png`.

## 7. Análise Comparativa de Modelos LLM — VitaCord

Este documento apresenta a análise comparativa entre os dois modelos candidatos avaliados para uso em produção no sistema VitaCord, considerando custo, latência, contexto, privacidade e suporte a function calling.

---

### Modelos Avaliados

| Critério | GPT-5.4 (OpenAI API) | Qwen3.5:9b (Local via Ollama) |
|---|---|---|
| **Custo de input** | $2.50 / 1M tokens | $0.00 |
| **Custo de output** | $15.00 / 1M tokens | $0.00 |
| **Contexto máximo** | 272k tokens (curto) | 256k tokens |
| **Latência média** | 8–10 segundos | ~12 segundos |
| **Privacidade / On-premise** | ❌ Dados enviados para servidores externos | ✅ 100% local, nenhum dado sai da máquina |
| **Function Calling** | ✅ Nativo e altamente confiável | ✅ Suportado, confiabilidade variável |
| **Infraestrutura necessária** | Nenhuma (API gerenciada) | Hardware dedicado com GPU |
| **Disponibilidade** | Alta (SLA gerenciado pela OpenAI) | Depende da infraestrutura local |

---

### Análise por Critério

#### Custo

O GPT-5.4 opera no modelo de cobrança por token. Considerando o perfil de uso do VitaCord — system prompt de ~800 tokens, histórico de conversa com poucos turnos e retorno de tools, estima-se uma média de **2.000 a 4.000 tokens por interação**. Para 10.000 interações mensais, o custo estimado ficaria entre **$35 e $70/mês**, o que é viável para uma operação inicial.

O Qwen3.5:9b não tem custo por token, mas exige um servidor com GPU dedicada para rodar em produção com múltiplos usuários simultâneos, o que representa um custo de infraestrutura a ser considerado.

---

#### Latência

O GPT-5.4 apresenta latência média de 8 a 10 segundos por resposta, condicionada à disponibilidade da API e à carga nos servidores da OpenAI. O Qwen3.5:9b, rodando localmente, apresenta média de 12 segundos, variando conforme o hardware disponível e o número de requisições simultâneas. Para o contexto do VitaCord, onde as interações não exigem respostas em tempo real, ambas as latências são aceitáveis.

---

#### Contexto Máximo

Ambos os modelos suportam janelas de contexto amplas: 272k e 256k tokens respectivamente, suficientes para qualquer cenário do sistema. Na prática, o uso do RAG garante que cada chamada opere com contexto curto, tornando esse critério pouco diferenciador para o projeto.

---

#### Privacidade e Conformidade com a LGPD

Este é o critério mais crítico para um sistema de saúde. O GPT-5.4 envia os dados para servidores externos da OpenAI, o que exige análise cuidadosa quanto à conformidade com a **LGPD (Lei nº 13.709/2018)**, especialmente por se tratar de dados sensíveis de saúde. Seria necessário avaliar os termos de uso da API e, possivelmente, firmar um DPA (Data Processing Agreement) com a OpenAI.

O Qwen3.5:9b processa todos os dados localmente, sem nenhuma transmissão externa, o que elimina esse risco por completo e torna a conformidade com a LGPD significativamente mais simples de garantir.

---

#### Suporte a Function Calling

O GPT-5.4 oferece suporte nativo e amplamente documentado a function calling, com alta confiabilidade na identificação de quando e como acionar as tools. O Qwen3.5:9b também suporta function calling via Ollama, mas com confiabilidade menos consistente em cenários mais complexos, podendo ocasionalmente falhar em identificar o momento correto de acionar uma tool ou interpretar incorretamente os argumentos.

---

Para o estágio atual do projeto, o **Qwen3.5:9b** é o modelo adotado por eliminar custos de API e garantir conformidade nativa com a LGPD. A migração para o **GPT-5.4** deve ser avaliada caso a escala do sistema exija maior confiabilidade no function calling ou infraestrutura local se torne inviável.
