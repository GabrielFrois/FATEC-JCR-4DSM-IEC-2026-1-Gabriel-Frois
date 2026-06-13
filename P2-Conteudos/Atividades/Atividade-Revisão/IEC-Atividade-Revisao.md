# Integração e Entrega Contínua – DSM
## Atividade Prática – Revisão para a Prova 2

---

## PARTE I 

**Questão 1** → **c) Uma parcela significativa do código pode não estar sendo validada pelos testes.**

---

**Questão 2** → **b) detectar problemas antes da implantação.**

---

**Questão 3** → **c) identificar vulnerabilidades em dependências.**

---

**Questão 4** → **b) a cobertura dos testes precisa ser ampliada.**

---

**Questão 5** → **c) Fornecem indicadores visuais sobre a saúde do projeto.**

---

## PARTE II 

**Questão 6**

**Teste automatizado** é um código que verifica automaticamente o comportamento de outro código, sem intervenção manual. No projeto do INPE, um teste automatizado poderia verificar se a função de envio de alerta de queimada dispara corretamente quando os dados de satélite atingem determinado limiar.

**Cobertura de testes** é a métrica que indica qual percentual do código-fonte é exercido (executado) durante a execução dos testes. No INPE, se o módulo de detecção de enchentes possui 200 linhas mas os testes só passam por 80 delas, a cobertura desse módulo é de 40%.

**Vulnerabilidade de software** é uma falha ou fraqueza em uma dependência (ou no próprio código) que pode ser explorada por agentes maliciosos. No projeto do INPE, uma biblioteca de parsing de dados geoespaciais desatualizada poderia conter uma vulnerabilidade que permita injeção de código, comprometendo os alertas enviados à população.

---

**Questão 7**

Mesmo que os 300 testes tenham passado com sucesso, uma cobertura de 40% significa que **60% do código nunca foi executado pelos testes**. Esse código sem cobertura pode conter erros, lógicas incorretas ou falhas de segurança que os testes simplesmente não alcançam.

No contexto do INPE, módulos críticos como o envio de alertas de enchentes ou o processamento de dados de satélite podem estar completamente fora dessa cobertura de 40%, indo para produção sem qualquer validação automatizada. Uma falha nesses módulos poderia resultar em alertas não enviados ou alertas falsos para a população, o que tem consequências diretas na segurança pública.

---

**Questão 8**

O relatório mostra que Controllers (95%) e Services (90%) estão bem cobertos, mas o módulo **Utils está com apenas 18%**, o que é crítico.

**Ações recomendadas:**

1. **Priorizar a escrita de testes para o módulo Utils** – funções utilitárias são frequentemente chamadas por múltiplos módulos. Uma falha nelas pode causar efeitos em cascata em todo o sistema.
2. **Investigar o que o módulo Utils contém** – se inclui funções de formatação de coordenadas geográficas, cálculo de severidade de eventos ou manipulação de datas de alertas, a baixa cobertura é especialmente arriscada.
3. **Definir um threshold mínimo de cobertura** no pipeline (ex.: 80%) para que o build falhe automaticamente se a cobertura cair abaixo do mínimo aceitável.

---

**Questão 9**

Alta cobertura indica que o código é *executado* pelos testes, mas não garante que os testes verificam os **cenários corretos** ou os **comportamentos esperados**.

Exemplos de situações onde alta cobertura não previne defeitos em produção:

- **Testes sem asserções significativas**: o código é percorrido mas nenhum valor de retorno é verificado adequadamente.
- **Casos de borda não testados**: o código cobre 95% dos caminhos felizes, mas não testa entradas inválidas, valores nulos ou condições de race condition.
- **Falhas de integração**: os testes unitários cobrem cada componente isoladamente, mas a integração entre eles pode falhar.
- **Problemas de ambiente**: o comportamento em produção difere do ambiente de testes (configurações, banco de dados, latência de rede).

No INPE, mesmo com 95% de cobertura nos testes unitários, uma falha na integração com a API de satélites pode impedir o recebimento de dados reais sem que nenhum teste unitário detecte isso.

---

**Questão 10**

**O que ele faz:** `npm test` executa a suíte de testes definida no `package.json` do projeto (geralmente Jest, Mocha ou similar). Ele roda todos os arquivos de teste, reporta quais passaram, quais falharam e, dependendo da configuração, gera o relatório de cobertura.

**Quando é executado:** É executado automaticamente pelo pipeline de CI/CD a cada push ou pull request para o repositório. No GitHub Actions, aparece como um step dentro do workflow YAML, por exemplo após a instalação das dependências (`npm install`).

**Importância no projeto do INPE:** Garante que nenhuma alteração no código introduza regressões nos alertas de queimadas, enchentes ou outros eventos climáticos. Se um desenvolvedor alterar a lógica de cálculo de risco e o `npm test` falhar, o pipeline bloqueia o merge automaticamente, protegendo o sistema antes que a falha chegue à população.

---

