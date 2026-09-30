# Validação, limitações e melhorias

## 0. Base de consulta atual — `Dados_MOA.PCI_Atual_Com Formulas.xlsx` (30/09/2026)

Esta é a planilha que alimenta o painel hoje e o modelo de preenchimento mensal
(especificação em `MODELO-DE-PLANILHA.md`). Suíte própria no navegador real,
**81 verificações, 0 falha**, com cálculo independente em Python direto no arquivo.
Somando as seis suítes: **223 verificações, 0 falha**.

**Leitura** — 22.581 linhas em DADOS, **75** em Índices (50 de projeto + **25 de total**),
59 em Ocorrências, e uma aba nova `Check` que o painel ignora.

| Recorte | ACDP | ASDP | PA | Incidente | DESVIOS | HH MÊS | Ocorrências | Recorde | IFA acum. |
|---|---|---|---|---|---|---|---|---|---|
| MOA + PCI | 1 | 10 | 8 | 41 | 22.521 | 3.817.009,14 | 59 | 730 | 2,88 |
| MOA | 0 | 2 | 0 | 7 | 7.376 | 1.323.225,24 | 8 | 730 | 1,51 |
| PCI | 1 | 8 | 8 | 34 | 15.145 | 2.493.783,90 | 51 | 579 | 3,61 |

Os cinco degraus somam **1 + 10 + 8 + 41 + 22.521 = 22.581**, exatamente o número de linhas
de DADOS: toda linha cai em um degrau, nenhuma fica fora nem entra duas vezes. Os subtotais
batem um a um com a aba `Check` da própria planilha.

**O que mudou de estrutura**

1. **DADOS ganhou `Classificação`, `Impacto` e `Data_Mês_Evento`** (121 → 124 colunas). É de
   `Classificação` que saem, por COUNTIFS, as colunas de contagem da aba Índices — e agora
   também o foco por degrau, que antes tentava adivinhar pelo `Tipo de Evento`. Por isso os
   degraus fecham exatos: ACDP 1 = 1, ASDP 10 = 10, PA 8 = 8, Incidente 41 = 41.
2. **A aba Índices ganhou um terceiro bloco, `PROJETO = MOA + PCI`**, que é a soma mês a mês
   dos outros dois. O painel reconhece e isola essas 25 linhas: elas não entram em nenhuma
   agregação (senão "os dois projetos" contaria tudo duas vezes), o filtro de PROJETO mostra
   só MOA e PCI, e no modo Planilha com os dois selecionados é esse bloco que responde.
3. **`S4` virou `Sev4`** na aba Índices; os dois nomes são aceitos.
4. **As colunas de índice passaram a acumular por projeto** — a soma reinicia em cada bloco.
   Era o principal problema da versão anterior, e está resolvido. Consequência: o Acumulado
   recalculado do painel agora **coincide** com a coluna gravada quando o período é todo o
   histórico (2,88 / 3,61 / 1,51), o que é a melhor conferência possível dos dois lados.

**Correções confirmadas em relação à versão anterior**

| Achado anterior | Estado |
|---|---|
| Índices misturando projetos na acumulação | corrigido (acumula por bloco) |
| `PCI 27/02/2025` com `Mês/ANO` fora do dia 1 | corrigido (01/02/2025) |
| `NOSSO RECORDE` como cópia de `DIAS SEM ACIDENTE` | corrigido (`MÁX` acumulado) |
| `IFCDP` de `PCI fev/25` com referência deslocada | não reaparece |
| `EXCLUÍDO` / `REJEITADO` / `TRAJETO` em Ocorrências | não existem mais (0 de 59 ocultos) |

**Os três modos do bloco 2, conferidos ponto a ponto:** Acumulado fecha em **2,88**
(11 acidentes ÷ 3,82 Mi HHT) e é consistente em todos os pontos
(`y = numerador acumulado × 1e6 ÷ HHT acumulado`); Planilha com os dois projetos usa as 25
linhas do bloco de total e dá o mesmo **2,88**; Mensal omite 1 mês (set/2026, HHT somado
abaixo de 1.000 h). Com um projeto só, Acumulado e Planilha também coincidem (PCI 3,61,
MOA 1,51).

