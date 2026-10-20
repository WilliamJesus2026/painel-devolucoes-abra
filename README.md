# Controle de Devoluções — Abra

Painel que cruza **Protocolo de Coleta** e **Tratativas Transportadora**, expondo gargalos de SLA e o responsável por cada caso.

Este repositório contém só o **código do app** (HTML/JS/CSS). Nenhum dado de cliente (CPF, nome, telefone, endereço) fica versionado aqui — os casos reais existem apenas no Firestore do projeto Firebase, protegido por regra de acesso.

## Stack

- App estático em `public/index.html` (sem build step)
- Firebase Firestore — banco de dados dos casos
- Firebase Storage — histórico de arquivos importados
- Firebase Auth (Google, domínio `@abracadabra.com.br`) — controla quem acessa
- Firebase Hosting — publica o app

## Rodar localmente

```bash
firebase emulators:start
# ou, para só servir o estático sem emuladores:
npx serve public
```

## Deploy

```bash
firebase deploy --project abra-devolucoes
```

## Importar dados

Não há script de importação neste repositório de propósito — os dados reais nunca devem tocar o Git.
Use a aba **"Importar / Base"** dentro do próprio painel (depois de logado com uma conta `@abracadabra.com.br`) para subir os arquivos do Innovaro (Protocolo de Coleta `.csv` e Tratativas Transportadora `.xlsx`). O painel identifica o tipo sozinho e grava direto no Firestore.

## Segurança

- `firestore.rules` e `storage.rules` restringem leitura/escrita a contas Google do domínio `@abracadabra.com.br`.
- O `FIREBASE_CONFIG` embutido no HTML não é segredo — é a configuração pública do SDK cliente; a segurança de fato está nas regras acima.

## Aba Perdas

Menu **Perdas** (Panorama · Perdas Cliente · Perdas Transportadora). Alimentada automaticamente pela importação do arquivo de **Tratativas Transportadora** do Innovaro: casos com tipo `Perdas Cliente`, `Perdas Cliente C/ Reenvio` e `Perdas Transportadora`.

- Motivo classificado por palavras-chave na descrição (ajustável por caso, no detalhe, e gravado como `motivo_manual`).
- Valor registrado = `VALOR_TITULO`; transportadora = `TIPO_SAIDA_NOVA_SAIDA` (fallback `TIPOSAIDA`, `NUMERO`).
- Perdas não saem do controle quando somem de um arquivo novo (são histórico mensal).
- "Gerar relatório" abre a impressão (PDF) só da aba atual; "Copiar resumo" gera texto para e-mail.

## Perdas por canal e valor estimado

- Canal **Marketplace** = usuários `Thairiny` e `João` (coluna USUARIO); **Relacionamento** = todos os demais. Seletor no topo da aba Perdas.
- O dashboard mostra o **Valor total estimado** das perdas (título → nota de saída → unitário × qtd).

## Controle de Situações

Menu **Controle → Controle de Situações**. Cada situação do Innovaro tem uma classe — **Pendente** (conta SLA), **Finalizada** (não gera atraso) ou **Desconsiderar** (fora do controle, ex.: Cancelada 1–4) — e um SLA em dias, editáveis na tabela ou por planilha (`Situação; Classe; SLA (dias)`). As regras ficam no Firestore (`config/sla_situacoes`). Protocolo de Coleta começa com 20 dias.

## Indicadores por transportadora

Menu **Controle → Indicadores por transportadora**. Cada caso (Pendente + Finalizada; Cancelada* e Perdas ficam fora) é uma tarefa criada na data da solicitação. Mostra tarefas no período, taxa de atendimento (Finalizadas ÷ total), SLA médio das finalizadas e em aberto hoje, por transportadora e por mês. Nos casos em aberto o time escolhe o **motivo da pendência** (No prazo / Relacionamento / Transporte / Cliente), gravado em `cases/{id}.motivo_pend` e refletido em tempo real. O SLA médio usa `finalizado_em`, registrado na importação em que o caso passa a Finalizado.

## Padrão visual e importação

Protocolo de Coleta, Tratativas e Controle de Situações usam o mesmo cabeçalho/estatísticas e a mesma barra de filtros das Perdas. Cada aba tem o botão **Importar planilha**; qualquer importação atualiza o painel inteiro. As Perdas mostram também o valor do mês e o acumulado de janeiro até o mês atual. O painel de Tratativas lista só situações Pendentes; Finalizadas e canceladas seguem nos indicadores.