**Questão 11**

**Não concordo com a decisão.**

Vulnerabilidades classificadas como **High** pelo `npm audit` representam riscos reais e documentados de exploração. Continuar o deploy sem corrigir ou mitigar essas vulnerabilidades significa colocar em produção um sistema com falhas de segurança conhecidas.

No contexto do INPE, o aplicativo processa dados sensíveis de monitoramento ambiental e envia alertas à população. Uma vulnerabilidade High poderia permitir:

- Manipulação de alertas (envio de falsos alertas ou supressão de alertas reais).
- Acesso não autorizado ao banco de dados de eventos climáticos.
- Comprometimento da integridade dos dados de queimadas e enchentes.

A decisão correta seria: **atualizar a dependência vulnerável** (usando `npm audit fix`) ou, se não houver correção disponível, avaliar se o risco é aceitável com controles compensatórios documentados — nunca simplesmente ignorar.

---

**Questão 12**

**Vulnerabilidade em dependências de software** ocorre quando uma biblioteca de terceiros utilizada pelo projeto possui uma falha de segurança conhecida (registrada em bancos como o CVE — Common Vulnerabilities and Exposures). O problema é que o código do projeto em si pode estar correto, mas ele depende de um componente externo com brecha exploitável.

**Exemplo no INPE:** Suponha que o aplicativo utilize a biblioteca `axios` (cliente HTTP) em uma versão antiga que possui uma vulnerabilidade de SSRF (Server-Side Request Forgery). Um atacante poderia forjar requisições a partir do servidor do INPE, potencialmente acessando recursos internos da rede ou manipulando as chamadas à API de dados de satélite. O `npm audit` identificaria essa vulnerabilidade e indicaria a versão corrigida para atualização.

---

**Questão 13**

**Não, 97% de cobertura não garante que o software está livre de erros.**

Cobertura mede **quais linhas/branches do código são executadas** durante os testes, não **se o comportamento do sistema está correto** em todos os cenários possíveis.

Um software com 97% de cobertura ainda pode ter defeitos por:

- **Testes mal escritos** que executam o código mas não verificam os resultados corretamente.
- **Lógica de negócio incorreta** que é testada e "aprovada" com dados inadequados.
- **Falhas de integração** entre componentes ou com sistemas externos.
- **Problemas de concorrência** que só se manifestam sob carga real.
- **Vulnerabilidades de segurança** que os testes funcionais não cobrem.

A cobertura é um indicador importante, mas deve ser analisada em conjunto com a qualidade dos testes, análise de vulnerabilidades e testes de integração.

---

**Questão 14**

Os relatórios de cobertura auxiliam o gerente na tomada de decisão sobre deploy de diversas formas:

- **Identificação de áreas de risco**: módulos com baixa cobertura (como um Utils com 18%) indicam partes do sistema que chegam à produção sem validação adequada, aumentando o risco de falhas.
- **Tendência histórica**: acompanhar a evolução da cobertura ao longo dos sprints indica se a qualidade está melhorando ou degradando.
- **Critério objetivo de go/no-go**: definir um threshold mínimo (ex.: 80%) torna a decisão de deploy menos subjetiva. Se a cobertura estiver abaixo do mínimo, o deploy é bloqueado automaticamente.
- **Priorização de esforços**: o relatório detalhado por módulo permite direcionar a equipe para escrever testes nas áreas mais críticas antes do próximo release.

No INPE, o gerente pode usar o relatório para garantir que os módulos de envio de alertas — os mais críticos para a missão — tenham cobertura adequada antes de cada implantação.

---

**Questão 15**

**Recomendação: não realizar o deploy até que as vulnerabilidades High sejam corrigidas.**

Análise do cenário:

- **Tests: 100% Passed** → ótimo, toda a suíte de testes está verde.
- **Coverage: 86%** → cobertura aceitável, indica boa maturidade de testes.
- **Vulnerabilities: 5 High** → **bloqueador crítico**.

A presença de 5 vulnerabilidades de severidade High significa que existem 5 brechas de segurança conhecidas e documentadas no sistema. Mesmo com testes e cobertura excelentes, o deploy colocaria em produção um sistema sabidamente vulnerável.

**Ações recomendadas:**
1. Executar `npm audit fix` para resolver as vulnerabilidades automaticamente quando possível.
2. Para as que não possuem correção automática, avaliar atualização manual da dependência ou substituição por alternativa segura.
3. Somente após zerar as vulnerabilidades High (e idealmente as Critical), aprovar o deploy.

---

**Questão 16**

Testes automatizados e Integração Contínua (CI) são complementares e praticamente inseparáveis.

A **Integração Contínua** é a prática de integrar as alterações de código ao repositório principal com frequência (várias vezes ao dia), acionando um pipeline automatizado a cada integração. Os **testes automatizados** são o mecanismo central que valida cada integração.