**Foco por degrau, agora pela `Classificação` de DADOS:** ACDP → 1 ocorrência e 1 linha de
DADOS; ASDP → 9 e 10; PA → 8 e 8; INCIDENTES → 41 e 41; DESVIOS → 22.520 linhas de DADOS.
As duas diferenças estão explicadas na tela: o ASDP a mais é uma linha em dobro em DADOS, e
o desvio a menos é a linha com `Contrato` minúsculo `moa`, que o COUNTIFS pega e o filtro de
projeto do painel não.

**Pirâmide redesenhada e marcador Sev4 (suíte `bird`, 32 verificações).** Cada nível tem
cartão com sigla e definição à esquerda e a fatia da pirâmide à direita, do vértice (0%) até
a base (100%). O marcador Sev4 conta, por nível, as linhas de DADOS com `Impacto = Sev4` no
mesmo recorte: **1 · 4 · 4 · 4 · 182 = 195**, exatamente `Soma(Índices[Sev4])`; com MOA
sozinho, **0 · 0 · 0 · 2 · 34 = 36**, também igual à coluna. Conferido em 1.280, 900 e 420 px:
sem rolagem horizontal, número do vértice dentro da fatia e cartão com largura útil.

**Nível de acidentes fatais no topo da pirâmide.** Incluído a pedido e em zero: não houve
acidente fatal no período. O nível conta qualquer grafia de `Fatal` ou `Óbito` em
`Classificação` — coluna que hoje assume só Desvios, ACDP, ASDP, PA e Incidente. O painel de
Qualidade dos dados registra, como informação, que a coluna precisa aceitar `Fatal` para um
acidente futuro ser contado sozinho.

**Segundo gráfico do bloco 2: os eventos, uma linha por grupo.** O numerador do índice,
aberto nos seus componentes e acumulado. Conferido com os dois projetos e todo o histórico:
IFA abre em **ACDP = 1** e **ASDP = 10**, que somados dão os 11 do numerador que produz o IFA
de 2,88; IFT abre em ACDP 1, ASDP 10 e PA 8. As duas linhas são monotônicas, como tem de ser
num acumulado. Legenda presente e nome escrito no último ponto de cada linha, então a
identidade não depende da cor. É um gráfico separado, e não um eixo duplo: taxa e contagem
têm escalas diferentes e sobrepor as duas num eixo só esconderia isso.

**Modos Mensal e Planilha removidos da tela.** Os dois cálculos continuam no modelo e na
suíte: `base` confere que a coluna gravada e o Acumulado recalculado coincidem em todo o
histórico (2,88 / 3,61 / 1,51), e `ajustes` confere o mensal ponto a ponto. O que saiu foram
os botões, por serem conferência e não leitura de reunião.

**Identidade Techint (suíte `base`).** Tokens `--marca` `#007DC3` e `--marca-forte`
`#002B5C` lidos do esquema TECHINT-02 do template, tipografia Arial/Calibri do fontScheme,
e o botão selecionado renderizando `rgb(0, 125, 195)`. A paleta da pirâmide e a escala de
severidade da rosca foram validadas pelo verificador de paletas em tema claro e escuro
(faixa de luminosidade, piso de croma, separação para daltonismo e contraste contra a
superfície). Duas ressalvas registradas: no tema escuro o roxo fica em 2,93:1 contra a
superfície e o par azul/roxo dá ΔE 7,8 para deuteranopia — as duas são cobertas pelo
rótulo escrito em cada nível, que é o recurso de apoio exigido, e pelo vão de 3 px entre
fatias. `replica.html` mantém cores e tipografia do `.pbix`.

**Compatibilidade:** os dois formatos anteriores continuam abrindo. A versão intermediária
dá 1 / 9 / 8 / 41 / 22.581 sem bloco de total; a original dá 1 / 7 / 11 / 44 / 22.581 sem
coluna Impacto e sem Classificação em DADOS, com o foco voltando ao `Tipo de Evento`.

## 1. O que foi conferido na base anterior (`Dados_MOA.PCI.xlsx`)

Suíte automatizada com o navegador real (Chromium), carregando o `Dados_MOA.PCI.xlsx`
e comparando os valores da tela com um cálculo independente feito em Python direto na
planilha. **56 verificações, 0 falha.** Os números desta seção são da base anterior e
estão mantidos como registro histórico — a seção 0 traz os da base atual.

