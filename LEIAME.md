# Painel QMSS MOA/PCI — HTML

Alimentado pela planilha **`Dados_MOA.PCI_Atual_Com Formulas.xlsx`** (versão de consulta de
30/09/2026), que é o modelo padrão de preenchimento. A estrutura esperada e o checklist
mensal estão em **`MODELO-DE-PLANILHA.md`**. Os formatos anteriores continuam sendo aceitos.

- **`index.html`** — página única da reunião semanal. É o entregável principal.
- **`replica.html`** — réplica fiel das três telas do `Gestao_Projeto_MOA_PCI.pbix`
  (Início, Reativos, Proativos), guardada só para conferir número a número contra o
  Power BI. Mesmos dados e mesmos cálculos da página única.

## Como abrir

1. Mantenha a pasta inteira junta (o HTML depende de `css/`, `js/`, `vendor/` e `assets/`).
2. Abra `index.html` com duplo clique (Chrome, Edge ou Firefox).

Abre já com os dados embutidos. Não precisa de servidor, internet, Python ou Power BI.

## Como atualizar no mês seguinte

Clique em **Atualizar dados** na barra superior e escolha a versão nova da planilha.
Tudo é recalculado do zero — não há valor digitado no HTML. A planilha é lida dentro do
navegador: nada sai do seu computador e o arquivo original não é alterado.

A planilha precisa manter as três abas (`DADOS`, `Índices`, `Ocorrências`) e os nomes de
coluna do modelo; a aba `Check` é ignorada. Se faltar alguma, a tela de carregamento diz
qual. Maiúsculas, acentos e espaços a mais nos cabeçalhos são tolerados, e os nomes das
versões anteriores (`CPD`/`SPD`/`Incidentes`, `S4`, `Tipo de ocorrência`, `Projeto`)
continuam funcionando.

**Linhas de total.** Se a aba `Índices` trouxer um bloco com `PROJETO = MOA + PCI` — ou
qualquer nome formado por projetos existentes unidos por `+` — o painel reconhece que são
somas, não um projeto novo: elas ficam fora de toda agregação (senão "os dois projetos"
contaria tudo duas vezes), não aparecem no filtro de `Projeto`, e alimentam o modo
**Planilha** quando a seleção cobre exatamente os projetos do total.

## A página

**Um filtro só** — `Projeto` e `Período` — vale para os quatro blocos ao mesmo tempo:
Índices, Ocorrências (por `Projeto` + ano/mês da `Data`) e DADOS (por `Contrato` +
ano/mês da `Data do evento`). Nenhum projeto marcado = todos, como no slicer do Power BI.

**O topo fica fixo.** Barra de estado, capa com os logotipos e a faixa de filtros
acompanham a rolagem — em tela larga, o bloco inteiro; em tela estreita, barra e capa
saem de cena e fica presa só a faixa de filtros, para não comer metade do celular. O
recorte vigente aparece sempre à direita dos filtros, então dá para rolar até o bloco 4
sem perder de vista qual projeto e qual período estão em tela.

Período: os atalhos **Tudo / 12 / 6 / 3 meses / mês mais recente**, mais **Escolher mês ▾**,
que abre um calendário com os três anos. Clique num mês para incluir ou tirar do recorte —
dá para montar qualquer combinação (só fevereiro/2025, ou março + julho, por exemplo).
Meses sem linha na aba Índices aparecem apagados. `limpar meses` volta para todo o
histórico. Escolher mês pela grade também resolve o caso de fevereiro/2025, em que a
planilha tem `01/02` para o MOA e `27/02` para o PCI: o botão **fev** pega os dois.

**Clicar num nível filtra o evento.** Os cinco níveis da pirâmide são botões. Clicando,
a página passa a mostrar só aquele tipo de evento, e a tela rola para as ocorrências.
O que cada um faz:

| Nível | Bloco 3 (Ocorrências) | Bloco 4 (DADOS) | Bloco 2 (gráfico) |
|---|---|---|---|
| FATAIS | Classificação = `Fatal` / `Óbito` | idem | vai para o MIFR |
| ACDP | Classificação = `ACDP` | Classificação = `ACDP` | vai para o IFCDP |
| ASDP | `ASDP` | `ASDP` | vai para o IFSDP |
| PA | `PA` | `PA` | vai para o IFPA |
| INCIDENTES | `Incidente` | `Incidente` | mantém o atual |
| DESVIOS | sem equivalente, não filtra | `Desvios` | mantém o atual |