Sem testes automatizados, a CI seria apenas um sistema de build — saberia que o código compilou, mas não que ele funciona corretamente. Os testes automatizados são o que transforma a CI em uma rede de segurança real: cada push aciona os testes, e qualquer regressão é detectada imediatamente, antes de chegar à branch principal.

No INPE, a CI garante que todo commit que altere a lógica de alertas seja automaticamente testado, impedindo que código com falhas seja integrado ao sistema principal que monitora eventos climáticos em tempo real.

---

**Questão 17**

A funcionalidade de recebimento de denúncias de queimadas é **crítica para a segurança pública**, o que exige uma estratégia rigorosa de testes por diversos motivos:

1. **Alto impacto de falhas**: uma denúncia não processada ou processada incorretamente pode atrasar a resposta a um incêndio real, colocando vidas em risco.
2. **Múltiplos caminhos de entrada**: denúncias podem vir de diferentes canais (app mobile, SMS, API) com formatos variados, exigindo testes para cada variação de entrada.
3. **Dados inválidos e ataques**: usuários podem enviar dados malformados ou maliciosos. Testes de validação de entrada garantem que o sistema não quebre nem seja explorado.
4. **Integração com sistemas externos**: a denúncia precisa ser processada e repassada aos órgãos competentes. Testes de integração garantem que essa cadeia funcione fim a fim.
5. **Disponibilidade**: durante picos (época de seca), o volume de denúncias aumenta muito. Testes de carga garantem que o sistema não falhe exatamente quando mais é necessário.

---

**Questão 18**

- **Build**: indica se o processo de compilação/build do projeto foi concluído com sucesso. Se estiver vermelho (failing), o código atual não compila corretamente e nenhum outro passo pode prosseguir.

- **Tests**: indica se a suíte de testes automatizados está passando. Um badge verde (passing) significa que todos os testes executaram sem falhas na última execução do pipeline.

- **Coverage**: exibe o percentual de cobertura de testes do código. Permite visualizar rapidamente se o projeto mantém um nível adequado de validação automatizada (ex.: "coverage: 87%").

- **Vulnerabilities**: indica o número e a severidade das vulnerabilidades encontradas nas dependências pelo `npm audit` (ou ferramenta equivalente). Um badge mostrando "0 vulnerabilities" ou "5 high" comunica imediatamente o estado de segurança do projeto.

---

**Questão 19**

A afirmação está **incorreta e é tecnicamente perigosa**.

Alta cobertura (>90%) é um indicador positivo, mas não elimina a necessidade de revisão de código pelos seguintes motivos:

1. **Cobertura não mede qualidade dos testes**: é possível ter 95% de cobertura com testes que não verificam nada de útil (sem asserções significativas).
2. **Revisão de código detecta problemas que testes não alcançam**: problemas de design, código ilegível, lógica de negócio incorreta, problemas de performance e de segurança muitas vezes só são identificados por um olhar humano.
3. **Testes não cobrem todos os cenários**: mesmo com alta cobertura de linhas, casos de borda, condições de corrida e falhas de integração podem passar despercebidos.
4. **Vulnerabilidades de segurança**: padrões de código inseguros (ex.: SQL injection, XSS) podem estar em código 100% coberto por testes que simplesmente não testam vetores de ataque.

No INPE, um desenvolvedor poderia cobrir 95% do código de processamento de alertas com testes, mas uma revisão de código poderia identificar que a lógica de priorização de severidade de eventos está invertida — o que os testes não pegaram por terem sido escritos com os mesmos dados incorretos.

**Testes automatizados e revisão de código são práticas complementares, não excludentes.**

---

**Questão 20**

Como responsável pela aprovação do deploy do projeto do INPE, analisaria os seguintes indicadores:

1. **Status do Build** – verificar se o pipeline de CI/CD passou completamente sem erros de compilação. Um build quebrado é bloqueador absoluto.

2. **Resultado dos Testes Automatizados** – todos os testes devem estar passando (0 falhas). Qualquer teste falhando indica regressão não resolvida.

3. **Cobertura de Testes** – verificar se está acima do threshold definido (ex.: mínimo 80%). Atenção especial aos módulos críticos de alertas, que devem ter cobertura ainda maior.

4. **Vulnerabilidades (npm audit)** – nenhuma vulnerabilidade de severidade Critical ou High deve estar presente. Moderate e Low devem ser documentadas com plano de correção.

5. **Relatório de Cobertura por Módulo** – verificar se módulos críticos (envio de alertas, processamento de denúncias, integração com satélites) têm cobertura adequada individualmente, não apenas a média geral.

6. **Histórico de Mudanças (Changelog/PR)** – revisar o que foi alterado neste release para avaliar o risco das mudanças e se os testes cobrem especificamente as novas funcionalidades.

7. **Testes de Integração e End-to-End** – confirmar que os fluxos críticos (denúncia → processamento → alerta → notificação) foram validados de ponta a ponta.

Somente com todos esses indicadores em estado satisfatório aprovaria o deploy, dado o impacto direto do sistema na segurança da população.