## Canais: Marketplace x Relacionamento

O canal é identificado pelo **usuário responsável** (coluna USUARIO). Por padrão, usuários que começam com `thairiny` ou `joao` são **Marketplace**; todos os outros são **Relacionamento**. A lista é editável em Controle de Situações → Canal → "Definir usuários do Marketplace" (salva em `config/canais`) e vale também para as Perdas.

## SLA de Apontamento

Menu **Controle → SLA de Apontamento**. Quatro tópicos (Recolhimento depois de entregue, Recolhimento SP cancelamento, Tratativas transportadora, Troca SP mesmo produto/rota emergencial), cada um com a tabela de prazos por situação (`SLA_TXT` no código): prazo 0 = visível, mas fora do SLA; dias corridos ou úteis (feriados nacionais 2026 em `SLA_FERIADOS`); base de contagem = criação, programação ou entrada na situação. O tópico de cada caso vem da coluna CLASSE do Innovaro (mapeamento ajustável na própria aba). A data de entrada na situação (`sit_desde`) só é conhecida quando a situação muda entre duas importações; antes disso o atraso é estimado (marcado com `~`).

## Dashboard

Página única (sem abas): faixa de números + gráficos (coleta reversa, atendimento por mês, perdas por mês, SLA de apontamento, tratativas por grupo) + lista "Atenção agora". Tudo clicável para abrir o detalhe. O antigo "Protocolo de Coleta" agora se chama **Coleta Reversa** na interface (a situação do Innovaro continua com o nome original).

## A pagar e Não retornou

Aba própria (menu **Transportadora**) para as situações *Transportadora a pagar* (1 e 2) e *Produto Não Retornou* (1 e 2), sem índice de atraso. Esses casos não aparecem no Controle de Situações, no SLA de Apontamento nem em Tratativas por grupo. Perdas também ficam fora do SLA (têm aba própria). *Conferência Expedição* é etapa da Logística: visível, mas sem SLA.

## Canais e motivo de apontamento

Canais por usuário: Relacionamento (padrão), Marketplace (Thairiny, João) e Logística (Dayane Costa), editáveis em "Definir usuários por canal" (`config/canais`). No SLA de Apontamento cada caso tem o motivo de não ter atualizado a tarefa (`motivo_apont`), com filtro. No Controle de Situações, pendências criadas antes de 01/07/2026 contam como finalizadas.

## Andamento na Coleta Reversa

Em Atrasados, Em risco e Todos há a coluna **Andamento**, ao lado de Situação: texto livre (com sugestões) para registrar o andamento de cada caso pendente. Fica salvo no caso (ndamento) e vai no CSV e no detalhe.

## Andamento, data de atualização e totais

Coluna **Andamento** (texto livre com sugestões) + **Andamento atualizado em** (data em que a equipe atualizou; botão ✓ hoje confirma a revisão sem mudar o texto; verde = hoje, âmbar = até 2 dias, vermelho = mais antigo). Não é data do Innovaro. Vale para Atrasados, Em risco, Todos (Coleta Reversa) e para as abas Transportadora a pagar / Produto não retornou (agora separadas). 'Todos' lista todas as coletas da planilha (inclui no CD e canceladas) e o total aparece no topo.

## Andamento define o prazo; previsão de coleta

O texto do **Andamento** manda no prazo do caso: contendo 'no prazo' o caso sai de Atrasados/Em risco e vai para No prazo (marcado com ✎); 'em risco' e 'atrasado' fazem o contrário; negações ('fora do prazo') são ignoradas. Apagar ou trocar o texto devolve o prazo calculado pela data. A coluna **Previsão de coleta** guarda a data que a transportadora informou (campo previsao_coleta, manual) e mostra se já venceu.

## Andamento move o caso entre as abas

O texto do Andamento define a aba do caso na Coleta Reversa: 'no prazo', 'em risco', 'atrasado', 'coleta realizada' (ou 'aguardando chegada no CD'), 'recebido no CD' e 'cancelada'. Tudo é recalculado na hora, em todo o painel, e marcado com ✎. Trocar ou apagar o texto devolve o caso para o cálculo normal. 'Todos' sempre lista todas as coletas da planilha. Coleta realizada também tem as colunas de andamento/previsão.