**Leitura** — 22.581 linhas em DADOS, 50 em Índices, 66 em Ocorrências; 26 itens no
slicer Mês/ANO; 2 projetos; 3 grafias de contrato.

**Reativos, PROJETO = MOA (seleção salva no PBIX), todos os meses**

| Indicador | Valor | Confere com a planilha |
|---|---|---|
| ACPD | 0 | sim |
| ASDP | 1 | sim |
| PA | 0 | sim |
| INCIDENTES | 9 | sim |
| DESVIOS TOTAIS | 7.385 | sim |
| NOSSO RECORDE (Máx) | 730 | sim |
| Soma de HH MÊS (controle) | 1.323.225,24 | sim |
| Ocorrências (Projeto = MOA) | 11 | sim |

As **8 séries de linha** (IFA, IFSDP, IFT, IS, IFCDP, IFS4, IFPA, MIFR) foram comparadas
ponto a ponto, nos 25 meses — todas idênticas.

**Reativos, PCI + MOA**: ACPD 1, ASDP 7, PA 11, INCIDENTES 44, DESVIOS 22.581, recorde 730,
66 ocorrências, série IFA idêntica.

**Reconciliação cruzada relevante:** `Soma(Índices[DESVIOS])` dos dois projetos = **22.581**,
exatamente o número de linhas da aba DADOS. Ou seja, a coluna DESVIOS foi montada sobre
**todos** os tipos de evento, não só sobre "Desvio" — o que confirma a decisão de não
filtrar `Tipo de Evento` na página Proativos.

**Proativos, Contrato = PCI + MOA (seleção salva no PBIX)**: 22.580 registros.
Rosca: MÉDIO 14.781, BAIXO 5.418, ALTO 2.180, ALTO - SEV4 193, (vazio) 6,
ALTO - Relevante 1, MEDIO 1. As 25 maiores categorias de Fator de Risco conferem uma a
uma. Tabela com 22.580 linhas, colunas na ordem do PBIX (Número do Evento, Descrição).
Incluindo a grafia `moa`: 22.581.

**Filtros** — testados isolados, combinados, com limpeza de seleção e sem dados:
um único mês (MOA set/2026 → 1 linha, 86 desvios, 1 ponto no gráfico, 0 ocorrências);
seleção sem correspondência (cartões viram `—`, gráficos e tabelas exibem aviso);
níveis do eixo (Ano 3 pontos, Trimestre 9, Mês 25); realce cruzado por ALTO
(2.180 registros em todos os visuais, e a rosca não se filtra a si mesma).

**Visual** — nenhum visual ultrapassa o canvas de 1280×720; nenhum erro de console;
inspeção das capturas das três páginas em 100% de zoom.

### Página única de reunião (`index.html`, layout próprio)

Suíte separada, **36 verificações, 0 falha**: estrutura dos 4 blocos e dos controles;
cartões, HH e série idênticos aos da réplica no mesmo recorte; sequência e recorde por
projeto; o mesmo recorte chegando aos quatro blocos (25 linhas de Índices, 11 ocorrências,
7.384 registros de DADOS com Projeto = MOA); soma da rosca igual ao total de registros;
troca de projeto e de período; realce cruzado restrito ao bloco 4; agrupamento de
variações; recorte sem dados; ausência de rolagem horizontal em 1280, 900 e 420 px; e
confirmação de que `replica.html` continua com os mesmos números.

**Ajustes verificados em suíte separada (20 verificações, 0 falha):** filtro de EXCLUÍDO e
REJEITADO na tabela de ocorrências (66 no total, 4 ocultos, 62 visíveis; botão devolve os
ocultos e volta a escondê-los); índice acumulado com `y = numerador acumulado × 1.000.000
÷ HHT acumulado` conferido ponto a ponto, numerador e HHT monotônicos; PCI fecha em 7
ocorrências (1 ACDP + 6 ASDP) sobre 2.493.784 HHT → IFA 2,81; a diluição depois do pico de
fev/25 confirmada mês a mês; com os dois projetos, 8 ocorrências sobre 3.817.009 HHT →
IFA 2,10, sem o aviso de soma de taxas (que reaparece no modo mensal); detecção das colunas
vazias que zeram IS e MIFR; e a réplica inalterada (11 ocorrências, sem filtro de tipo).