O bloco 4 filtra pela coluna `Classificação` de DADOS — a mesma que as fórmulas da aba
Índices usam nos COUNTIFS —, por isso os dois lados fecham. Em planilhas antigas, que não
tinham essa coluna, o filtro cai no `Tipo de Evento` e a tela avisa onde não há
equivalente.

Um chip verde mostra o foco ativo com a contagem de cada lado — `cartão 41 · ocorrências 41
· DADOS 44` — para você ver na hora quando um lado diverge do outro, e cada bloco explica
em uma linha o que fez. Clicar no mesmo nível de novo, ou em `limpar foco`, desfaz.
HH TRABALHADAS fica fora da pirâmide e não é clicável: não é um tipo de evento.

**1 · Pirâmide de Bird** — os seis níveis do período: **acidentes fatais** no vértice,
depois ACDP, ASDP, PA, Incidentes e Desvios na base. Abaixo da pirâmide, centralizados, a
sequência sem acidente por projeto (valor do mês mais recente do recorte, com o recorde
histórico na mesma linha) e as HH trabalhadas.

**O nível de acidentes fatais está em zero porque não houve nenhum no período.** Se um vier
a ocorrer, basta `Classificação` em DADOS aceitar também o valor `Fatal` (ou `Óbito`) — hoje
ela assume Desvios, ACDP, ASDP, PA e Incidente — e o painel passa a contá-lo sozinho. O
painel de Qualidade dos dados registra isso como informação.

Cada nível tem um cartão à esquerda, com a sigla, e a fatia da pirâmide à direita, com o
número. As fatias vão do vértice (0%) até a base (100%) em passos
iguais: a proporção real é de 1 para 22.521 e, desenhada em escala, o vértice sumiria.
Somadas, as fatias dão exatamente as 22.581 linhas de DADOS.

**O marcador Sev4** no cartão de cada nível conta quantos daqueles eventos tiveram
`Impacto = Sev4` — potencial de fatalidade. Ele responde ao mesmo recorte de projeto e
período e sai do cruzamento de `Classificação` e `Impacto` na aba DADOS, porque a aba
Índices só traz o Sev4 total do mês, sem quebra por classificação. A soma dos cinco
marcadores fecha com a coluna `Sev4` da aba Índices. Sem essas duas colunas em DADOS
(planilhas antigas), os marcadores não aparecem.

Números grandes aparecem em ordem de grandeza: **22,5 Mil**, **3,8 Mi**. Abaixo de mil, o
número é exibido inteiro. O valor exato fica no tooltip do nível.

**2 · Indicadores** — um índice por vez, escolhido no seletor. O ponto vermelho marca os
que têm algum valor diferente de zero no período; os outros trazem a etiqueta *zerado*.
Abaixo do seletor, uma linha explica em português comum o que o índice mede — por exemplo,
*IFA: quantos acidentes com afastamento (ACDP) mais acidentes sem afastamento (ASDP)
aconteceram a cada 1 milhão de horas trabalhadas*.

**Dois gráficos, um sobre o outro:**

1. **o índice**, acumulado — cada ponto é `(Σ ocorrências até o mês × 1.000.000) ÷ (Σ HHT até
   o mês)`, contado a partir do primeiro mês do recorte. É a curva que mostra o índice
   diluindo conforme as horas trabalhadas se acumulam sem novas ocorrências, e subindo a cada
   ocorrência nova. Recalculado das colunas de contagem e de `HH MÊS` — não é a soma dos
   índices gravados. Como ocorrências e HHT são somados antes da divisão, vale com vários
   projetos selecionados;
2. **os eventos contados nele**, com **uma linha por grupo**. No IFA são duas: ACDP e ASDP,
   cada uma acumulando sozinha. No IFT são três (ACDP, ASDP, PA); no IS, duas (dias perdidos
   e dias debitados); nos índices de um componente só, uma. Cada linha tem o nome escrito no
   último ponto, além da legenda. É um gráfico separado, e não um eixo duplo no primeiro, de
   propósito: taxa e contagem têm escalas diferentes e sobrepor as duas num eixo só esconde
   isso.

A cor de cada linha é a do nível correspondente da pirâmide, para ligar os dois visuais.

A nota abaixo dos gráficos fecha a conta do período e avisa quando o indicador depende de
coluna vazia (é o caso de IS e MIFR).

