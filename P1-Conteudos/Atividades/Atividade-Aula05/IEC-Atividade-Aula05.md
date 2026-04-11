# IEC - Atividade Aula 06

## Exercício 1
```json
{
  "scripts": {
    "test": "jest"
  },
  "devDependencies": {
    "eslint": "^10.0.3",
    "jest": "^29.7.0",
    "prettier": "^3.8.1"
  }
}
```

## Exercício 2
`tests/soma.test.js`
```js
function soma(a, b) {
  return a + b;
}

test("soma de 2 + 3 deve ser 5", () => {
  expect(soma(2, 3)).toBe(5);
});
```

## Exercício 3
`tests/alerta.test.js`
```js
function classificarAlerta(nivel) {
  if (nivel > 80) return "Crítico";
  if (nivel > 50) return "Alto";
  return "Moderado";
}

test("alerta crítico se nível maior que 80", () => {
  expect(classificarAlerta(90)).toBe("Crítico");
});

test("alerta alto se nível entre 51 e 80", () => {
  expect(classificarAlerta(70)).toBe("Alto");
});

test("alerta moderado se nível até 50", () => {
  expect(classificarAlerta(30)).toBe("Moderado");
});
```

## Exercício 4
`.github/workflows/ci.yml`
```
name: CI com ESLint e Testes

on:
  push:
    branches: [ dev ]
  pull_request:
    branches: [ dev ]

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

      - name: Rodar Jest
        run: npm test
```

## Exercício 5
`tests/alerta.test.js`
```js
// Alterar:
expect(classificarAlerta(90)).toBe("Crítico");

// Para:
expect(classificarAlerta(90)).toBe("Alto");
```