**Seleção de mês e foco por cartão (34 verificações, 0 falha):** grade de meses com 3 anos
× 12, 25 meses habilitados e 11 apagados; fevereiro/2025 sozinho traz as 2 linhas
(`01/02` do MOA e `27/02` do PCI); combinações de meses e retorno ao atalho Tudo; os 5
cartões clicáveis e HH não clicável; um único foco ativo por vez; ACDP→1, ASDP→7, PA→11
ocorrências, com o gráfico pulando para IFCDP, IFSDP e IFPA; INCIDENTES com cartão 44,
DADOS 44 e ocorrências 41 (divergência exibida); DESVIOS filtrando DADOS para
`Tipo de Evento = Desvio`; blocos sem equivalente ficando sem filtro, com aviso;
`limpar foco` restaurando o recorte; foco combinado com mês exato; e a réplica inalterada.

**Pirâmide de Bird (20 verificações, 0 falha):** ordem dos cinco degraus, larguras
contínuas de 30% a 100% (cada degrau começando onde o anterior termina), valores
idênticos aos dos cartões anteriores, formato de ordem de grandeza (`22,6 Mil`;
`3,8 Mi` para HH) com o valor exato preservado no tooltip, HH fora da pirâmide e não clicável,
clique em cada degrau filtrando e marcando só ele, recorte com zeros (MOA: ACDP 0,
ASDP 1) e ausência de rolagem horizontal com o texto do topo cabendo dentro do trapézio
em 1280, 900 e 420 px.

**Topo fixo (13 verificações, 0 falha):** em 1600×950 o bloco inteiro (barra, capa e
filtros) fica colado no alto após rolar 2.000 px, ocupando 169 px (18% da tela); em
420×860 barra e capa saem e fica presa só a faixa de filtros, 201 px (23%); em ambos os
casos sem rolagem horizontal, com os botões de projeto acessíveis sem voltar ao topo, e
o título do bloco 3 parando abaixo do topo fixo ao clicar num degrau da pirâmide.

Reconciliações adicionais: no recorte PCI+MOA dos últimos 3 meses, `Soma(Índices[DESVIOS])`
= **1.212** e o número de linhas de DADOS no mesmo recorte = **1.212**. Com MOA e todo o
histórico, a soma das fatias da rosca = **7.384** = registros de DADOS do recorte.

Três diferenças deliberadas nesta página, todas visíveis na tela:

1. **Dias sem acidente** é o valor do **mês mais recente do recorte** (a sequência atual),
   e **recorde** é o **máximo de todo o histórico** daquele projeto. O cartão da página
   Reativos continua sendo `Máx(NOSSO RECORDE)` no contexto, como no PBIX. São números
   diferentes de propósito: um responde "estamos há quantos dias sem acidente", o outro
   reproduz o relatório.
2. **As fórmulas exibidas estão com a sigla correta** (no PBIX quatro caixas trazem
   "IFSDP" trocado). A réplica mantém o texto errado; esta página, não.
3. **Um recorte só para as três tabelas:** Projeto e Período valem ao mesmo tempo para
   Índices, Ocorrências (por `Projeto` + ano/mês da `Data`) e DADOS (por `Contrato` +
   ano/mês da `Data do evento`). No modelo do PBIX essa ligação não existe. Efeito prático:
   começando em MOA, o bloco 4 mostra **7.384** registros, e não os **22.580** da página
   Proativos do Power BI, que ignora o filtro de projeto dos outros visuais.
4. **HH trabalhadas** entrou como sexto cartão: é `Soma(Índices[HH MÊS])`, o divisor de
   todos os índices de frequência. Não existe como visual no PBIX.
5. **Índices acumulados ponderados por HHT** no modo padrão do bloco 2, recalculados das
   colunas de contagem e de `HH MÊS` em vez de somar os índices gravados. O Power BI só
   mostra o valor mensal. Isso corrige, nesta página, o problema de somar taxas entre
   projetos. O modo **Mensal** reproduz o comportamento do relatório.
6. **A tabela de ocorrências esconde `EXCLUÍDO` e `REJEITADO`** (4 dos 66 registros), com
   botão para exibi-los. A réplica continua mostrando os 66.
