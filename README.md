# Dashboard — Planilha de Execuções

Dashboard executivo do escritório Borges Macedo Advocacia, alimentado em tempo real
pela **"Planilha de Execuções"** do Google Sheets.

## Acesso

A página abre numa tela de login e **nada é carregado antes da senha correta**.

A proteção não é uma verificação em JavaScript — isso seria inútil num repositório
público, já que qualquer pessoa leria a senha no código ou pularia a checagem pelo
devtools. Em vez disso:

- O **ID da planilha não existe em texto puro** no código. Ele fica cifrado com
  **AES-256-GCM** dentro do bloco `COFRE`.
- A chave de decifragem é derivada com **PBKDF2-SHA256, 310.000 iterações**, a partir do
  login + senha digitados.
- O AES-GCM é autenticado: **senha errada faz a decifragem falhar**. Não existe um "sim"
  para forjar — sem a senha, a página simplesmente não sabe qual planilha ler.
- A sessão fica só na aba (`sessionStorage`) e **termina ao fechar a aba**. O botão
  **Sair** encerra na hora.
- A leitura da planilha usa o **export CSV público** do Google Sheets (sem chave de API),
  então a planilha precisa continuar compartilhada como "qualquer pessoa com o link pode
  ver".

O login não fica documentado aqui. Peça a quem administra o dashboard.

### Trocar a senha

Abra o **`gerar-senha.html`** direto do seu computador (não pela internet). Preencha o ID
da planilha e o login/senha desejados — o campo "Chave da API do Google" pode ficar com
qualquer texto (não é usado por este dashboard, que lê só o CSV público). Substitua o
bloco `const COFRE = {…};` do `index.html` pelo texto gerado, depois é só commit e push.

> Nunca faça commit da senha, e não cole o ID da planilha em texto puro em nenhum arquivo
> do repositório.

## Como funciona

- O dashboard lê 8 abas da planilha, uma requisição CSV por aba (por `gid`, não pelo
  nome — assim não quebra se alguma aba for renomeada): **Procedentes, Fase Recursal,
  Aguard. Pgto., ITIV/Rec J., Penhora, Impug. à Exec., Alvarás e Acordos**.
- Os dados são buscados no navegador a cada carregamento, com atualização automática a
  cada **5 minutos** (e também ao voltar para a aba do navegador).
- Não há build nem servidor: é um único arquivo `index.html`.
- **Qualquer alteração feita na planilha aparece no dashboard sem precisar mexer no
  código.**

## O que é exibido

| Seção | Conteúdo |
|---|---|
| Resumo Executivo | Valor total em execução, valor e quantidade por aba, taxa de conversão de alvarás, processos a diligenciar, prazos vencidos e a vencer em 7 dias |
| Alvarás por Mês | Soma do valor dos alvarás por mês previsto de recebimento, separada por situação (Solicitado / Recebido), com tabela de todos os processos |
| Acordos por Mês | Soma do valor dos acordos por mês do prazo fatal de pagamento, com tabela de todos os processos |
| Valores por Aba | Comparativo de valor total entre Aguard. Pgto., ITIV/Rec J., Penhora, Impug. à Exec., Alvarás e Acordos |
| Funil de Execução | Quantidade de processos em acompanhamento em cada fase, da procedência ao acordo/alvará |
| Composição da Carteira | Distribuição dos processos por tipo de objeto de ação e maiores réus (contrapartes) por valor em execução |
| Pontos de Atenção | Alertas gerados automaticamente: prazos vencidos, prazos vencendo em 7 dias, processos a diligenciar, alvarás sem mês de previsão cadastrado, concentração por réu, processos em recuperação judicial |

## Sobre as colunas "Mês Previsto"

- **Alvarás**: usa a coluna **"Mês Previsto Recebimento"**. Aceita data completa
  (`dd/mm/aaaa`), mês/ano (`mm/aaaa`) ou o nome do mês por extenso. Linhas sem nada
  preenchido caem no grupo **"Sem previsão"** — e viram um alerta na seção de Pontos de
  Atenção, já que isso indica dado faltando na planilha, não um erro do dashboard.
- **Acordos**: usa a coluna **"Fatal Pgto."** como data de previsão de pagamento.

## Estrutura do repositório

```
index.html        Dashboard (HTML + CSS + JS, sem build)
gerar-senha.html   Gera o bloco COFRE para trocar login/senha (rodar localmente)
logo.png / favicon.png
.github/workflows/pages.yml   Publica no GitHub Pages a cada push em main
```
