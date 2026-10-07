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

## Fechamento mensal de SLA

Menu **Controle → Fechamento mensal de SLA**. Retrato fixo por mês (`snapshots/ind_AAAA-MM`): casos com dias acima do SLA da situação, calculado só por data + SLA (sem motivo nem ajustes manuais). O mês corrente acumula (quem estourou o prazo continua listado mesmo se resolvido depois); na virada o retrato é congelado. Meses sem retrato aparecem como "Não salvo" (calculados agora) e só são gravados pelo botão.