7. **Clicar num degrau da pirâmide filtra os blocos de evento** pelo tipo equivalente
   (tabela em `MAPEAMENTO.md`). O Power BI não liga os cartões de Índices às outras
   tabelas — não há relacionamento no modelo. A ligação aqui é por nome de tipo, com as
   contagens conferidas uma a uma e exibidas na tela; onde não existe equivalente, o bloco
   não é filtrado e diz isso.
8. **Seleção de mês exato** por uma grade de calendário, além dos atalhos de período.
9. **O realce cruzado é restrito ao bloco 4.** Risco, fator e local só existem em DADOS;
   propagá-los para os cartões de Índices não teria significado.

## 2. O que NÃO foi possível verificar

1. **Não abri o relatório no Power BI.** A comparação foi feita contra a *definição* dos
   visuais dentro do `.pbix` (arquivo `Report/Layout`, que é JSON legível), não contra a
   tela renderizada.
2. **O modelo tabular (`DataModel`) não é legível neste ambiente** — é um backup comprimido
   do Analysis Services. Não consegui ler o Power Query (M) das três consultas. Se houver
   filtro ou coluna calculada na origem que não aparece no relatório, ele não está
   reproduzido aqui. O que dá para afirmar: o relatório não usa nenhuma medida DAX, e o
   teste de reconciliação do item anterior indica que não há filtro de `Tipo de Evento`.
3. **Nível do eixo dos 8 gráficos de linha.** O PBIX coloca a hierarquia de datas completa
   (Ano ▸ Trimestre ▸ Mês ▸ Dia) no eixo e não grava o nível de detalhamento aberto. Adotei
   **Mês**, porque as faixas fixas do eixo Y (25, 15, 200) só fazem sentido com valores
   mensais — somados por ano eles estouram a escala. O seletor **Eixo dos gráficos** na
   barra superior troca para Trimestre ou Ano, se no Power BI estiver diferente.
4. **Cor de fundo da página Início.** O arquivo declara papel de parede com a cor 0 do
   tema (`#118DFF`) e 16% de transparência, o que resulta no azul que você vê. É o que
   está no arquivo, mas é estranho num relatório de identidade verde, e o título verde
   sobre azul tem contraste ruim. **Me mande um print da página Início do Power BI** e eu
   acerto em uma linha (`--fundo-inicio` em `css/style.css`).
5. **Rótulos da rosca.** O PBIX não grava o estilo dos rótulos de dado da rosca; usei
   categoria na legenda + percentual dentro da fatia.

## 3. Divergências conscientes em relação ao PBIX

| # | No Power BI | Aqui | Motivo |
|---|---|---|---|
| 1 | A tabela de ocorrências **não responde** aos slicers (não há relacionamento possível entre `Ocorrências` e `Índices`) | Responde a PROJETO e Mês/ANO, cruzando `Projeto` e ano/mês da `Data` | Decisão sua. Sem isso, cartões e tabela mostram recortes diferentes na mesma tela |
| 2 | Eixo Y fixo corta o que passa do topo (ex.: IFS4 chega a 191 com eixo até 25) | O eixo é ampliado quando o dado ultrapassa a faixa, e o título do eixo ganha um `*` | O comportamento original esconde exatamente os meses piores |
| 3 | Clique num ponto de dado **realça** os outros visuais (mantém o total em cinza) | Clique **filtra** os outros visuais | Realce parcial exigiria redesenhar todas as primitivas; o efeito prático de leitura é o mesmo |
| 4 | Gráfico de local desenha milhares de colunas com rolagem | Desenha as **60 maiores**, com rolagem, e informa quantas categorias existem | 15.528 colunas travam o navegador; nenhuma agregação foi alterada |
| 5 | Rótulos de dado sobrepostos são ocultados por densidade interna | Ocultados quando colidiriam | Mesma ideia, implementação própria |
| 6 | Um `lineChart` vazio, sem consulta, empilhado atrás do gráfico de IFA | Não reproduzido | Visual sem dado nenhum, provável sobra de edição |
| 7 | 35 hexágonos na página Início, dois deles exatamente na mesma posição | 34 desenhados | Um é duplicata perfeita do outro |

## 4. Defeitos do relatório original **mantidos** na réplica

Reproduzidos porque a fidelidade foi pedida — e listados aqui porque valem correção no
PBIX:

1. **Todos os 8 gráficos de linha têm o eixo Y intitulado "IFA"**, inclusive os de IS,
   IFS4, IFPA e MIFR.
2. **Quatro caixas de fórmula estão com a sigla errada**: o gráfico de IFT mostra
   "IFSDP = ACDP + ASDP + PA…", o de IS mostra "IFSDP = DP + DD…", o de MIFR mostra
   "IFSDP = AG…" e a fórmula do IFA aparece sem os parênteses de `(ACDP + ASDP)`.
3. **A legenda "Limite / Resultado" não tem série de limite** em gráfico nenhum, e as
   cinco colunas `Referência ...` da planilha estão todas zeradas. Hoje a legenda promete
   algo que a tela não entrega.
4. **A página Início é o único caminho de navegação**, e os dois botões ficam em blocos
   agrupados sem rótulo de acessibilidade.

## 5. Achados de qualidade dos dados

Todos calculados em tempo de carga e visíveis no botão **Qualidade dos dados**. Nada foi
corrigido automaticamente.

### Achados da base de consulta atual (30/09/2026)

**Graves**

A. **21 números de evento repetem linhas IDÊNTICAS em DADOS** — mesma classificação, mesmo
   contrato, mesma data, mesma descrição. Não são dois eventos com o mesmo número: é a mesma
   linha lançada duas vezes. Como as contagens da aba Índices são COUNTIFS sobre DADOS, essas
   linhas extras entram nos degraus da pirâmide. Efeito medido: **+1 em ASDP** (o evento
   11252, de 04/12/2025) e **+3 em Incidente** (101027, 103049, 105945), além de **+17 em
   DESVIOS**. Números corretos, tirando as duplicatas: ACDP 1, ASDP 9, PA 8, Incidente 38,
   Desvios 22.504 — 22.560 eventos distintos em 22.581 linhas. É por isso que o degrau ASDP
   mostra 10 e a tabela de ocorrências mostra 9: a tela exibe a divergência, não a esconde.

B. **`Dias perdidos`, `Dias debitados` e `Acidentes graves` continuam vazias nas 75 linhas** —
   IS e MIFR seguem estruturalmente em zero. Nada mudou desde a primeira base.

C. **`HH MÊS` = 1,00 em PCI set/2024, PCI set/2026 e MOA set/2026** (e 2,00 na linha de total
   de set/2026). Marcador de mês não fechado no divisor de todos os índices de frequência.
   O modo Mensal omite o ponto; o Acumulado o absorve sem estrago porque o numerador também
   é pequeno, mas um acidente nesses meses produziria um índice na casa do milhão.

**Alertas**

D. **3 ocorrências sem correspondência em DADOS:** os eventos **111262**, **116866** e
   **118110** têm `Nº do evento` em Ocorrências que não existe na coluna
   `Número do Evento` de DADOS. Os três são Incidentes — é exatamente por isso que
   Ocorrências mostra 41 incidentes enquanto DADOS tem 38 distintos (38 + 3 = 41).

E. **`Impacto` com a grafia `médio` em minúscula em 1 linha de DADOS**, ao lado de `Médio`
   em 14.783. O COUNTIFS não distingue caixa, então a aba Índices não sente; qualquer
   segmentação por essa coluna criaria duas categorias.

F. **`Contrato` com a grafia `moa` em 1 registro**, além de `MOA` em 7.384. Vira um terceiro
   item na lista de contratos e sai do recorte padrão — é a diferença entre o degrau DESVIOS
   (22.521, contado pelo COUNTIFS, que ignora caixa) e as 22.520 linhas que o foco por
   degrau mostra.

G. **`Descrição do Local Exato` continua texto livre:** 15.528 valores distintos em 22.581
   registros; padronizando espaço, ponto final e caixa cairia para 11.082. O botão *agrupar
   variações* do bloco 4 mede o estrago. Segue sendo a melhoria de maior retorno.

H. **`Risco Potencial` com categorias residuais** (vazio, `MEDIO` sem acento,
   `ALTO - Relevante`), como nas bases anteriores.

I. **Sigla unificada em `ACDP`**, igual à planilha. O degrau da pirâmide e o botão de foco da
   página de reunião usam `ACDP` por decisão sua. A réplica do PBIX (`replica.html`) mantém
   `ACPD`, que é o rótulo do relatório original. O carregamento aceita as duas grafias.