## Protocolo Hello e andamento no SLA

Coluna **Protocolo Hello** (nº de coleta, campo editável) nas abas da Coleta Reversa: só clientes Hello têm campo; os demais mostram '—'. A busca também encontra pelo protocolo. Na lista do **SLA de Apontamento** (inclui NF-e Emitida) há Andamento / situação editável com data de atualização e o Protocolo Hello.

## Controle de Situações: coluna F, tarefas em aberto e de/para

- Importar o CSV do Controle de Situações lê a coluna **em andamento** (F) como andamento / motivo da pendência de cada caso (o que for digitado no painel prevalece sobre a planilha).
- Importar a planilha **Tarefas em aberto - recol cliente** (aba Base) enriquece os casos já existentes: último apontamento vira data de finalização (SLA médio) ou data de entrada na situação (SLA de Apontamento), e traz n. coleta / motivo / NFE pendente / unidade quando preenchidos.
- De/para do tipo de saída → transportadora (tabela da aba Resumo) em SAIDA_MAE; 'Não saiu' (acerto de estoque) e tipos sem de/para ficam fora dos indicadores. Perdas entram como finalizadas, como na planilha de controle.

## Planilha base: Controle de Situações

O CSV 'Controle de Situações' é a base do painel. Ao importar, quem não está mais no arquivo sai do painel (concluído automaticamente), mas só dentro do mesmo universo: mesma CLASSE do Innovaro ou, em casos antigos sem classe, o mesmo fluxo do SLA. Tratativas Transportadora não são afetadas. A planilha 'Tarefas em aberto' é só modelo/enriquecimento. A aba mostra qual planilha está valendo.

## Logística no Controle de Situações

Regra automática por situação (LOG_SITS no código): *Recol P/ Acomp Cliente*, *Recol P/Acomp Transportad* e *Recol P/Acomp Transp 2* são da Logística (responsável Dayane.Costa), qualquer que seja o usuário da planilha. Elas saem dos números e da lista principal de casos em aberto e aparecem num bloco próprio embaixo, com andamento editável. Novos filtros em casos em aberto: **Situação do caso** e **Responsável**. Cada andamento registra quem editou (e-mail logado) e a data.

## Motivo da pendência e registro da equipe

No Controle de Situações o motivo da pendência é exatamente o texto da coluna F ('em andamento') da planilha; os filtros, o gráfico e o quadro de pendências usam esses mesmos textos (sem texto: 'Sem informação'). Cada andamento registra quem atualizou (nome, a partir do e-mail logado; a primeira edição pede o nome, salvo em config/usuarios), a data e um histórico das últimas 10 alterações (visível no detalhe do caso). Andamentos que vêm da planilha ficam em nome de quem importou.

## Dados integrados entre as abas

Andamento, data/nome da atualização, previsão de coleta, protocolo Hello e motivo da pendência são um dado só do caso: editar em qualquer aba vale para todas, inclusive para cópias do mesmo caso com a mesma chave (coleta e tratativa). **Andamento** = texto livre (coluna F da planilha); **Motivo da pendência** (última coluna de todas as tabelas de casos) = lista fixa, preenchida automaticamente por andamento e situação, e editável (volta ao automático pela primeira opção). As buscas existem em todas as listas de casos (inclusive Histórico e Atrasos por região) e as abas da Coleta Reversa filtram também por andamento.

## SLA reinicia com a atualização; Logística ampliada

No SLA de Apontamento, atualizar o andamento (ou marcar '✓ hoje') reinicia o prazo do caso: sai de atrasado e só volta se o prazo passar de novo sem nova atualização. A Logística (LOG_SITS) agora cobre Recol P/ Acomp Cliente, Recol P/Acomp Transportad / Transp 2, Recol Acomp C/Reenvio Tra e Cobrança Realizada (1 e 2); o bloco da Logística tem filtro por situação. 'Sem andamento' = caso em aberto sem texto na coluna F.

## Usuários e acessos (administração)

