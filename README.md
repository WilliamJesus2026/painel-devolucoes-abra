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