### Achados da base anterior (`Dados_MOA.PCI.xlsx`)

**Graves**

0. **`Dias perdidos`, `Dias debitados` e `Acidentes graves` estão vazias nas 50 linhas da
   aba Índices.** Consequência direta: **IS e MIFR são estruturalmente zero** — não por
   ausência de acidente, mas por ausência de dado. Dois dos oito indicadores do relatório
   não medem nada hoje. O IS é justamente o índice de gravidade, que é o que diferencia um
   afastamento de um dia de um de trezentos.
1. **`Mês/ANO` fora do dia 1:** a linha `PCI 27/02/2025` (total dias = 2) quebra o mês de
   fevereiro/2025 em dois itens no slicer — "01/02/2025" traz só o MOA e "27/02/2025" só o
   PCI. Qualquer seleção mensal fica errada nesse mês.
2. **`HH MÊS` = 1,00** em PCI set/24, PCI set/26 e MOA set/26. Como HHT é o divisor de
   todos os índices de frequência, um único acidente nesses meses produziria um índice na
   casa do milhão. Parece marcador de mês ainda não fechado — set/2026 está em andamento.
3. **Índice gravado que não bate com a fórmula do próprio relatório:** `PCI fev/25 IFCDP`
   está 5,65 na planilha, mas ACDP = 1 e HHT = 93.860,21 dão **10,65**. O valor 5,65
   corresponde ao HHT de **agosto/2025** — é uma referência de célula deslocada.
4. **`Descrição do Local Exato` é texto livre:** 15.528 valores distintos em 22.581
   registros. Só padronizando espaços, ponto final e caixa, cairia para 11.082 — 4.446
   categorias a menos. O gráfico de desvios por local, como está, não é utilizável para
   decisão.

**Alertas**

5. `Contrato` com a grafia `moa` em 1 registro, além de `MOA` em 7.384 — vira um terceiro
   item no slicer e some do recorte padrão.
6. `Risco Potencial` com 6 registros vazios, 1 `MEDIO` (sem acento) e 1 `ALTO - Relevante` —
   três fatias residuais na rosca, sem cor definida no tema.
7. **21 `Número do Evento` repetidos** em DADOS: a coluna não serve como chave única.
8. **`NOSSO RECORDE` é cópia literal de `DIAS SEM ACIDENTE`**, não o máximo histórico.
   Quando a contagem zera por acidente, o "recorde" zera junto. O cartão usa `Máx`, então
   só mostra o número certo com todos os meses selecionados; filtrando um mês, ele exibe a
   contagem daquele mês com o rótulo "recorde".

## 6. Sugestões de melhoria, por prioridade

### Qualidade dos dados

**P1 — Remover as 21 linhas duplicadas de DADOS.** São linhas idênticas, não eventos
distintos, e inflam os degraus da pirâmide em 1 ASDP, 3 Incidentes e 17 Desvios. Como fazer:
na coluna `Número do Evento`, `Dados > Remover Duplicados` sobre a chave completa, ou uma
coluna auxiliar `=CONT.SE($B$2:B2;B2)` e filtrar o que vier maior que 1. Impacto: os degraus
passam a ser ACDP 1, ASDP 9, PA 8, Incidente 38, Desvios 22.504, e a tabela de ocorrências
para de divergir do degrau ASDP. Custo: minutos. Esta é a correção de maior valor hoje.

**P1 — Conferir os eventos 111262, 116866 e 118110**, que estão em Ocorrências e não em
DADOS. Ou faltou exportar, ou o número está errado. Custo: minutos.

**P2 — Acrescentar `Fatal` aos valores aceitos em `Classificação`.** Não houve acidente fatal
até aqui, então o zero do topo da pirâmide está correto. A preparação vale mesmo assim: com o
valor previsto na coluna, um evento futuro é contado sem mexer no painel. Custo: minutos.

**P2 — Padronizar as três linhas com `HH MÊS` = 1,00.** Enquanto o mês não fechar, deixar o
HHT vazio evita índice falso; um valor simbólico no divisor, não.

**P2 — Corrigir `médio` para `Médio` na coluna `Impacto` de DADOS** e `moa` para `MOA` na
coluna `Contrato`. Uma validação de dados na planilha resolve os dois de vez.

