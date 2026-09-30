# Mapeamento dos indicadores

> **Base atual:** `Dados_MOA.PCI_Atual_Com Formulas.xlsx`. Nela, `CPD` virou `ACDP`,
> `SPD` virou `ASDP`, `Incidentes` virou `Incidente`, e a aba Ocorrências trocou
> `Tipo de ocorrência`/`Projeto` por `Classificação`/`Contrato`, ganhando `Impacto`,
> `Nº do evento`, `Ano`, `Mês` e `Texto`. O painel aceita os dois conjuntos de nomes —
> ver `MODELO-DE-PLANILHA.md`. Os cálculos abaixo não mudaram.

Origem das regras: `Report/Layout` do arquivo `Gestao_Projeto_MOA_PCI.pbix`
(definição de cada visual) e as três abas de `Dados_MOA.PCI.xlsx`.

**O modelo não tem nenhuma medida DAX.** Todos os visuais usam agregação direta de
coluna (`Sum`, `Máx`, `Contagem`). Não há coluna calculada visível no relatório: `IFA`,
`IFT`, `IFCDP`, `IFSDP`, `IS`, `IFS4`, `IFPA`, `MIFR`, `DIAS SEM ACIDENTE` e
`NOSSO RECORDE` já chegam calculados da planilha. O HTML **soma o que está gravado**,
exatamente como o Power BI — não recalcula os índices (mas confere as fórmulas; ver `VALIDACAO.md`).

As três tabelas do modelo: `Desvios` (é a aba **DADOS**, renomeada), `Índices` e
`Ocorrências`. Não existe relacionamento entre elas.

---

## Página Reativos — fonte: aba `Índices`

Filtros que afetam todos os visuais desta página: `PROJETO` e `Mês/ANO`.

| Visual | Campo no PBIX | Regra | Formatação |
|---|---|---|---|
| Cartão **ACDP** | `Índices[CPD]` | Soma | inteiro |
| Cartão **ASDP** | `Índices[SPD]` | Soma | inteiro |
| Cartão **PA** | `Índices[PA]` | Soma | inteiro |
| Cartão **INCIDENTES** | `Índices[Incidentes]` | Soma | inteiro |
| Cartão **DESVIOS TOTAIS** | `Índices[DESVIOS]` | Soma | inteiro |
| Cartão **NOSSO RECORDE** | `Índices[NOSSO RECORDE]` | **Máximo** (não é soma) | inteiro, fundo `#78B92E` |
| Linha **IFA** | `Índices[IFA]` | Soma por nível da hierarquia de `Mês/ANO` | eixo Y fixo −1 a 25, cor `#13A903` |
| Linha **IFSPD** | `Índices[IFSDP]` | Soma | −1 a 15, `#78B92E` |
| Linha **IFT** | `Índices[IFT]` | Soma | −1 a 25, `#2DA21E` |
| Linha **IS** | `Índices[IS]` | Soma | −1 a 200, `#13A903` |
| Linha **IFCPD** | `Índices[IFCDP]` | Soma | −1 a 25, `#13A903` |
| Linha **IFS4** | `Índices[IFS4]` | Soma | −1 a 25, `#13A903` |
| Linha **IFPA** | `Índices[IFPA]` | Soma | −1 a 15, `#2DA21E` |
| Linha **MIFR** | `Índices[MIFR]` | Soma | −1 a 1, eixo Y oculto, `#2DA21E` |
| Tabela de ocorrências | aba `Ocorrências`: `Data`, `Descrição`, `Tipo de ocorrência`, `Projeto` | linhas detalhadas, ordem `Data` decrescente | data dd/mm/aaaa |

Fórmulas impressas nas caixas de texto do próprio relatório (reproduzidas na tela):

```
IFA   = (ACDP + ASDP) x 1.000.000 / HHT
IFT   = (ACDP + ASDP + PA) x 1.000.000 / HHT
IFCDP = ACDP x 1.000.000 / HHT
IFSDP = ASDP x 1.000.000 / HHT
IS    = (DP + DD) x 1.000.000 / HHT
IFS4  = S4 x 1.000.000 / HHT
IFPA  = PA x 1.000.000 / HHT
MIFR  = AG x 1.000.000 / HHT
```

Equivalência dos nomes: `CPD` = ACDP (com perda de dias), `SPD` = ASDP (sem perda de dias),
`HH MÊS` = HHT, `Dias perdidos` = DP, `Dias debitados` = DD, `Acidentes graves` = AG.
Quatro caixas do PBIX estão com o rótulo errado (IFT, IS e MIFR aparecem como "IFSDP");
foram mantidas como estão — ver `VALIDACAO.md`.

---

## Página única de reunião — o que difere

Os cálculos dos cartões, da rosca, dos rankings e das tabelas são os mesmos das tabelas
acima. Dois pontos são próprios desta página:

**Índices acumulados (modo padrão).** A série NÃO soma os índices já gravados na planilha.
Um índice de frequência é uma razão e não pode ser somado nem ter média tirada. O ponto do
mês *m* é recalculado a partir das colunas de contagem e de `HH MÊS`:

```
índice acumulado(m) = (Σ ocorrências do 1º mês do recorte até m × 1.000.000)
                      ÷ (Σ HH MÊS do 1º mês do recorte até m)
```

| Indicador | Numerador acumulado | Colunas de Índices |
|---|---|---|
| IFA | ACDP + ASDP | `CPD` + `SPD` |
| IFT | ACDP + ASDP + PA | `CPD` + `SPD` + `PA` |
| IFCDP | ACDP | `CPD` |
| IFSDP | ASDP | `SPD` |
| IS | DP + DD | `Dias perdidos` + `Dias debitados` |
| IFS4 | S4 | `S4` |
| IFPA | PA | `PA` |
| MIFR | AG | `Acidentes graves` |

Denominador em todos: `Σ HH MÊS`. Como ocorrências e HHT são somados **antes** da divisão,
o acumulado é válido com vários projetos selecionados.

O modo **Mensal** usa a mesma origem sem acumular: `ocorrências do mês × 1.000.000 ÷ HHT do
mês`. Meses com `HH MÊS` abaixo de 1.000 h ficam sem ponto — nesses meses o divisor é
marcador, não hora trabalhada.

O modo **Planilha** lê a coluna gravada, sem recálculo. Na base atual essa coluna já vem
acumulada por fórmula (`=SUM($E$2:E12)*1000000/SUM($C$2:C12)`) e a soma varre a coluna
inteira, misturando PCI e MOA — o valor gravado numa linha depende da posição física dela.
Use esse modo só para conferir contra o arquivo.

`Dias perdidos`, `Dias debitados` e `Acidentes graves` estão vazias em todas as linhas
da planilha, então IS e MIFR ficam estruturalmente em zero — a página avisa isso na nota
abaixo do gráfico.

**Linhas de TOTAL.** A aba `Índices` da base atual traz um terceiro bloco com
`PROJETO = MOA + PCI`, que é a soma mês a mês dos outros dois. O painel detecta qualquer
`PROJETO` cujo nome seja uma união (`+`) de projetos existentes na própria aba e o trata como
total: fora de toda agregação, fora do filtro de `Projeto`, e usado no modo **Planilha**
quando a seleção cobre exatamente os projetos citados no nome. Sem isso, marcar MOA e PCI
contaria tudo duas vezes.

**Gráfico de eventos do bloco 2.** Abaixo do gráfico do índice, o numerador aberto nos seus
componentes, uma linha acumulada por grupo: IFA → ACDP e ASDP; IFT → ACDP, ASDP e PA; IS →
dias perdidos e dias debitados; os demais, uma linha só. Cada linha vem de
`Soma(Índices[<coluna>])` acumulada no eixo de tempo do recorte, e a cor é a do nível
correspondente da pirâmide. Conferido: as linhas do IFA fecham em 1 e 10, cuja soma é o
numerador (11) que produz o IFA de 2,88.

**Marcador Sev4 por nível.** Novo. A aba Índices tem a coluna `Sev4`, mas só com o total
do mês — não dá para saber quanto dele é ACDP, ASDP, PA, incidente ou desvio. A quebra vem
de DADOS, contando as linhas em que `Impacto = Sev4` para cada valor de `Classificação`, no
mesmo recorte de projeto e período dos níveis. Conferido: 1 + 4 + 4 + 4 + 182 = **195**, que
é exatamente `Soma(Índices[Sev4])` no mesmo recorte; com MOA sozinho, 0 + 0 + 0 + 2 + 34 =
**36**, igual à coluna. Sem `Classificação` e `Impacto` em DADOS, o marcador não aparece.

**Pirâmide de Bird.** Os cinco níveis usam exatamente os mesmos valores dos antigos
cartões — `Soma(Índices[CPD])`, `[SPD]`, `[PA]`, `[Incidentes]` e `[DESVIOS]` no recorte.
Só a apresentação mudou: largura fixa e crescente por degrau (a proporção real, de 1 para
22.521, não é desenhável) e número em ordem de grandeza acima de mil (22.521 → "22,5 Mil";
3.817.009 h → "3,8 Mi"), com o valor exato no tooltip. Nenhuma agregação foi alterada.
Na base atual os cinco degraus somam exatamente as 22.581 linhas de DADOS.

**Foco por degrau.** Clicar em ACDP, ASDP, PA, INCIDENTES ou DESVIOS filtra os blocos de
evento. As ligações foram conferidas contra a planilha, somando todos os projetos e todo o
período:

| Degrau | Valor em Índices | Ocorrências `Classificação` | DADOS `Classificação` |
|---|---|---|---|
| FATAIS | sem coluna | `Fatal` / `Óbito` (nenhum registro hoje) | `Fatal` / `Óbito` (nenhum registro hoje) |
| ACDP | `ACDP` = 1 | `ACDP` = 1 ✓ | `ACDP` = 1 ✓ |
| ASDP | `ASDP` = 10 | `ASDP` = 9 | `ASDP` = 10 ✓ |
| PA | `PA` = 8 | `PA` = 8 ✓ | `PA` = 8 ✓ |
| INCIDENTES | `Incidente` = 41 | `Incidente` = 41 ✓ | `Incidente` = 41 ✓ |
| DESVIOS | `DESVIOS` = 22.521 | sem equivalente | `Desvios` = 22.521 ✓ |

