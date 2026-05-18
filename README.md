<<<<<<< HEAD
# blua-diagnostics
=======
#  BluaDiagnostics — Plataforma de Cuidado Remoto Proativo

> Challenge Care Plus × Sprint 1 — Exploração, Arquitetura e PoC

---

##  Integrantes do Grupo

| Nome | RM |
|------|----|
| Miguel Fontres Costa | RM-566761 |
| Gabriel de Paula Gil | RM-568107 |
| Thiago Gracia | RM-567971 |
| Diego Leite Asprino | RM-561662 |
| Pedro Cesar Branco | RM-567971 |

---

##  Persona Atendida: Beneficiário Final em Autoavaliação

**Justificativa:** Escolhemos o **beneficiário final (paciente)** como persona primária por três razões estratégicas:

1. **Maior impacto em escala**: A Care Plus possui 600 mil beneficiários. Um agente de check-up digital que atende diretamente o paciente multiplica o alcance sem depender da disponibilidade médica.
2. **Encaixe com o pilar de check-up digital**: A autoavaliação conversacional é o ponto de entrada natural — o paciente relata sintomas e sinais vitais, e o sistema filtra quem realmente precisa de atendimento urgente vs. orientação preventiva.
3. **Menor risco regulatório na PoC**: Atender o beneficiário com orientação (não diagnóstico) é mais seguro regulatoriamente do que um agente que gera prescrições, tornando a PoC viável sem aprovações médico-legais extensas.

**Perfil da persona:**
- Usuário do app Blua, 30–55 anos
- Busca orientação rápida sobre sintomas sem precisar ligar para a operadora
- Pode ou não ter histórico de condições crônicas (hipertensão, diabetes)
- Espera linguagem acessível, não jargão médico

---

##  Stack Técnica

| Componente | Escolha | Justificativa |
|------------|---------|---------------|
| **LLM** | Claude claude-sonnet-4-20250514 (Anthropic) | Melhor equilíbrio custo-qualidade clínica, suporte nativo a function calling estruturado, contexto 200K tokens |
| **Framework** | LangChain + LangGraph | LangChain para RAG/chains; LangGraph para orquestração multi-agente (bônus Sprint 2) |
| **Vector Store** | ChromaDB (local) | Open source, sem dependência de nuvem, fácil para PoC |
| **Embeddings** | `text-embedding-3-small` (OpenAI) ou `voyage-3` (Anthropic) | Alta qualidade semântica para textos clínicos em português |
| **API Key Management** | `python-dotenv` + `.env` | Nunca commits de keys |
| **Ambiente** | PyCharm + `venv` / Google Colab (notebook) | Desenvolvido localmente, notebook compatível com Colab |

###  Análise Comparativa de Modelos

| Critério | Claude claude-sonnet-4-20250514 ✅ | GPT-4o |
|----------|--------------------------------|--------|
| Custo (input/1M tokens) | ~$3,00 | ~$2,50 |
| Custo (output/1M tokens) | ~$15,00 | ~$10,00 |
| Contexto máximo | 200K tokens | 128K tokens |
| Latência média (P50) | ~1,2s | ~1,0s |
| Function calling estruturado | ✅ Nativo (tool_use) | ✅ Nativo |
| Suporte a português clínico | ✅ Excelente | ✅ Muito bom |
| Privacidade / on-premise | ❌ Cloud only | ❌ Cloud only |
| Qualidade clínica (raciocínio) | ✅ Superior em textos longos | ✅ Muito bom |
| Rejeição de jailbreak médico | ✅ Robusto por treinamento |

**Decisão**: Claude Sonnet foi escolhido pelo contexto maior (histórico clínico extenso), pela robustez intrínseca contra geração de diagnósticos definitivos, e pelo suporte via SDK Anthropic com tipagem Python clara.

---

##  Riscos Clínicos e Éticos Mapeados

| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| Alucinação em contexto médico | Média | Crítico | RAG com fontes validadas; respostas sempre com disclaimer; HITL obrigatório |
| Viés demográfico no triagem | Baixa | Alto | Prompts neutros; testes com personas diversas no eval set |
| Diagnóstico definitivo pelo LLM | Alta (sem guardrail) | Crítico | Guardrail explícito no system prompt; casos de jailbreak no eval set |
| Violação LGPD — dados de saúde | Média | Crítico | Dados mock na PoC; sem PII real; política de retenção 0 em logs |
| Responsabilidade sobre prescrição | Alta (pilar 2) | Crítico | Prescrição SEMPRE requer aprovação médica; agente apenas sugere |
| Dependência excessiva do app | Média | Médio | Escalada para atendimento humano em red flags; disclaimer padrão |
| Falso negativo em red flag | Baixa | Crítico | Protocolo Manchester simplificado no RAG; threshold conservador |

---

##  Estrutura do Repositório

```
blua_diagnostics/
├── README.md                    ← Este arquivo
├── .env.example                 ← Template de variáveis de ambiente
├── requirements.txt             ← Dependências Python
├── docs/
│   └── arquitetura.svg          ← Fluxograma da arquitetura
├── evals/
│   └── sprint1_eval_set.json    ← 10+ casos de avaliação
├── prompts/
│   └── system_prompt.md         ← System prompt do agente
├── tools/
│   └── tools_spec.json          ← Contrato das tools (JSON Schema)
└── notebooks/
    └── sprint1_poc.ipynb        ← PoC funcional
```

---

## Como Executar

```bash
# 1. Clone o repositório
git clone https://github.com/seu-grupo/blua-diagnostics.git
cd blua-diagnostics

# 2. Crie e ative o ambiente virtual
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Instale as dependências
pip install -r requirements.txt

# 4. Configure as variáveis de ambiente
cp .env.example .env
# Edite .env e insira sua ANTHROPIC_API_KEY

# 5. Execute o notebook
jupyter notebook notebooks/sprint1_poc.ipynb
```