Os modos **Mensal** e **Planilha** saíram da tela: a reunião lê o acumulado, e os outros dois
eram conferência. Os dois cálculos continuam no modelo e na suíte de testes — o Mensal
(ocorrências do mês ÷ HHT do mês) e a coluna gravada no Excel, que com todo o histórico
coincide com o acumulado: 2,88 para os dois projetos, 3,61 para PCI, 1,51 para MOA.

**3 · Ocorrências do período** — aba Ocorrências, no mesmo recorte, com as colunas
Data, **Impacto**, Classificação, Contrato e Descrição. Os botões de **Impacto**
(Alto, Médio, Sev4, Baixo, Baixo (NL), Ambiental) filtram a tabela; nenhum marcado = todos.
Cabeçalho clicável ordena. Registros classificados como **EXCLUÍDO** e **REJEITADO** ficam
fora por padrão — na base atual não existe nenhum, então o botão de exibi-los nem aparece.

Os quatro blocos não trazem mais texto explicativo abaixo do título: a tela é de reunião e
o que ela mostra está descrito aqui e em `MAPEAMENTO.md`. Os avisos que dependem do estado
— foco ativo, realce cruzado, registros ocultos — continuam aparecendo no lugar deles.

**4 · Indicadores proativos — desvios** — risco potencial, fatores de risco, locais e a
tabela de registros. Clicar numa fatia ou barra aplica o realce aos outros visuais
*deste bloco*; os blocos 1 a 3 não mudam, porque vêm de outra tabela. O botão
**agrupar variações** nos dois rankings vem desligado: o ranking sai cru, como no Power
BI. Ligado, junta valores que só diferem por espaço, pontuação, acento ou maiúsculas —
use para decidir onde atuar, não para reportar número.

**Qualidade dos dados** na barra superior abre os achados de consistência da planilha.

## Identidade visual

O painel segue o esquema **TECHINT-02** do template corporativo (`POR- TEC_Presentación
16-9.pptx`): azul `#007DC3` como cor de destaque, azul-marinho `#002B5C` em títulos e
cartões cheios, cinzas do template nas bordas e nos textos secundários, e as duas faces do
fontScheme — **Arial** em títulos e rótulos, **Calibri** no texto corrido. Não há CDN de
fonte (a página abre offline), então a pilha cai para equivalentes métricos quando a
máquina não tiver as duas instaladas.

Duas exceções deliberadas:

1. **A pirâmide de Bird** usa a paleta do infográfico de referência (âmbar, vermelho,
   turquesa, azul, roxo), reescalonada para passar nas verificações de contraste e de
   daltonismo em tema claro e escuro. Ali a cor só ordena os níveis: a identidade de cada
   um vem do rótulo escrito, não da cor.
2. **A rosca de risco potencial** passou a ser uma escala ordenada de severidade —
   verde, âmbar, laranja, vermelho. No Power BI o ALTO era verde e o MÉDIO azulado, o que
   invertia a leitura (item P3 de `VALIDACAO.md`). O vermelho `#E31B23` do template fica
   reservado a estado (Sev4 e severidade máxima) e nunca é usado como cor de série.

`replica.html` continua com as cores e a tipografia do `.pbix`: ali o critério é fidelidade
ao relatório original, não identidade visual.

## Arquivos

```
index.html            página única da reunião
replica.html          réplica das três telas do Power BI (conferência)
css/style.css         tokens de cor e tipografia + estilos da réplica
css/painel.css        layout em blocos da página única
js/loader.js          camada 1 — leitura do Excel e tipagem
js/model.js           camada 2 — tratamento, qualidade e cálculo
js/charts.js          camada 3 — primitivas de gráfico em SVG
js/painel.js          camada 4 — página única
js/app.js             camada 4 — réplica
js/dados.js           instantâneo dos dados (apague o arquivo e a linha nos dois HTML
                      para as páginas sempre começarem pedindo o Excel)
vendor/xlsx.mini.min.js  leitor de .xlsx (SheetJS, Apache-2.0 — licença em vendor/)
assets/*.png          logotipos e ícones extraídos do próprio .pbix
MODELO-DE-PLANILHA.md estrutura esperada da planilha e checklist mensal
MAPEAMENTO.md         indicador → origem → regra de cálculo
VALIDACAO.md          o que foi conferido, divergências, pendências e melhorias
```

Nenhum dado fica gravado no navegador.