Na base atual as cinco contagens de Índices são calculadas por `COUNTIFS` sobre a coluna
`Classificação` da aba **DADOS** (antes era sobre Ocorrências), então o bloco 4 fecha exato
com o degrau. Restam duas divergências, ambas exibidas na tela:

1. **ASDP 10 contra 9 em Ocorrências** — o evento 11252 (04/12/2025) aparece duas vezes em
   DADOS, em linhas idênticas. O COUNTIFS conta duas; a lista de ocorrências, uma.
2. **DESVIOS 22.521 contra 22.520 no foco** — a linha com `Contrato` minúsculo `moa`. O
   COUNTIFS ignora caixa e a pega; o filtro de projeto do painel, não.

Em planilhas sem a coluna `Classificação` em DADOS, o foco do bloco 4 cai no `Tipo de
Evento`: ACDP, ASDP e PA ficam sem equivalente, INCIDENTES casa com 44 registros e DESVIOS
com 22.511.

Onde não existe equivalente, o bloco **não é filtrado** e a tela diz isso. Onde os dois
lados divergem (INCIDENTES e DESVIOS), o chip de foco mostra as duas contagens. Nenhuma
ligação foi inventada: cada uma acima tem a contagem conferida ou está marcada como
inexistente.

**Ocorrências.** A tabela traz `Data`, `Impacto`, `Classificação`, `Contrato` e
`Descrição`. Os botões de Impacto filtram por contagem simples da coluna; nenhum marcado =
todos. O filtro de `EXCLUÍDO` / `REJEITADO` continua no código para arquivos antigos — na
base atual não há registro com essas classificações, então nada é escondido.

---

## Página Proativos — fonte: aba `DADOS` (tabela `Desvios`)

Filtros que afetam todos os visuais: `Contrato` e `Data do evento` (Ano ▸ Mês),
mais o realce cruzado.

| Visual | Campo no PBIX | Regra | Cores |
|---|---|---|---|
| Rosca **Risco Potencial** | categoria `DADOS[Risco Potencial]`, valor = Contagem da mesma coluna | contagem de não vazios por categoria, ordem decrescente | MÉDIO `#75AAAE`, BAIXO `#AD5129`, ALTO `#148030`, ALTO - SEV4 `#FF1010`; demais categorias usam a sequência do tema |
| Colunas **Quantidade de desvios por local** | `DADOS[Descrição do Local Exato]` | contagem por categoria, ordem decrescente | `#78B92E` |
| Barras **Fator de Risco** | `DADOS[Fator de Risco / Aspecto N1]` | contagem por categoria, ordem decrescente | `#0C63B3` |
| Tabela | `DADOS[Número do Evento]` (sem agregação) e `DADOS[Descrição do Evento]` | linhas detalhadas, ordem crescente por descrição | coluna do número com 79 px, como no PBIX |

**Não há filtro de `Tipo de Evento`** — nem no relatório nem aqui. A página conta os
22.581 registros da aba DADOS (22.511 Desvios, 44 Incidentes, 20 Ocorrências, 6 Acidentes),
embora os títulos falem em "desvios". Decisão confirmada: manter como o relatório.

---

## Cores e tipografia

O relatório usa o tema base **CY23SU04** do Power BI, sem tema customizado. Cores
literais vieram do próprio arquivo; as derivadas foram calculadas com a fórmula de
tonalidade do Power BI (`sombra = c × (1 + p)`, `clareamento = c + (255 − c) × p`):

| Onde | Origem | Resultado |
|---|---|---|
| Verde da marca | literal | `#78B92E` |
| Borda dos rótulos da pirâmide | literal | `#B4EF56` |
| Rosca — MÉDIO | tema[8] `#197278` +40% | `#75AAAE` |
| Rosca — BAIXO | tema[2] `#E66C37` −25% | `#AD5129` |
| Rosca — ALTO | tema[9] `#1AAB40` −25% | `#148030` |
| Barras Fator de Risco | tema[0] `#118DFF` −30% | `#0C63B3` |
| Legenda "Limite" | tema[3] | `#6B007B` |
| Fundo da página Início | tema[0] com 16% de transparência | `#379FFF` — **pendente de confirmação** |

Tamanhos de fonte em pontos, convertidos a 1 pt = 4/3 px: título 18 pt, rótulos de
página 16 pt, título de gráfico 10 pt, eixos e fórmulas 8 pt, rótulos da pirâmide 9 pt,
cartões 19 pt, cartão do recorde 27 pt. Fonte: Segoe UI (padrão do tema).
Datas em dd/mm/aaaa, números no padrão pt-BR (`.` milhar, `,` decimal); índices com
2 casas, contagens inteiras. Nenhum indicador é monetário.
