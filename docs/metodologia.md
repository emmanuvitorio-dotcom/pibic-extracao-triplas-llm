# Metodologia detalhada

## Formato das triplas

As triplas extraídas seguem o formato `sujeito :: predicado :: objeto`, delimitado por `::`.
Exemplo de referência usado no notebook:

```
[DRUG]::bioavailability::56.9%
```

## Estratégias de prompting

- **Zero-Shot**: o modelo recebe apenas a instrução da tarefa e o texto fonte, sem exemplos.
- **Few-Shot**: o modelo recebe exemplos de triplas corretamente extraídas antes do texto fonte,
  para guiar o formato e o nível de granularidade esperado.

## Critérios de avaliação do juiz (LLM-as-a-judge)

Cada conjunto de triplas extraídas é avaliado pelo juiz (Maritaca `sabia-4`) em 4 critérios,
cada um numa escala de 1 a 5:

| Critério | O que avalia |
|---|---|
| **Formato** | As triplas seguem uma estrutura sujeito-predicado-objeto válida e coerente? |
| **Completude** | As triplas capturam as relações importantes presentes no texto fonte? |
| **Corretude** | As triplas refletem fielmente o que está escrito no texto fonte? |
| **Ausência de alucinação** | As triplas não inventam informação que não está no texto? |

## Amostragem para avaliação humana

- Fração amostrada: **1%** dos resultados com triplas extraídas.
- Seed fixa: `SEED = 42` (garante reprodutibilidade da amostra entre execuções).
- Formato de saída: planilha Excel com colunas em branco para preenchimento pelo especialista.
- Recomendação: rodar a amostragem apenas com o experimento completo (ou uma fração
  significativa dele) já processado, já que a amostra reflete o estado atual do checkpoint no
  momento da execução.

## Tratamento de falhas

- **HTTP 429 (cota/limite de taxa esgotado)**: o modelo afetado é marcado como "esgotado nesta
  sessão" e a execução segue para os demais modelos/itens, sem interromper o pipeline.
- **Erros transitórios** (timeout, falha de rede): tratados com retry simples.
- **Checkpoint incremental**: salvo periodicamente no Google Drive, permitindo retomar a
  execução exatamente de onde parou, mesmo após queda de sessão do Colab.

## Modelos de raciocínio (`gpt-oss-120b` / `gpt-oss-20b`)

Esses modelos usam tokens internos de "pensamento" antes de gerar a resposta visível, com nível
de esforço controlado por `reasoning_effort`. Para preservar comparabilidade entre todos os
modelos candidatos, o parâmetro foi mantido no valor padrão (não alterado seletivamente apenas
para esses modelos). Para compensar o consumo de tokens de raciocínio, `max_tokens` foi
configurado com margem generosa (1500) para reduzir o risco de respostas cortadas.