Menu **Administração → Usuários e acessos** (só para administradores; padrão: admin1@abracadabra.com.br, definido no código e nas regras do banco). O administrador cria o login (e-mail @abracadabra.com.br) por uma sessão secundária do Firebase, para não perder a própria sessão, recebe uma senha provisória (mostrada uma vez) e pode enviar e-mail para a pessoa criar a própria senha. Por usuário: papel (usuário/administrador), abas visíveis, pode editar, pode importar, ativo. Perfis rápidos: acesso total, Logística, Perdas/Financeiro e somente consulta. Contas antigas entram na lista no primeiro acesso com acesso total. Os perfis ficam em usuarios/{uid}.

Limites: a restrição por aba, editar e importar vale na tela; o banco bloqueia a LEITURA de contas desativadas e só administradores mexem em usuarios. Isolar dados por área (para que ninguém consiga ler outra aba) exigiria separar os dados em coleções por área. Usuários não são excluídos do Firebase Auth pelo painel (só desativados).

## Filtros, detalhamento e desempenho

- Dashboard filtrável por ano, mês, transportadora e canal: números, gráficos e a lista 'Atenção agora' seguem o recorte. SLA de Apontamento também filtra por transportadora e canal. Janeiro a junho aparecem mês a mês no Controle (continuam contados como finalizados).
- Clicar nos quadradinhos dos indicadores (Dashboard, Controle, SLA, A pagar/Não retornou, Perdas) abre a lista dos casos por trás do número, com busca, ordenação, CSV e clique para abrir o caso; os cartões de transportadora e canal têm 'ver casos'.
- Desempenho: só a aba aberta é desenhada; o cálculo é reaproveitado até os dados mudarem (DATA_VER); buscas com atraso de 180 ms; tabelas limitadas a 400 linhas.
- SLA médio precisa da data de finalização: importe uma vez a planilha 'Tarefas em aberto' (xlsx).
- A cada publicação trocar const BUILD em public/index.html (abas abertas avisam 'Atualizar agora').

## Cruzamento Controle x Tarefas, Histórico e Média

- Importar o CSV do Controle de Situações (coluna **Andamento** ou 'em andamento') atualiza o andamento dos casos quando a PLANILHA muda de um dia para o outro; o que a equipe digitou no painel fica protegido até a planilha trazer um valor novo. O andamento continua editável.
- Importar a planilha de tarefas (**Tarefa….csv** com Último Apontamento / Criada / média, ou o xlsx 'Tarefas em aberto') cruza por chave: grava ultimo_apontamento, media_dias, finalizado_em (encerrados) e sit_desde (em aberto). A coluna **Média (dias)** aparece no Controle e a aba **Média e cruzamento** detalha por situação e transportadora.
- **Histórico** = banco de casos encerrados que nunca é apagado: Finalizado, Cancelado, Saiu da planilha e Perda (as perdas também ficam na aba Perdas). Busca por cliente/NF/CPF/situação/andamento e filtros por estado, ano e transportadora.
- Casos que saem da planilha sem finalizar deixam o Controle e vão para o Histórico.

## Status do caso editável e SLA por mês (14/10/2026)
- Controle de Situações, bloco Logística e Histórico: coluna **Status do caso** (Automático / Em aberto / Finalizado / Cancelado) em todas as linhas, inclusive as já atendidas. Campo `status_manual` no caso; vale também na Coleta Reversa, nos Perdas e no Histórico (Em aberto reabre o caso). Andamento e Motivo da pendência agora também são editáveis nos casos atendidos.
- Nova tabela **SLA médio por mês de criação** (abaixo do gráfico mensal): criadas, finalizadas, % atendimento, SLA médio das finalizadas, média de dias de todas as tarefas e dos casos em aberto. Assim outubro aparece mesmo com poucas finalizadas.

## Melhorias de design do Controle (14/10/2026)
- **Faixa de alertas** no topo do Controle: abertos há 60+ dias, 30 a 59 dias, sem andamento (clicáveis, filtram a lista) e a média de dias do mês atual contra o anterior.
- **Dias coloridos** na lista (verde < 15, âmbar 15–29, vermelho 30+ para casos em aberto) e coluna **Δ vs mês anterior** na tabela de SLA por mês (▲ pior, ▼ melhor).
- **Colunas** ▾: esconder/mostrar colunas da lista (preferência guardada no navegador); no celular as menos importantes já saem sozinhas.
- **Visões**: filtros prontos e **💾 Salvar visão** com nome (guardadas no navegador de quem salvou).
- **Linha do tempo** no detalhe do caso: andamentos, mudanças de status, motivo e “✓ hoje”, com quem e quando (campo `alt_hist`, últimos 15).

