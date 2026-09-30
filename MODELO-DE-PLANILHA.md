# Modelo padrão de planilha

Arquivo de referência: **`Dados_MOA.PCI_Atual_Com Formulas.xlsx`** (versão de consulta de
30/09/2026). É o formato que o painel espera quando você clica em **Atualizar dados**.
Quatro abas; o painel usa três e ignora a `Check`.

Cabeçalhos são comparados ignorando maiúsculas, acentos e espaços a mais — `HH MÊS`,
`hh mes` e `HH  Mês` são a mesma coluna. A ordem das colunas não importa; colunas extras
são ignoradas.

---

## Aba `DADOS` — base de eventos

Exportação do sistema, 124 colunas. O painel usa dez:

| Coluna | Uso no painel |
|---|---|
| `Número do Evento` | chave do registro; liga com `Nº do evento` de Ocorrências |
| `Classificação` | **níveis da pirâmide e foco** — `Desvios`, `ACDP`, `ASDP`, `PA`, `Incidente`; prever também `Fatal` deixa o topo da pirâmide pronto para quando houver |
| `Impacto` | **marcador Sev4 por nível da pirâmide** e verificação de qualidade |
| `Tipo de Evento` | rótulo auxiliar; era a base do foco antes de existir `Classificação` |
| `Data do evento` | recorte por período (aceita data ou texto `dd/mm/aaaa`) |
| `Descrição do Evento` | tabela do bloco 4 |
| `Descrição do Local Exato` | ranking "Locais" |
| `Fator de Risco / Aspecto N1` | ranking "Fatores de risco" |
| `Risco Potencial` | rosca do bloco 4 |
| `Contrato` | recorte por projeto (PCI / MOA) |

`Data_Mês_Evento` e as colunas `Ano` / `Mês` servem às fórmulas da aba Índices; o painel
recalcula o mês a partir de `Data do evento`.

**Cada linha de DADOS é um evento.** Uma linha lançada duas vezes conta duas vezes em todo
COUNTIFS da aba Índices, e portanto na pirâmide. Hoje há 21 números repetindo linhas
idênticas (ver `VALIDACAO.md`).

---

## Aba `Ocorrências` — eventos classificados, um por linha

É a lista de leitura do bloco 3. Desde esta versão os degraus da pirâmide **não** saem
daqui: saem de DADOS, pelas fórmulas da aba Índices.

| Coluna | Uso no painel | Valores observados |
|---|---|---|
| `Nº do evento` | referência ao registro em DADOS | — |
| `Data` | recorte por período | data |
| `Impacto` | coluna e filtro no bloco 3 | Alto, Médio, Sev4, Baixo, Baixo (NL), Ambiental |
| `Classificação` | foco por degrau nesta tabela | `ACDP`, `ASDP`, `PA`, `Incidente` |
| `Descrição` | tabela do bloco 3 | — |
| `Contrato` | recorte por projeto | PCI / MOA |
| `Ano`, `Mês`, `Texto` | auxiliares das fórmulas da aba Índices | — |

Todo `Nº do evento` lançado aqui precisa existir em DADOS. Hoje três não existem.

---

## Aba `Índices` — uma linha por projeto por mês, mais o bloco de total

| Coluna | Uso no painel |
|---|---|
| `PROJETO` | filtro de projeto |
| `Mês/ANO` | filtro de período — **sempre o primeiro dia do mês** |
| `HH MÊS` | divisor de todos os índices e cartão HH trabalhadas |
| `DESVIOS` | base da pirâmide |
| `ACDP`, `ASDP`, `PA`, `Incidente` | degraus da pirâmide |
| `Sev4` | numerador do IFS4 (antes chamada `S4`); confere o total dos marcadores da pirâmide |
| `Dias perdidos`, `Dias debitados`, `Acidentes graves` | numeradores de IS e MIFR |
| `IFA`, `IFT`, `IFCDP`, `IFSDP`, `IS`, `IFS4`, `IFPA`, `MIFR` | modo **Planilha** do bloco 2 |
| `DIAS SEM ACIDENTE`, `NOSSO RECORDE` | cartões de sequência |
| `Data Inicio`, `Data Fim`, `total dias` | não usados |

### O bloco `MOA + PCI` é um TOTAL, não um projeto

A aba traz três blocos: 25 linhas do PCI, 25 do MOA e 25 com `PROJETO = MOA + PCI`, que é a
soma mês a mês dos outros dois. O painel reconhece qualquer nome de projeto formado por
outros projetos unidos por `+` e trata essas linhas como total:

1. elas ficam **fora** de toda agregação, senão "os dois projetos" contaria tudo duas vezes;
2. `PROJETO` no filtro mostra só MOA e PCI;
3. no modo **Planilha**, quando a seleção cobre exatamente MOA e PCI, é esse bloco que o
   gráfico usa — somar as duas linhas de projeto somaria taxas.

Se você acrescentar outro total (`MOA + PCI + XYZ`, por exemplo), o painel o reconhece do
mesmo jeito, desde que cada parte do nome seja um `PROJETO` existente na própria aba.

### As colunas de índice vêm acumuladas por projeto

A fórmula é do tipo `=((SUM($E$2:E7)+SUM($F$2:F7))*1000000/SUM($C$2:C7))`, reiniciando em
cada bloco. Ou seja: cada linha traz o índice acumulado do projeto do começo do bloco até
aquele mês, não o valor do mês. Isso está **correto** e o acumulado do painel confere com
ele (IFA final: PCI 3,61, MOA 1,51, MOA + PCI 2,88). Duas consequências práticas:

1. essas colunas ignoram o filtro de período — por isso o modo padrão do painel é o
   Acumulado recalculado, que obedece a projeto **e** período;
2. se as linhas de um bloco forem reordenadas ou embaralhadas com outro projeto, os valores
   mudam. Mantenha um bloco contínuo por projeto, em ordem de mês.

O painel **não depende** das colunas de índice para os modos Acumulado e Mensal: ele
recalcula a partir de `ACDP`, `ASDP`, `PA`, `Dias perdidos`, `Dias debitados`,
`Acidentes graves`, `Sev4` e `HH MÊS`.

---

## Aba `Check` — conferência, ignorada pelo painel

Tabela dinâmica de DADOS por `Classificação` × `Impacto`. Serve de conferência manual: o
total geral (22.581) tem de ser o número de linhas de DADOS, e os subtotais têm de bater
com os degraus da pirâmide. Na versão atual batem exatamente: ACDP 1, ASDP 10, PA 8,
Incidente 41, Desvios 22.521.

---

## Compatibilidade com versões anteriores

O painel continua abrindo os dois formatos anteriores. Equivalências aceitas:

| Campo | Modelo atual | Aceito também |
|---|---|---|
| Índices | `ACDP` | `CPD` |
| Índices | `ASDP` | `SPD` |
| Índices | `Incidente` | `Incidentes` |
| Índices | `Sev4` | `S4` |
| Ocorrências | `Classificação` | `Tipo de ocorrência` |
| Ocorrências | `Contrato` | `Projeto` |

Sem a coluna `Impacto` em Ocorrências, o bloco 3 não mostra a coluna nem os botões de
filtro. Sem `Classificação` em DADOS, o foco por degrau volta a usar `Tipo de Evento` e
avisa na tela onde não há equivalente. Sem bloco de total, o modo Planilha volta a somar as
linhas dos projetos e a exibir o aviso de soma de taxas.

---

## Checklist de preenchimento mensal

1. Atualizar `DADOS` com a exportação do mês, mantendo `Contrato`, `Classificação`,
   `Impacto` e `Data_Mês_Evento` preenchidos em todas as linhas — e conferindo que nenhuma
   linha entrou duas vezes.
2. Lançar as ocorrências do mês em `Ocorrências`, com `Classificação` exatamente
   `ACDP`, `ASDP`, `PA` ou `Incidente`, `Impacto` preenchido e `Nº do evento` existente em
   DADOS. Em DADOS, `Impacto = Sev4` é o que alimenta o marcador de potencial de
   fatalidade em cada nível da pirâmide — grafar `S4` ou `SEV 4` faz o evento sumir do
   marcador.
3. Em `Índices`, acrescentar uma linha por projeto com `Mês/ANO` **no dia 1** e `HH MÊS`
   fechado, dentro do bloco do projeto, e a linha correspondente no bloco `MOA + PCI`. As
   colunas de contagem se preenchem sozinhas pelas fórmulas.
4. Preencher `Dias perdidos`, `Dias debitados` e `Acidentes graves` quando houver — hoje
   estão vazias nas 75 linhas, e por isso IS e MIFR ficam em zero.
5. Atualizar a aba `Check` e conferir que o total geral é igual ao número de linhas de
   DADOS.
6. Abrir o painel, clicar em **Atualizar dados** e conferir o botão
   **Qualidade dos dados**: ele lista o que ficou inconsistente.
