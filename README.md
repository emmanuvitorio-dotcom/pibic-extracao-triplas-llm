# PIBIC — Extração de Triplas para Grafos de Conhecimento via LLMs

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/emmanuvitorio-dotcom/pibic-extracao-triplas-llm/blob/main/notebooks/PIBIC_Projeto_Final.ipynb)
[![Python 3](https://img.shields.io/badge/python-3-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Pipeline de pesquisa (PIBIC) para comparar **Large Language Models (LLMs)** na
tarefa de **extração de triplas (sujeito, predicado, objeto)** a partir de um dataset de textos
farmacológicos (metabolismo e absorção de drogas), com avaliação automática e humana da
qualidade das extrações.

## Sumário

- [Visão geral](#visão-geral)
- [Metodologia](#metodologia)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como rodar](#como-rodar)
- [Configuração de credenciais](#configuração-de-credenciais)
- [Checkpoints e retomada de execução](#checkpoints-e-retomada-de-execução)
- [Métricas e avaliação](#métricas-e-avaliação)
- [Limitações conhecidas](#limitações-conhecidas)
- [Status do projeto](#status-do-projeto)
- [Autor](#autor)

## Visão geral

O projeto constrói um pipeline reprodutível para:

1. Extrair triplas de conhecimento (`sujeito :: predicado :: objeto`) de textos farmacológicos
   usando múltiplos LLMs, em duas estratégias de prompting (**Zero-Shot** e **Few-Shot**).
2. Medir métricas automáticas de desempenho: número médio de tokens, latência média por
   requisição e taxa de parsing válido das triplas.
3. Avaliar a qualidade das extrações usando **LLM-as-a-judge**, segundo 4 critérios (nota de
   1 a 5 cada): **Formato**, **Completude**, **Corretude** e **Ausência de Alucinação**.
4. Selecionar uma amostra aleatória de **1%** dos resultados para avaliação humana
   especializada, com seed fixa para reprodutibilidade.

**Modelos candidatos (via [Groq](https://groq.com/)):**

| Modelo | Família | Observação |
|---|---|---|
| `openai/gpt-oss-120b` | OpenAI (via Groq) | modelo de raciocínio, `reasoning_effort` padrão |
| `qwen/qwen3.6-27b` | Alibaba / Qwen (via Groq) | `reasoning_effort="none"` |
| `llama-3.3-70b-versatile` | Meta (via Groq) | — |

**Juiz (LLM-as-a-judge):** [Maritaca](https://www.maritaca.ai/) `sabia-4`, via API própria
compatível com OpenAI. O juiz é de uma família diferente dos modelos candidatos, o que reduz o
risco de *self-preference bias* (viés do juiz favorecer respostas no seu próprio estilo).

## Metodologia

O notebook principal (`notebooks/PIBIC_Projeto_Final.ipynb`) está organizado em 13 seções
sequenciais:

1. **Setup** — instalação de dependências e configuração inicial.
2. **Carregamento do dataset (XML)** — leitura do dataset de textos farmacológicos
   (metabolismo/absorção de drogas) e cópia para o Google Drive.
3. **Prompts de extração** — templates de prompt Zero-Shot e Few-Shot.
4. **Execução nos modelos** — chamadas às APIs com tratamento de erro 429 (cota esgotada) e
   retry para falhas transitórias.
5. **Sistema de checkpoint** — persistência incremental de progresso no Google Drive.
6. **Definições do LLM-as-a-judge** — funções de avaliação automática.
7. **Loop principal de execução** — orquestra todas as combinações (texto × modelo ×
   estratégia) com checkpoint e tolerância a falhas.
8. **Modo teste** — validação do pipeline numa amostra pequena antes do experimento completo.
9. **Execução do experimento completo** — dataset completo (~2.628 drogas / ~4.700 textos).
10. **Métricas automáticas** — tokens, latência, taxa de parsing.
11. **Avaliação do experimento completo pelo juiz**.
12. **Amostragem de 1%** para revisão humana especializada (exportada em Excel).
13. **Resumo final comparativo** — consolida métricas automáticas e notas do juiz por
    modelo/estratégia.

> ⚠️ **Nota metodológica:** o dataset completo gera aproximadamente **28.000 chamadas de
> extração** (4 combinações de modelo/estratégia × ~7.000 textos) mais ~24.000 chamadas de
> avaliação pelo juiz. O notebook assume o **Groq Developer Tier** para rodar o experimento
> completo numa única sessão; no plano gratuito, a execução precisa ser distribuída em várias
> sessões/dias — o sistema de checkpoint dá suporte a isso nativamente.

## Estrutura do repositório

```
.
├── notebooks/
│   └── PIBIC_Projeto_Final.ipynb   # Notebook principal (pipeline completo)
├── data/
│   └── README.md                   # Instruções para obter/posicionar o dataset
├── docs/
│   └── metodologia.md              # Detalhamento metodológico e critérios de avaliação
├── requirements.txt                 # Dependências Python (referência local/fora do Colab)
├── .gitignore
├── LICENSE
└── README.md                        # Este arquivo
```

## Como rodar

O notebook foi desenvolvido para rodar no **Google Colab** (usa `google.colab.drive`,
`google.colab.userdata` e `google.colab.auth`).

1. Abra `notebooks/PIBIC_Projeto_Final.ipynb` no [Google Colab](https://colab.research.google.com/).
2. Configure as credenciais de API (veja [Configuração de credenciais](#configuração-de-credenciais)).
3. Rode a **Seção 1 (Setup)** para instalar dependências e montar o Google Drive.
4. Na **Seção 2**, faça upload do dataset XML (`metabolism_absorption.xml`) — veja
   `data/README.md`. Isso só é necessário na primeira execução; nas seguintes, o arquivo é lido
   direto do Drive.
5. Rode a **Seção 8 (Modo teste)** primeiro, com uma amostra pequena (10–20 itens), para validar
   o pipeline antes de gastar cota de API no dataset completo.
6. Se os resultados do teste estiverem corretos, prossiga para a **Seção 9** (experimento
   completo).

## Configuração de credenciais

O notebook **não contém nenhuma chave de API no código** — todas as credenciais são lidas em
tempo de execução via `google.colab.userdata` (Colab Secrets). Antes de rodar, configure no
painel de *Secrets* do Colab (ícone de chave 🔑 na barra lateral):

| Nome do secret | Uso |
|---|---|
| `GROQ_API_KEY` | Acesso aos modelos candidatos via Groq |
| `Maritaca` | Acesso ao juiz (Maritaca `sabia-4`) |
| `PLANILHA_RESULTADOS_ID` | ID da planilha do Google Sheets usada para registrar os resultados |

O notebook também se conecta a uma planilha do Google Sheets (para registro dos resultados),
usando autenticação interativa (`google.colab.auth.authenticate_user()`). O ID da planilha
também é lido via Colab Secrets (`PLANILHA_RESULTADOS_ID`), não fica hardcoded no código —
nenhuma credencial ou identificador sensível fica salvo em texto no notebook.

## Checkpoints e retomada de execução

Todo o progresso é salvo incrementalmente em CSV no Google Drive
(`PASTA_PROJETO = /content/drive/MyDrive/PIBIC_grafos_conhecimento`):

- `checkpoint_resultados.csv` — extrações já realizadas.
- `checkpoint_avaliacao_juiz.csv` — avaliações do juiz já realizadas.
- Checkpoints equivalentes com sufixo `_TESTE` para o modo teste, mantidos separados dos
  resultados do experimento completo.

Cada execução é identificada por uma chave única `(drug, categoria, modelo, estrategia)`. Antes
de processar um item, o código verifica se ele já está no checkpoint e pula se sim — permitindo
interromper e retomar o experimento em qualquer ponto, inclusive em dias diferentes.

Erros de limite de taxa (HTTP 429) são tratados sem travar a execução: o modelo afetado é
marcado como esgotado na sessão atual e o pipeline segue para os próximos itens.

## Métricas e avaliação

**Automáticas** (calculadas por chamada de API):
- Número médio de tokens (prompt e resposta)
- Latência média por requisição
- Taxa de parsing válido das triplas extraídas

**LLM-as-a-judge** (nota 1–5 por critério):
- **Formato** — estrutura sujeito-predicado-objeto válida e coerente
- **Completude** — captura das relações relevantes do texto fonte
- **Corretude** — fidelidade ao texto fonte
- **Ausência de alucinação** — ausência de informação inventada

**Humana:** amostra de 1% dos resultados exportada em Excel, com colunas em branco para revisão
por especialista, usando seed fixa (`SEED = 42`) para reprodutibilidade.

## Limitações conhecidas

- O juiz (`sabia-4`, Maritaca) é de família diferente dos modelos candidatos avaliados, o que
  ajuda a mitigar (mas não elimina completamente) o risco de viés de avaliação.
- Os modelos `gpt-oss-120b` e `gpt-oss-20b` (quando usados) são modelos de raciocínio e
  consomem tokens internos de "pensamento" antes da resposta visível; o parâmetro
  `reasoning_effort` foi mantido no padrão em todos os modelos por comparabilidade, com
  `max_tokens` generoso (1500) para reduzir risco de respostas cortadas.
- O experimento completo depende de disponibilidade e limites de taxa da API Groq
  (especialmente do Developer Tier); instabilidades no upgrade de tier foram observadas durante
  o desenvolvimento (17/06/2026).

## Status do projeto

Pesquisa em andamento (bolsa PIBIC, Fatec Campinas). O notebook principal
(`PIBIC_Projeto_Final.ipynb`) reflete a versão mais atual do pipeline; versões anteriores e
experimentos exploratórios ficam fora deste repositório para manter o histórico limpo.

## Autor

Emmanuel — bolsista PIBIC, Fatec Campinas (Gestão da Tecnologia da Informação).
