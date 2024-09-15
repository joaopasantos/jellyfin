# Sistematização: Configuração de Pipeline CI/CD para o Projeto Jellyfin

Este fork do projeto Jellyfin foi criado para servir como o projeto de código aberto que será utilizado para o objetivo de configurar um pipeline de Integração e Entrega Contínuas (CI/CD).

## Pipeline

A pipeline construída está contida no arquivo `jellyfin-ci.yml` da pasta `.github/workflows/`.

Abaixo, cada etapa será apresentada de forma detalhada.

### `test`

A primeira etapa configurada na pipeline é a etapa `test`, ela tem um nome auto-explicativo. Nela, a automação irá executar os testes do projeto utilizando a CLI DotNet e gerar o relatório de cobertura de testes utilizando a ferramenta ReportGenerator.

Esta etapa é composta pelos seguintes passos:

- `actions/checkout@v4`: Esta é uma ação pré-existente do repositório de ações do GitHub. Essa ação realiza o check out do repositório de código para que o workflow consiga acessá-lo.
- `actions/setup-dotnet@v4`: Esta é uma ação pré-existente do repositório de ações do GitHub. Essa ação automatiza a configuração do ambiente .NET que será utilizado no workflow. A ação aceita múltiplos parâmetros que personalizam o ambiente a ser configurado, mas no caso da pipeline configurada definimos apenas a versão do .NET a ser utilizada. 
- `dotnet test`: Este comando realiza o build da solução e executa os testes unitários da aplicação. Ao fim de sua execução um arquivo `coverage.cobertura.xml` é gerado, contendo os dados sobre a cobertura de testes.
- `danielpalme/ReportGenerator-GitHub-Action@v5`: Esta é uma ação pré-existente que engloba o uso da ferramenta ReportGenerator. Ao ser executada, ela utiliza o arquivo gerado no passo anterior para gerar um relatório de cobertura de testes nos formatos configurados (no nosso caso, um documento no formato Markdown e compatível com o GitHub).
- `actions/upload-artifact@v4`: Esta é uma ação pré-existente do repositório de ações do GitHub. Ela comprime e anexa o diretório especificado à execução do workflow. 
- `cat coveragereport/SummaryGithub.md >> $GITHUB_STEP_SUMMARY`: Nesse passo, utilizamos a variável `$GITHUB_STEP_SUMMARY` para publicar o relatório de cobertura de teste na página de execução do workflow.

### `analyze`

A etapa `analyze` depende da etapa `test` para iniciar sua execução. Nela, a automação irá executar o processo de análise de qualidade de código usando a ferramenta CodeQL para cada uma das linguagens configuradas na matriz. O resultado da análise pode ser obervado na aba `Security`, opção `Code scanning`, onde todos os alertas gerados na análise estarão listados.

Esta etapa é composta pelos seguintes passos:

- `actions/checkout@v4`
- `actions/setup-dotnet@v4`
- `github/codeql-action/init@v3`: Esta é uma ação pré-existente do repositório de ações do CodeQL. Ao ser executada, ela inicializa as ferramentas CodeQL e cria uma codebase para análise.
- `github/codeql-action/autobuild@v3`: Esta é uma ação pré-existente do repositório de ações do CodeQL. Ao ser executada, ela executa a ferramenta de autobuild do CodeQL e prepara o artefato para a análise de código.
- `github/codeql-action/analyze@v3`: Esta é uma ação pré-existente do repositório de ações do CodeQL. Ela executa a análise de código utilizando a ferramenta CodeQL. O resultado da análise é disponibilizado no repositório do GitHub.

### `build`

A etapa `build` é executada em paralelo à etapa `test` quando a pipeline é acionada. Nela, a automação irá executar o processo de build do DotNet para compilar o projeto e suas dependências.

Esta etapa é composta pelos seguintes passos:

- `actions/checkout@v4`
- `actions/setup-dotnet@v4`
- `dotnet --list-sdks`: Neste passo já temos o repositório disponível para ser utilizado no workflow e o ambiente .NET configurado. Ao ser executado, apresenta na saída quais versões de SDK .NET estão disponíveis para uso. O objetivo dela é ser informativa, apresentando um pouco do ambiente configurado no passo anterior.
- `dotnet restore`: Este passo executa a ação de restore do .NET CLI, identificando e baixando, caso necessário, as dependências do projeto.
- `dotnet build Jellyfin.Server --no-restore --configuration Release`: Este passo executa a ação de build do .NET CLI que compila o projeto e suas dependências em um conjunto de binários. As flags passadas configuram o processo para que ele não realize a ação de `restore`, já executada no passo anterior, e para que realize o build utilizando as configurações da categoria *Release*.
- `dotnet publish Jellyfin.Server --configuration Release --output ./deploy-artifact`: Este passo executa a ação de publicar do .NET CLI que empacota o projeto e  gera a aplicação em seu formato pronto para deploy.
- `actions/upload-artifact@v4`

## Melhorias

Algumas melhorias que eu vejo para a pipeline construída são as seguintes:

### Refinar Análise de Cobertura de Testes

No estado atual a etapa de testes apenas gera um relatório contendo as informações geradas pela análise de cobertura de testes, uma possível evolução seria utilizar essas informações ativamente na pipeline. Um cenário que vejo seria a definição de regras estabelecendo um mínimo de cobertura de testes para que aquele snapshot esteja apto para prosseguir na integração contínua.

### Refinar Análise de Qualidade

Na definição atual da etapa de análise de qualidade, os alertas apenas são listados na página dedicada à análise de código. Uma possível melhoria para essa etapa seria a utilização ativa das informações geradas, definindo quality gates para julgar se um respectivo snapshot do projeto está apto a prosseguir na integração contínua ou não.

### Refinar processo de Build

A etapa de build atual apenas anexa o artefato gerado ao workflow acionado. Uma melhoria para essa etapa seria a automação do versionamento e criação de releases, de modo que uma versão do projeto fosse automaticamente publicada para release se passasse com sucesso em todas as etapas estabelecidas da pipeline.

## Reflexão Pessoal

Realizar esse projeto foi muito interessante. Eu já tinha um pouco de experiência com a utilização de workflows no GitHub porque a sistematização da disciplina "Gerência de Configuração" abordou um cenário similar a esse. Mas nesse caso, o fato do projeto escolhido utilizar a linguagem DotNet, com a qual eu não tinha experiência alguma antes, e de um dos passos ser a utilização de uma ferramenta de análise de código tornou ele um verdadeiro desafio.

Foi muito benéfico ter essa oportunidade de obter novos conhecimentos na utilização da DotNet e da ferramenta CodeQL, que eu sequer conhecia antes de iniciar as aulas da disciplina, além de poder me aprofundar ainda mais na utilização dos workflows do GitHub.