_As quatro correções pedidas na versão anterior — acumulação por projeto, `Mês/ANO` no dia 1,
`NOSSO RECORDE` como máximo acumulado e a referência deslocada do `IFCDP` — foram feitas e
estão confirmadas na seção 0._

**P1 — Transformar `Descrição do Local Exato` em lista suspensa.** O botão *agrupar
variações* da página Reunião mede o estrago: no recorte PCI+MOA dos últimos 3 meses, o
local mais frequente aparece com **14** ocorrências no ranking cru e com **82** depois de
juntar as grafias ("Predio do gad", "Prédio do gad", "Predio do GAD", "Prédio do GAD"…).
Priorizar pelo ranking cru é priorizar errado por um fator de quase 6.
 Uma coluna auxiliar com
30 a 50 locais padronizados (GAD, Scale Pit, Rua 32, Gasômetro, Galpão 34, Prédio PCI…),
mantendo o texto livre como observação. Impacto: é o que separa o gráfico de local de ser
decorativo ou virar ferramenta de priorização. Custo: alto na primeira carga, baixo depois.

**P2 — Padronizar `Contrato` e `Risco Potencial` na origem** (`MOA`/`moa`, `MÉDIO`/`MEDIO`,
`ALTO - Relevante`). Uma validação de dados na planilha resolve. Custo: baixo.

**P2 — Marcar o mês em andamento.** Enquanto `HH MÊS` estiver com valor simbólico,
sinalizar a linha como provisória (ou deixar HHT vazio, que evita índice falso).

### Clareza dos indicadores

**P1 — Definir os limites.** Preencher as colunas `Referência ...` com a meta corporativa e
plotar a linha de limite; hoje a legenda "Limite" aponta para o nada. Sem meta, nenhum dos
8 gráficos permite dizer se o mês foi bom.

**P1 — Recalcular `NOSSO RECORDE` como máximo histórico acumulado** (na planilha, um
`MÁXIMO` acumulado de `DIAS SEM ACIDENTE` por projeto) e mostrar os dois números lado a
lado: sequência atual e recorde. Hoje o cartão pode exibir "730" e "recorde" para coisas
diferentes conforme o filtro.

**P2 — Corrigir os títulos de eixo e as fórmulas trocadas** listados na seção 4.

**P2 — Nunca somar índices de frequência entre projetos.** Selecionando PCI + MOA, os
gráficos somam taxas — matematicamente sem sentido (em fev/25 a soma dá 10,65, que é a taxa
do PCI, mas com o HHT do MOA ignorado). O certo é recalcular:
`(Σ ocorrências × 1.000.000) / Σ HHT`. Isso exige uma medida DAX no PBIX e mudaria números
publicados, então não fiz nada — precisa da sua aprovação.

**Feito — paleta da rosca de risco.** A escala passou a ser ordenada por severidade
(verde, âmbar, laranja, vermelho). Antes o ALTO era verde e o MÉDIO azulado, o que invertia
a leitura intuitiva. Vale para a página de reunião; a réplica mantém as cores do PBIX.

**P3 — Rever o contraste da página Início** (título verde `#78B92E` sobre azul `#118DFF`).

### Usabilidade

**P2 — Mostrar `Tipo de Evento` na página Proativos**, como slicer ou como coluna na
tabela: hoje 70 registros que não são desvios entram em gráficos intitulados "desvios".

**P3 — Colocar o total de registros do recorte** num cartão da página Proativos (aqui ele
aparece na faixa de filtros ativos).

### Desempenho e manutenção

**P2 — Reduzir a planilha às colunas usadas.** DADOS tem 121 colunas, o relatório usa 8.
O arquivo cai de 10 MB para menos de 1 MB e a carga, de ~8 s para ~1 s.

**P3 — Padronizar `Data do evento` e `Data` como data de verdade** na planilha (hoje são
texto dd/mm/aaaa). Elimina a dependência de detecção automática de tipo e o risco de
inverter dia/mês em máquina com locale diferente.

**P3 — Criar uma tabela de calendário e uma tabela de projetos** no modelo do Power BI.
Resolve de uma vez o relacionamento com `Ocorrências`, a hierarquia de datas e o problema
do `Mês/ANO` fora do dia 1.
