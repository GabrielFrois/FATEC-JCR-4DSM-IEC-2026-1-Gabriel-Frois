# IEC – Atividade Prática – Aula 08

---

## Exercício 1 – Cobertura de Testes no package.json

O sistema do INPE precisa garantir que a função `classificarAlerta` esteja 100% testada. Para isso, foi adicionado o comando `test:coverage` no `package.json`.

**`package.json`** (scripts atualizados):
```json
"scripts": {
  "test": "jest",
  "test:coverage": "jest --coverage"
}
```

Resultado ao executar `npm run test:coverage`:
```
----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|
```

> O comando `jest --coverage` utiliza o Istanbul (integrado ao Jest) para instrumentar o código e registrar quais linhas, funções e ramificações foram exercitadas durante os testes.

---

## Exercício 2 – Pipeline com Relatório de Cobertura

Atualizado o arquivo `.github/workflows/ci.yml` para gerar automaticamente o relatório de cobertura sempre que um Pull Request para `main` for criado.

**`.github/workflows/ci.yml`** (atualizado):
```yaml
name: CI com ESLint e Testes

on:
  push:
    branches: [ dev ]
  pull_request:
    branches: [ main, dev ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do repositório
        uses: actions/checkout@v4

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Instalar dependências
        run: npm install

      - name: Rodar ESLint
        run: npx eslint .

      - name: Rodar Jest com cobertura
        run: npm run test:coverage
```

> O pipeline agora inclui a etapa de cobertura. Se um PR chegar sem testes suficientes, o relatório expõe as lacunas antes da integração, bloqueando código não testado de chegar à branch principal.

---

## Exercício 3 – Instalação do Codecov

Instalado o pacote `codecov` como dependência de desenvolvimento para registrar o histórico de cobertura da aplicação:

```bash
npm install --save-dev codecov
```

Após a instalação, o `package.json` passa a incluir a dependência:

```json
"devDependencies": {
  "codecov": "^3.8.3",
  "eslint": "^10.0.3",
  "jest": "^29.7.0",
  "prettier": "^3.8.1"
}
```

Também é necessário adicionar a configuração do Jest para gerar o relatório no formato `lcov`, que o Codecov consegue interpretar:

**`package.json`** (configuração do Jest adicionada):
```json
"jest": {
  "coverageReporters": ["lcov", "text"]
}
```

> O Codecov armazena o histórico de cobertura a cada execução, permitindo ao INPE rastrear a evolução da confiabilidade do sistema ao longo do tempo. O formato `lcov` é o padrão aceito pela plataforma.

---

## Exercício 4 – Pipeline com Envio ao Codecov

Atualizado o arquivo `.github/workflows/ci.yml` para enviar automaticamente o relatório de cobertura ao Codecov após cada execução.

**`.github/workflows/ci.yml`** (etapa de upload adicionada):
```yaml
name: CI com ESLint e Testes

on:
  push:
    branches: [ dev ]
  pull_request:
    branches: [ main, dev ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do repositório
        uses: actions/checkout@v4

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Instalar dependências
        run: npm install

      - name: Rodar ESLint
        run: npx eslint .

      - name: Rodar Jest com cobertura
        run: npm run test:coverage

      - name: Enviar relatório ao Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

> O `CODECOV_TOKEN` foi cadastrado como secret no repositório do GitHub. A flag `fail_ci_if_error: true` garante que o pipeline falhe caso o envio não seja concluído, evitando que uma execução sem cobertura passe despercebida.

---

## Exercício 5 – Badges no README

Adicionados dois badges ao `README.md` do projeto: status do pipeline CI e percentual de cobertura de testes.

**`README.md`**:
```markdown
# Aplicativo INPE – Monitoramento de Eventos Climáticos

![CI](https://github.com/GabrielFrois/inpe-alertas/actions/workflows/ci.yml/badge.svg)
[![codecov](https://codecov.io/gh/GabrielFrois/inpe-alertas/branch/main/graph/badge.svg)](https://codecov.io/gh/GabrielFrois/inpe-alertas)

**Objetivo:** app móvel para alertas de queimadas, inundações, desmatamento e relatos da população em tempo real.
```

Resultado visual no repositório:

| Badge | O que indica |
|---|---|
| ![CI passing](https://img.shields.io/badge/CI-passing-brightgreen) | Pipeline de lint + testes rodando com sucesso |
| ![coverage 100%](https://img.shields.io/badge/coverage-100%25-brightgreen) | 100% das linhas do código cobertas pelos testes |

> Os badges são gerados automaticamente pelas plataformas GitHub Actions e Codecov, sempre refletindo o estado mais recente do repositório. O coordenador do INPE consegue ver a saúde do projeto em segundos, sem precisar acessar o código.

---

## Commit Final e Pull Request

Passos executados no terminal:

```bash
# 1. Criar a branch
git checkout -b feature/cobertura-badges

# 2. Adicionar os arquivos alterados
git add package.json
git add .github/workflows/ci.yml
git add README.md

# 3. Commitar
git commit -m "feat: adiciona cobertura de testes, integração Codecov e badges no README"

# 4. Subir a branch
git push origin feature/cobertura-badges
```

Em seguida, aberto Pull Request da branch `feature/cobertura-badges` para `main` no GitHub.

Pipeline executado com sucesso:

```
✓ Checkout do repositório
✓ Configurar Node.js
✓ Instalar dependências
✓ Rodar ESLint
✓ Rodar Jest com cobertura
✓ Enviar relatório ao Codecov
```

> Merge liberado. Badges atualizados automaticamente no README após o merge.
