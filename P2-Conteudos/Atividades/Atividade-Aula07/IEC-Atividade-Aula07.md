# IEC – Atividade Prática – Aula 07

---

## Exercício 1 – Teste de Integração Completo

Criado o arquivo `tests/processamento.test.js` com a função `processarAlerta` que combina classificação e envio de notificação.

**`alertas.js`** (módulo compartilhado):
```js
function classificarAlerta(nivel) {
  if (nivel > 80) return "Crítico";
  if (nivel > 50) return "Alto";
  return "Moderado";
}

function enviarNotificacao(nivel) {
  const classificacao = typeof nivel === "string" ? nivel : classificarAlerta(nivel);
  return `Notificação enviada: ${classificacao}`;
}

function processarAlerta(nivel) {
  const classificacao = classificarAlerta(nivel);
  return enviarNotificacao(classificacao);
}

module.exports = { classificarAlerta, enviarNotificacao, processarAlerta };
```

**`tests/processamento.test.js`**:
```js
const { processarAlerta } = require("../alertas");

test("processamento completo de alerta crítico", () => {
  const resultado = processarAlerta(90);
  expect(resultado).toBe("Notificação enviada: Crítico");
});
```

Resultado ao executar `npm test`:
```
PASS tests/processamento.test.js
  ✓ processamento completo de alerta crítico
```

---

## Exercício 2 – Mock de Função Externa

Criado o arquivo `tests/mock.test.js` simulando uma chamada a uma API externa com `jest.fn()`, sem depender de rede ou serviços reais.

**`tests/mock.test.js`**:
```js
const api = { enviar: jest.fn(() => "Simulado!") };

test("simulação de envio", () => {
  const resposta = api.enviar();
  expect(resposta).toBe("Simulado!");
});

test("mock registra chamadas realizadas", () => {
  api.enviar();
  expect(api.enviar).toHaveBeenCalled();
});
```

Resultado ao executar `npm test`:
```
PASS tests/mock.test.js
  ✓ simulação de envio
  ✓ mock registra chamadas realizadas
```

---

## Exercício 3 – Verificação de Segurança

Executado no terminal para auditar as dependências do projeto:

```bash
npm audit
```

Saída esperada (sem vulnerabilidades):
```
found 0 vulnerabilities
```

Caso vulnerabilidades sejam encontradas, corrigir com:

```bash
npm audit fix
```

> O comando analisa o `package-lock.json` e verifica as dependências no banco de dados de vulnerabilidades do npm.

---

## Exercício 4 – Workflow de Segurança

Criado o arquivo `.github/workflows/security.yml` para automatizar a auditoria de dependências no CI.

**`.github/workflows/security.yml`**:
```yaml
name: Auditoria de Segurança

on:
  push:
    branches: [ dev ]
  pull_request:
    branches: [ dev ]
  schedule:
    - cron: '0 8 * * 1'

jobs:
  security:
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

      - name: Rodar auditoria de segurança
        run: npm audit --audit-level=high
```

> O workflow roda a cada push/PR na branch `dev` e toda segunda-feira às 8h (via `schedule`).

---

## Exercício 5 – Commit Final e Pull Request

Passos executados no terminal:

```bash
# 1. Criar a branch
git checkout -b feature/seguranca

# 2. Adicionar os arquivos novos
git add tests/processamento.test.js tests/mock.test.js tests/integracao.test.js
git add alertas.js
git add .github/workflows/security.yml

# 3. Commitar
git commit -m "feat: adiciona testes avançados e workflow de segurança"

# 4. Subir a branch
git push origin feature/seguranca
```

Em seguida, aberto Pull Request da branch `feature/seguranca` para `dev` no GitHub.

Pipeline executado com sucesso: **lint + testes + auditoria**.

---

## Exercício 6 – Teste Unitário Isolado

Adicionado teste unitário ao arquivo `tests/alerta.test.js` para validar a função `classificarAlerta` de forma isolada.

**`tests/alerta.test.js`** (trecho adicionado):
```js
// Ex. 6 Aula 07 - teste unitário isolado
test("alerta alto", () => {
  expect(classificarAlerta(70)).toBe("Alto");
});
```

Resultado ao executar `npm test`:
```
PASS tests/alerta.test.js
  ✓ alerta crítico se nível maior que 80
  ✓ alerta alto se nível entre 51 e 80
  ✓ alerta moderado se nível até 50
  ✓ alerta alto
```

---

## Exercício 7 – Teste de Integração entre Funções

Criado o arquivo `tests/integracao.test.js` validando a combinação de `classificarAlerta` com `enviarNotificacao`.

**`tests/integracao.test.js`**:
```js
const { classificarAlerta, enviarNotificacao } = require("../alertas");

test("classificação + notificação", () => {
  const alerta = classificarAlerta(90);
  const resultado = enviarNotificacao(alerta);
  expect(resultado).toBe("Notificação enviada: Crítico");
});

test("classificação + notificação para nível alto", () => {
  const alerta = classificarAlerta(70);
  const resultado = enviarNotificacao(alerta);
  expect(resultado).toBe("Notificação enviada: Alto");
});

test("classificação + notificação para nível moderado", () => {
  const alerta = classificarAlerta(30);
  const resultado = enviarNotificacao(alerta);
  expect(resultado).toBe("Notificação enviada: Moderado");
});
```

Resultado ao executar `npm test`:
```
PASS tests/integracao.test.js
  ✓ classificação + notificação
  ✓ classificação + notificação para nível alto
  ✓ classificação + notificação para nível moderado
```

---

## Exercício 8 – Simulação de Erro em Teste

No arquivo `tests/alerta.test.js`, o valor esperado foi alterado propositalmente para demonstrar como o CI detecta regressões:

```js
// Alteração proposital para simular falha
expect(classificarAlerta(90)).toBe("Alto"); // errado: deveria ser "Crítico"
```

Saída do `npm test` com o erro:
```
FAIL tests/alerta.test.js
  ● alerta crítico se nível maior que 80
    Expected: "Alto"
    Received: "Crítico"
```

Após observar o erro, o valor foi corrigido:

```js
expect(classificarAlerta(90)).toBe("Crítico"); // correto
```

> Testes que falham são tão importantes quanto os que passam — eles protegem o código de regressões acidentais.

---

## Exercício 9 – Relatório de Cobertura

Executado o comando para gerar o relatório de cobertura de testes:

```bash
npm test -- --coverage
```

Ou via script configurado no `package.json`:

```bash
npm run test:coverage
```

**`package.json`** (scripts):
```json
"scripts": {
  "test": "jest",
  "test:coverage": "jest --coverage"
}
```

Saída esperada:
```
----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
alertas.js|     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|
```

> A cobertura mede quais linhas do código foram exercitadas pelos testes. O objetivo é manter a cobertura alta para garantir qualidade.

---

## Exercício 10 – Commit e PR com Novos Testes

Passos executados no terminal:

```bash
# 1. Criar a branch
git checkout -b feature/testes

# 2. Adicionar os arquivos de teste
git add tests/

# 3. Commitar
git commit -m "test: adiciona testes unitários e de integração da aula 07"

# 4. Subir a branch
git push origin feature/testes
```

Pull Request aberto de `feature/testes` para `dev`.

Pipeline executado no GitHub Actions com sucesso:

```
✓ Checkout do repositório
✓ Configurar Node.js
✓ Instalar dependências
✓ Rodar ESLint
✓ Rodar Jest
✓ Relatório de cobertura
```

> Nenhuma falha encontrada. Merge liberado.
