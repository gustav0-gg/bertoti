# 📚 Repositório de Atividades Acadêmicas — FATEC

> Portfólio de atividades práticas, estudos de caso, implementações e projetos desenvolvidos por **Gustavo Garcia Gomes** durante o curso de **Análise e Desenvolvimento de Sistemas (ADS)** na FATEC, com foco nas disciplinas lecionadas pelo **Prof. Me. Giuliano Bertoti**.

---

## 🧭 Mapa de Navegação

Clique em uma das seções abaixo para acessar diretamente os detalhes e a documentação completa de cada matéria:

| Pasta | Disciplina | Destaques das Atividades | Acesso Rápido |
|---|---|---|:---:|
| [`/engsoftware1`](./engsoftware1) | **Engenharia de Software I** | Conceitos fundamentais, tradeoffs de engenharia, diagramas de classe UML, código Java e testes automatizados (Hospital e Universidade). | [Ver README](./engsoftware1/README.md) |
| [`/engsoftware3`](./engsoftware3) | **Engenharia de Software III** | Portfólio completo de contribuições no projeto **IAbertoti** (Spring AI, RAG, 5 padrões GoF) e na **PRO4TECH-API** (LightRAG, JWT, PostgreSQL, Chats Contextuais). | [Ver README](./engsoftware3/README.md) |
| [`/IHC`](./IHC) | **Interação Humano-Computador** | **Bot MedInfo**: Agente de IA com DSPy e Phi-3.5-mini (Ollama) para consulta de medicamentos via CLI e Telegram com geração de SQL e regras pós-processadas. | [Ver README](./IHC/readme.md) |

---

## 📂 Detalhamento por Disciplina

### 1. 🟡 [Engenharia de Software I (`/engsoftware1`)](./engsoftware1/README.md)
Documentação das atividades iniciais de fundamentos de Engenharia de Software, modelagem orientada a objetos e práticas de qualidade de código.

- **Conceitos Fundamentais:** Análise crítica sobre a distinção entre programação, ciência da computação e a disciplina de engenharia, destacando a importância da manutenibilidade e longevidade do código.
- **Análise de Tradeoffs:**
  - *Performance vs. Manutenibilidade*
  - *Tempo de Desenvolvimento vs. Qualidade*
  - *Segurança vs. Usabilidade*
- **Atividade Prática 1 (Sistema Hospitalar):**
  - Modelagem com Diagrama de Classes UML
  - Implementação orientada a objetos em Java
  - Testes de unidade automatizados
- **Atividade Prática 2 (Sistema Acadêmico / Universidade):**
  - 2º Diagrama de Classes UML expandido
  - Implementação em Java com associações de entidades
  - Suíte de testes automatizados

📄 **Documentação completa:** Consulte o [`engsoftware1/README.md`](./engsoftware1/README.md).

---

### 2. 🟣 [Engenharia de Software III (`/engsoftware3`)](./engsoftware3/README.md)
Portfólio detalhado das contribuições de nível avançado desenvolvidas no 4º semestre de ADS, combinando inteligência artificial, padrões de arquitetura e desenvolvimento corporativo.

- **Projeto IAbertoti:**
  - **Infraestrutura Spring AI & Ollama:** Integração do framework Spring AI com modelos LLM locais (DeepSeek-R1, Llama 3) e chaveamento dinâmico.
  - **Pipeline RAG em Memória:** Base de conhecimento vetorial (`SimpleVectorStore`) para recuperação de contexto institucional e técnico da disciplina.
  - **5 Padrões de Projeto GoF com IA:**
    - *Strategy* (estratégias de prompting/respostas)
    - *Observer* (notificação e logging de interações de IA)
    - *Factory* (instanciação desacoplada de provedores e modelos)
    - *Decorator* (enriquecimento de prompts e sanitização)
    - *Adapter* (compatibilização de payloads e saídas de LLMs)
  - **Interface Web & Testes:** Interface interativa de chat e suíte de testes unitários/integração.
  - **Modelagem e Documentação:** Diagramas em PlantUML e Mermaid, documentação acadêmica e roteiro de demonstração.
- **Projeto API (PRO4TECH-API):**
  - **Segurança & Autenticação:** Implementação de JWT, interceptores de rotas e integração com o frontend.
  - **Modelagem Relacional:** Evolução e normalização de schemas para suporte a projetos e perfis de desenvolvedores.
  - **Pipeline de IA com LightRAG:** Indexação baseada em grafos + vetores, integração com PostgreSQL (pgvector) e Ollama.
  - **Chats Contextuais Inteligentes:** Chat geral corporativo, análise automática de rascunhos de projetos no cadastro e histórico de pull requests/commits.

📄 **Documentação completa:** Consulte o [`engsoftware3/README.md`](./engsoftware3/README.md).

---

### 3. 🟢 [Interação Humano-Computador (`/IHC`)](./IHC/readme.md)
Desenvolvimento de um agente conversacional inteligente focado em acessibilidade e facilidade de interação para consulta de medicamentos e saúde pública.

- **Bot MedInfo:** Agente em linguagem natural construído com **DSPy** e **Phi-3.5-mini (3.8B)** rodando 100% localmente via Ollama.
- **Pipeline de Atendimento:**
  1. *Classificação de intenção* (consulta, posologia, interação medicamentosa, fora de escopo).
  2. *Geração de SQL* dinâmico a partir do português.
  3. *Mecanismo de autocorreção (Retry)* para queries SQLite.
  4. *Interpretação e humanização* do resultado do banco.
  5. *`aplicar_regras()` determinístico:* Pós-processamento que garante conformidade clínica, isenção médica e segurança independente do modelo.
- **Canais de Interação:**
  - CLI interativa via terminal
  - Bot no **Telegram** com suporte a typing indicator, rate limiting assíncrono e formatação Markdown
- **Otimização:** Uso do otimizador `BootstrapFewShot` do DSPy para auto-seleção dos melhores exemplos de few-shot.
- 🔗 **Repositório da Atividade:** [gustav0-gg/Atividade-de-IHC](https://github.com/gustav0-gg/Atividade-de-IHC)

📄 **Documentação completa:** Consulte o [`IHC/readme.md`](./IHC/readme.md).

---

## 🛠️ Tecnologias e Ferramentas

```
Linguagens & Frameworks:
├── Java (Spring Boot, Spring AI, JUnit 5)
├── Python (DSPy, python-telegram-bot)
└── TypeScript / JavaScript (Node.js, Vue/React)

Inteligência Artificial & LLMs:
├── Ollama (Phi-3.5-mini, DeepSeek-R1, Llama 3)
├── RAG (SimpleVectorStore, LightRAG, pgvector)
└── DSPy (Signatures, Modules, BootstrapFewShot)

Bancos de Dados:
├── PostgreSQL
└── SQLite

Engenharia de Software & Modelagem:
├── Padrões GoF (Strategy, Observer, Factory, Decorator, Adapter)
├── UML (PlantUML, Mermaid)
└── Testes Automatizados (Unit & Integration)
```

---

## 🌲 Estrutura de Diretórios

```
bertoti/
├── README.md               # Este índice central de atividades
├── engsoftware1/           # Atividades de Engenharia de Software I
│   └── README.md
├── engsoftware3/           # Atividades e Portfólio de Engenharia de Software III
│   └── README.md
└── IHC/                    # Atividade de Interação Humano-Computador (Bot MedInfo)
    └── readme.md
```

---

<p align="center">
  Desenvolvido por <strong>Gustavo Garcia Gomes</strong><br>
  Faculdade de Tecnologia de São José dos Campos — Prof. Jessen Vidal (FATEC)
</p>
