# Sistematização: Configuração de Pipeline CI/CD para o Projeto Jellyfin

Este fork do projeto Jellyfin foi criado para servir como o projeto de código aberto que será utilizado para o objetivo de configurar um pipeline de Integração e Entrega Contínuas (CI/CD).

## Pipeline

A pipeline construída está contida no arquivo `jellyfin-ci.yml` da pasta `.github/workflows/`.

Abaixo, cada etapa será apresentada de forma detalhada.

### `build`

A primeira etapa configurada na pipeline é a etapa `build`, ela tem um nome auto-explicativo. Nela, a automação irá executar o processo de build do .NET para compilar o projeto e suas dependências em um conjunto de binários.

Esta etapa é composta pelos seguintes passos:

- `actions/checkout@v4`: Esta é uma ação pré-existente do repositório de ações do GitHub. Essa ação realiza o check out do repositório de código para que o workflow consiga acessá-lo.
- `actions/setup-dotnet@v4`: Esta é uma ação pré-existente do repositório de ações do GitHub. Essa ação automatiza a configuração do ambiente .NET que será utilizado no workflow. A ação aceita múltiplos parâmetros que personalizam o ambiente a ser configurado, mas no caso da pipeline configurada definimos apenas a versão do .NET a ser utilizada. 
- `dotnet --list-sdks`: Neste passo já temos o repositório disponível para ser utilizado no workflow e o ambiente .NET configurado. Ao ser executado, apresenta na saída quais versões de SDK .NET estão disponíveis para uso. O objetivo dela é ser informativa, apresentando um pouco do ambiente configurado no passo anterior.
- `dotnet restore`: Este passo executa a ação de restore do .NET CLI, identificando e baixando, caso necessário, as dependências do projeto.
- `dotnet build --no-restore --configuration Release`: Este passo executa a ação de build do .NET CLI que compila o projeto e suas dependências em um conjunto de binários. As flags passadas configuram o processo para que ele não realize a ação de `restore`, já executada no passo anterior, e para que realize o build utilizando as configurações da categoria *Release*.

### `analyze`

A etapa seguinte configurada na pipeline é a etapa `analyze`. Nela, a automação irá executar o processo de análise de qualidade de código usando a ferramenta CodeQL para cada uma das linguagens configuradas na matriz. O resultado da análise pode ser obervado na aba `Security`, opção `Code scanning`, onde todos os alertas gerados na análise estarão listados.

Esta etapa é composta pelos seguintes passos:

- `github/codeql-action/init@v3`
- `github/codeql-action/autobuild@v3`
- `github/codeql-action/analyze@v3`

## Melhorias

## Reflexão Pessoal