## SLA Transportadora (14/10/2026)
- **Nova aba Coleta Reversa → SLA Transportadora.** Prazo de 20 dias corridos da programação do protocolo até a chegada no CD (romaneio de entrada). Mostra média/mediana, % no prazo, a caminho, atrasados, ranking por transportadora, chegadas por mês, funil (criação → protocolo → coleta no cliente → CD) e lista de casos com farol.
- **Previsão no CD sempre preenchida:** a informada vale; sem ela o painel estima = programação + mediana da transportadora nos últimos 90 dias (selo “estimada”).
- **Datas gravadas na importação:** `recebido_cd_em` (primeira importação com romaneio), `recolhido_em` (virou “Prod Recolhido no Cliente”), `sit_desde` na troca de aba; também editáveis na tabela. A primeira importação depois desta versão é a linha de base; antes disso a chegada aparece como “aprox.”.
- **Controle de Situações:** colunas “Previsão no CD” e “Além da previsão”, alerta “Coleta além da previsão no CD”, linha de SLA de coleta nos cards por transportadora e item no detalhe do caso.

## Atraso por etapa no SLA Transportadora (15/10/2026)
- **Etapa 1: criação da tarefa → protocolo de coleta.** Prazo máximo de 3 dias (`CONFIG.PRAZO_PROTOCOLO_DIAS`, definido pela equipe em 16/10/2026). **Etapa 2: protocolo → chegada no CD**, prazo de 20 dias (casos a caminho contam os dias já passados).
- Aba SLA Transportadora: bloco "Atraso por etapa" (% fora do prazo e atraso médio de cada etapa, a caminho já atrasados), filtro "Atraso por etapa", colunas "Criação → Protocolo" e "Atraso até o CD" na lista, 3 colunas novas no ranking e no CSV.
- Controle de Situações: coluna "Atraso SLA coleta" (protocolo +N d / CD +N d), linha de "protocolo fora do prazo" nos cards por transportadora e as duas etapas no detalhe do caso.

## SLA da coleta no topo do Controle de Situações (16/10/2026)
- Faixa "SLA da coleta — transportadora" logo abaixo do cabeçalho de indicadores: % fora do prazo da etapa 1 (criação → protocolo, máx. 3 dias) e da etapa 2 (protocolo → CD, 20 dias corridos), com atraso médio e quantos seguem em aberto, mais as duas médias. Os dois primeiros cartões filtram a lista (alertas `atrp` e `atrcd`) e há atalhos nas visões prontas. Acompanha os filtros do Controle (ano, mês, transportadora, canal...).

## Importação: Andamento da planilha com aviso e escolha (20/10/2026)
- A importação agora mostra no final quantos Andamentos foram atualizados pela planilha, quantos ficaram como no painel e quantos vieram sem texto; avisa se a coluna "Andamento" não for encontrada (aceita variações do nome).
- Quando a planilha traz um Andamento diferente do painel mas o painel guarda o que a equipe digitou (planilha igual à última importação, ou primeira vez que a coluna chega), o painel pergunta se deve **atualizar com a planilha** (com a data de hoje) ou manter o painel, em vez de decidir sozinho.

## Perdas finalizadas no Histórico (20/10/2026)
- Perdas contam como **Finalizado** no Histórico (o cartão "Perdas" continua, filtrando as que também estão na aba Perdas). A aba Perdas segue alimentada por todas as linhas de Perdas da planilha. A mudança de "Coleta realizada" feita no mesmo dia foi desfeita a pedido (as abas de Coleta Reversa voltaram ao comportamento anterior).

## Importação: terceiro formato da planilha de tarefas (20/10/2026)
- O arquivo "Tarefa (data_hora).xlsx" com colunas **Chave | Situação | Entidade | Último Apontamento | Programação | Data | Hora** passa a ser aceito. Cruza por chave com os casos do painel, grava o último apontamento (e a data desde que o caso está na situação) e calcula a média de dias = Data (criação) → Último Apontamento, já que esse formato não traz a coluna de média. Antes o painel respondia "faltam CHAVE e/ou SITUACAO" porque só reconhecia o Controle de Situações e os dois formatos antigos de tarefas.
