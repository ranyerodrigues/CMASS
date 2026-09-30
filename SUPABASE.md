# Supabase — projeto QMSS MOA/PCI

Banco criado para guardar as três abas da planilha. O painel **continua funcionando sem ele**:
a leitura pelo Excel não mudou. O Supabase é a base para histórico consolidado e para o painel
acessível por link.

| | |
|---|---|
| Projeto | `QMSS MOA/PCI` |
| Referência | `oxloaufdkksweeelhgvq` |
| URL | `https://oxloaufdkksweeelhgvq.supabase.co` |
| Região | us-west-2 |
| Postgres | 17 |

## Tabelas

| Tabela | Aba de origem | Linhas hoje |
|---|---|---|
| `indices` | Índices | 75 (50 de projeto + 25 do bloco de total) |
| `ocorrencias` | Ocorrências | 59 |
| `eventos` | DADOS | **0 — pendente, ver abaixo** |
| `carga` | — | carimbo da última carga |

Os nomes das colunas seguem os campos que o `js/loader.js` já usa, então a leitura pelo
Supabase e a leitura pelo Excel entregam o mesmo objeto. As colunas de índice calculado
(`IFA`, `IFT`, …) **não** foram carregadas de propósito: são derivadas, o painel as recalcula
e guardá-las criaria duas versões da mesma verdade.

`public.eh_total(projeto)` marca as linhas de TOTAL (`PROJETO = 'MOA + PCI'`). Toda consulta
de agregação precisa filtrar `where not public.eh_total(projeto)`, senão conta tudo duas vezes.

`public.resumo()` devolve as contagens das três tabelas e a data da última carga, em uma
chamada só. É o que o carregador usa para conferir, e é por onde o painel pode mostrar
"dados atualizados em".

## Conferência da carga

Os números batem com o painel, um a um:

| Recorte | ACDP | ASDP | PA | Incidente | DESVIOS | HH | Sev4 | Recorde | IFA |
|---|---|---|---|---|---|---|---|---|---|
| MOA + PCI | 1 | 10 | 8 | 41 | 22.521 | 3.817.009,14 | 195 | 730 | 2,88 |
| MOA | 0 | 2 | 0 | 7 | 7.376 | 1.323.225,24 | 36 | 730 | 1,51 |
| PCI | 1 | 8 | 8 | 34 | 15.145 | 2.493.783,90 | 159 | 579 | 3,61 |

Ocorrências: 59 no total — Incidente 41, ASDP 9, PA 8, ACDP 1; 9 com Impacto Sev4.

## Segurança

RLS ligada nas quatro tabelas, com uma política só: **leitura pública, escrita nenhuma**.
A chave publicável (`sb_publishable_…`) pode ir para o painel — ela não escreve. A chave de
serviço escreve e **nunca** pode aparecer em HTML ou JavaScript publicado; ela é usada só no
carregador local. O verificador de segurança do Supabase não aponta nenhum problema.

## Os 22.581 eventos ainda não estão no banco

O container onde este projeto foi montado não alcança o Supabase pela rede, então as tabelas
pequenas foram carregadas por SQL, mas a aba DADOS não tem como passar por esse caminho.

Use **`ferramentas/carregar-supabase.html`**, que faz a carga das três tabelas de uma vez:

1. abra o arquivo com duplo clique (ele é local e não deve ir para servidor nenhum);
2. cole a chave de serviço — Supabase → Project Settings → API → `service_role`;
3. escolha a planilha do mês;
4. clique em **Substituir os dados no Supabase**.

A carga apaga e reescreve cada tabela inteira, então rodar duas vezes não duplica nada. No
fim ele confere a contagem gravada contra a contagem da planilha e mostra as duas lado a
lado. Testado com a base atual: 75 / 59 / 22.581, todas conferindo.

## O que ainda não foi feito

O painel **não lê do Supabase** — continua lendo o Excel no navegador. Ligar uma coisa na
outra é uma decisão com três consequências, e por isso não foi feita por conta própria:

1. a página passaria a precisar de internet, e hoje ela abre offline, que é o que a faz
   funcionar em sala de reunião;
2. a chave publicável ficaria no JavaScript entregue ao navegador — aceitável com a RLS que
   está no lugar, mas é uma decisão consciente, não um detalhe;
3. alguém precisa rodar o carregador todo mês, ou o banco fica velho enquanto a planilha anda.

O caminho que mantém as duas coisas: o painel tenta o Supabase e, se não houver rede ou
configuração, cai para o instantâneo embutido. Fica como próximo passo, a combinar.
