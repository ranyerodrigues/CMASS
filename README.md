# Painel QMSS MOA/PCI

Painel HTML da reunião semanal de Qualidade, Meio Ambiente, Segurança e Saúde dos projetos
MOA e PCI. Abre com duplo clique, sem servidor, sem internet e sem Power BI: toda a leitura
da planilha acontece dentro do navegador.

## Por onde começar

| Arquivo | O que é |
|---|---|
| [`LEIAME.md`](LEIAME.md) | como abrir, como atualizar no mês seguinte e o que cada bloco mostra |
| [`MODELO-DE-PLANILHA.md`](MODELO-DE-PLANILHA.md) | estrutura esperada da planilha e checklist de preenchimento mensal |
| [`MAPEAMENTO.md`](MAPEAMENTO.md) | cada indicador → coluna de origem → regra de cálculo |
| [`VALIDACAO.md`](VALIDACAO.md) | o que foi conferido, divergências encontradas e melhorias por prioridade |

## Estrutura

```
index.html              página única da reunião — o entregável
replica.html            réplica fiel das três telas do Power BI, para conferência
css/style.css           tokens de cor e tipografia + estilos da réplica
css/painel.css          layout em blocos da página única
js/loader.js            camada 1 — leitura do Excel e tipagem
js/model.js             camada 2 — tratamento, verificações de qualidade e cálculo
js/charts.js            camada 3 — primitivas de gráfico em SVG
js/painel.js            camada 4 — página única
js/app.js               camada 4 — réplica
js/dados.js             instantâneo dos dados, para a página abrir já preenchida
vendor/                 SheetJS (Apache-2.0), leitor de .xlsx
assets/                 logotipos e ícones
ferramentas/            scripts de build (ver abaixo)
```

As quatro camadas são separadas de propósito: carregamento, tratamento, cálculo e
apresentação não se misturam. Nenhum valor é digitado nos gráficos — tudo vem da planilha.

## Build

Dois scripts, ambos sem dependência além de Python 3 (o de instantâneo usa `openpyxl`):

```bash
# regenera js/dados.js a partir de uma planilha
python3 ferramentas/gera-snapshot.py caminho/Dados_MOA.PCI_Atual_Com_Formulas.xlsx "Dados_MOA.PCI_Atual_Com Formulas.xlsx"

# regenera artifact.html (index.html com os CSS embutidos), usado na publicação
python3 ferramentas/gera-artifact.py
```

`artifact.html` é gerado e por isso não entra no versionamento. As planilhas também não:
são dado de projeto, e o repositório guarda só o código.

## Dados

O painel lê três abas — `DADOS`, `Índices` e `Ocorrências` — e ignora a `Check`. Formatos
anteriores da planilha continuam sendo aceitos; as equivalências estão em
`MODELO-DE-PLANILHA.md`.

Nada é gravado no navegador, nada sai da máquina de quem abre e a planilha original nunca é
alterada.
