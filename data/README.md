# Dados

O dataset usado neste projeto (`metabolism_absorption.xml`) contém textos farmacológicos sobre
metabolismo e absorção de ~2.628 drogas. Ele **não está incluído neste repositório** por ser um
arquivo grande e/ou de origem restrita.

## Como obter

- Se você é o autor do projeto: o arquivo original está salvo no Google Drive, na pasta
  `PIBIC_grafos_conhecimento`, e também pode ser re-enviado diretamente pela Seção 2 do notebook
  principal na primeira execução.
- Se você está reproduzindo o pipeline com outro dataset: qualquer XML no mesmo formato
  (elementos `<drug>` com sub-elementos de nome e texto de metabolismo/absorção) pode ser usado
  — ajuste o parsing na Seção 2 do notebook conforme necessário.

## Onde o notebook espera encontrar o arquivo

Por padrão, o notebook salva/lê o dataset em:

```
/content/drive/MyDrive/PIBIC_grafos_conhecimento/metabolism_absorption.xml
```

Ao rodar a Seção 2 pela primeira vez, se o arquivo não existir nesse caminho, o notebook solicita
o upload interativo e salva uma cópia permanente ali — nas execuções seguintes, o upload é
pulado automaticamente.

## Saídas geradas

Os checkpoints e resultados (CSV) e a planilha de amostra para revisão humana (Excel) também são
salvos na mesma pasta do Drive e não fazem parte deste repositório (dados de execução, não
código-fonte).
