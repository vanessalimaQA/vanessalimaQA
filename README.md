<div align="center">

# Financial Validation Platform

### Framework de Quality Engineering e Automação para Validações Financeiras

**Java 21 • Playwright • REST Assured • SOAP • SQL Server • JUnit 5 • Maven**

Uma arquitetura de automação criada para validar fluxos financeiros complexos através das camadas de **Web, API, SOAP, Banco de Dados e processamento Batch**.

<br>

**Desenvolvido por Vanessa Lima**
*QA Automation • Quality Engineering*

</div>

---

## ✦ Sobre o projeto

Validar um sistema financeiro vai muito além de verificar se uma tela abriu corretamente ou se uma API retornou `200 OK`.

Uma única operação financeira pode atravessar diferentes camadas até chegar ao seu resultado final:

```text
Massa de Teste
      ↓
Web / API / SOAP
      ↓
Regras de Negócio
      ↓
Processamento Financeiro
      ↓
Banco de Dados
      ↓
Processamento Batch
      ↓
Validação Pós-Processamento
```

A **Financial Validation Platform** nasceu a partir dessa necessidade.

O projeto está sendo desenvolvido como uma plataforma de automação capaz de validar um fluxo financeiro desde a entrada da operação até o resultado persistido após o processamento.

A arquitetura separa responsabilidades entre:

* regras de negócio;
* cálculos financeiros;
* dados de teste;
* integrações;
* acesso ao banco;
* execução dos cenários;
* validações;
* evidências e relatórios.

O objetivo não é apenas automatizar casos de teste.

> **O objetivo é construir uma plataforma onde regras financeiras possam ser validadas de forma reutilizável, rastreável e independente da tecnologia utilizada para executar a operação.**

---

# ✦ Princípios de Engenharia

Algumas decisões orientam a construção do framework.

### Regra de negócio não deve depender da interface

Os cálculos financeiros ficam isolados na camada de negócio.

Isso permite validar uma regra sem precisar abrir navegador, chamar API ou acessar banco de dados.

---

### Testes de integração não devem impedir o desenvolvimento local

Testes que dependem de SQL Server ou ambientes externos são separados da suíte padrão.

Assim:

```bash
mvn test
```

continua executando os testes locais mesmo quando o desenvolvedor não possui acesso à infraestrutura corporativa.

---

### Massa de teste deve representar cenários de negócio

Valores financeiros não ficam espalhados arbitrariamente pelos testes.

Os cenários são modelados explicitamente:

```text
Pagamento abaixo do mínimo
Pagamento mínimo
Pagamento parcial
Pagamento total
Pagamento acima do total
```

---

### Infraestrutura deve ser substituível

A regra financeira não precisa saber se a operação será executada futuramente por:

```text
SOAP
REST
Web
Banco
Batch
```

A camada de negócio permanece independente da tecnologia.

---

### Valores financeiros exigem precisão

Cálculos monetários utilizam:

```java
BigDecimal
```

evitando problemas de precisão associados a tipos de ponto flutuante.

---

# ✦ Arquitetura

O framework utiliza uma arquitetura em camadas para manter responsabilidades bem definidas.

```text
                         TEST LAYER
                             │
                             ▼
                    Cenários / Test Flow
                             │
                             ▼
                      BUSINESS LAYER
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        Regras Financeiras  Cálculos    Validators
              │              │              │
              └──────────────┼──────────────┘
                             │
                             ▼
                    INTEGRATION LAYER
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
             REST           SOAP         Database
              │              │              │
              └──────────────┼──────────────┘
                             │
                             ▼
                     Sistemas Externos
```

Essa separação permite que regras financeiras sejam testadas localmente enquanto integrações são executadas somente nos ambientes onde a infraestrutura necessária está disponível.

---

# ✦ Estrutura do projeto

```text
src
├── main
│   └── java
│       └── com.automation
│
│           ├── business
│           │   └── Regras e cálculos financeiros
│           │
│           ├── config
│           │   └── Configuração de ambientes
│           │
│           ├── core
│           │   └── Infraestrutura do framework
│           │
│           ├── database
│           │   └── JDBC e utilitários de banco
│           │
│           ├── excel
│           │   └── Suporte a testes Data-Driven
│           │
│           ├── factory
│           │   └── Factories do framework
│           │
│           ├── flow
│           │   └── Orquestração dos fluxos
│           │
│           ├── integrations
│           │   ├── rest
│           │   └── soap
│           │
│           ├── model
│           │   └── Modelos de domínio
│           │
│           ├── pages
│           │   └── Page Objects
│           │
│           ├── repository
│           │   └── Acesso aos dados
│           │
│           ├── testdata
│           │   └── Geração e preparação de massas
│           │
│           ├── utils
│           │   └── Utilitários compartilhados
│           │
│           └── validator
│               └── Validações de domínio
│
└── test
    └── java
        └── com.automation.tests
            ├── builder
            ├── business
            ├── excel
            ├── fixtures
            ├── flow
            ├── rest
            ├── service
            ├── testdata
            ├── validator
            └── web
```

---

# ✦ Domínio Financeiro

Uma das principais áreas atualmente implementadas é a validação de **pagamentos de fatura**.

Em vez de definir valores fixos diretamente nos testes, o framework representa cada comportamento através de cenários financeiros.

| Cenário         | Regra                                                    |
| --------------- | -------------------------------------------------------- |
| `BELOW_MINIMUM` | Pagamento abaixo do valor mínimo                         |
| `MINIMUM`       | Pagamento exatamente no valor mínimo                     |
| `PARTIAL_20`    | Pagamento parcial utilizando 20% do intervalo disponível |
| `PARTIAL_60`    | Pagamento parcial utilizando 60% do intervalo disponível |
| `TOTAL`         | Pagamento integral da fatura                             |
| `ABOVE_TOTAL`   | Pagamento acima do valor total                           |

---

# ✦ Cálculos Financeiros Dinâmicos

Considere uma fatura:

```text
Valor Total  = R$ 250,00
Valor Mínimo = R$ 200,00
```

Para gerar pagamentos parciais, o framework primeiro calcula:

```text
diferença = valorTotal - valorMinimo
```

Resultado:

```text
diferença = 250 - 200
diferença = 50
```

### Pagamento parcial de 20%

```text
pagamento = mínimo + (diferença × 20%)

pagamento = 200 + 10

pagamento = R$ 210,00
```

### Pagamento parcial de 60%

```text
pagamento = mínimo + (diferença × 60%)

pagamento = 200 + 30

pagamento = R$ 230,00
```

Com isso, os testes não dependem de valores arbitrários.

Uma fatura de R$ 250,00 e outra de R$ 3.000,00 podem utilizar exatamente o mesmo cenário.

---

# ✦ Pipeline dos Cenários Financeiros

Atualmente, a preparação de um pagamento percorre:

```text
InvoicePaymentData
        │
        ▼
PaymentScenario
        │
        ▼
PaymentScenarioResolver
        │
        ▼
PaymentAmountCalculator
        │
        ▼
PaymentContext
        │
        ▼
PaymentScenarioFlow
        │
        ▼
PaymentInstruction
```

Cada componente possui uma responsabilidade específica.

### `InvoicePaymentData`

Representa os dados financeiros necessários para o cenário.

### `PaymentScenario`

Representa **qual comportamento financeiro será testado**.

### `PaymentScenarioResolver`

Traduz o cenário para a regra financeira correspondente.

### `PaymentAmountCalculator`

Executa os cálculos monetários.

### `PaymentContext`

Mantém o contexto financeiro calculado.

### `PaymentScenarioFlow`

Orquestra a preparação do cenário.

### `PaymentInstruction`

Produz uma instrução de pagamento independente da tecnologia que futuramente executará a operação.

---

# ✦ Validação em Banco de Dados

O acesso ao banco é isolado através de repositories e infraestrutura JDBC.

Uma das relações utilizadas atualmente é:

```text
BoletosEmitidos
       │
       │ Id_Historico
       ▼
HistoricosCorrentes
```

Essa relação permite recuperar informações financeiras como:

```text
Id_Conta
Id_Historico
Id_Boleto
saldoAtualFinal
valorMinimoExtrato
```

Para os cenários atuais de pagamento de fatura, utilizamos inicialmente:

```text
id_tipoboleto = 7
```

As queries ficam isoladas dos testes.

Isso mantém a responsabilidade clara:

```text
TESTE
  ↓
FLOW
  ↓
BUSINESS
  ↓
REPOSITORY
  ↓
DATABASE
```

O teste valida comportamento.

O repository conhece SQL.

---

# ✦ Estratégia de Testes

O framework separa diferentes níveis de validação.

```text
                 ┌─────────────────────┐
                 │       E2E           │
                 └──────────┬──────────┘
                            │
                 ┌──────────▼──────────┐
                 │     Integração      │
                 └──────────┬──────────┘
                            │
                 ┌──────────▼──────────┐
                 │ Regras de Negócio   │
                 └──────────┬──────────┘
                            │
                 ┌──────────▼──────────┐
                 │     Unitários       │
                 └─────────────────────┘
```

A ideia é validar a regra o mais próximo possível do domínio e utilizar testes End-to-End quando realmente precisamos provar a integração entre diferentes componentes.

Isso reduz:

* tempo de execução;
* dependência de ambiente;
* falsos negativos;
* manutenção desnecessária.

---

# ✦ Testes Locais

A suíte principal pode ser executada com:

```bash
mvn test
```

Testes dependentes de infraestrutura externa não participam da execução padrão.

---

# ✦ Testes de Integração

Testes que precisam de banco ou infraestrutura real são identificados com:

```java
@Tag("integration")
```

O Maven exclui por padrão:

```text
external
integration
```

Isso permite que qualquer pessoa clone o projeto e execute a suíte local sem possuir acesso ao SQL Server utilizado nos testes de integração.

---

# ✦ Segurança e Configuração

Credenciais não devem fazer parte do código-fonte.

Arquivos locais contendo informações sensíveis são ignorados pelo Git.

O projeto utiliza templates seguros:

```text
application.properties.example
users.properties.example
```

Cada ambiente pode criar sua própria configuração local.

Informações sensíveis também podem ser fornecidas através de variáveis de ambiente:

```text
DB_URL
DB_USER
DB_PASSWORD
```

Exemplo:

```powershell
$env:DB_URL="jdbc:sqlserver://SERVER:PORT;databaseName=DATABASE"
$env:DB_USER="YOUR_USER"
$env:DB_PASSWORD="YOUR_PASSWORD"
```

> **Nenhuma credencial real deve ser versionada no repositório.**

---

# ✦ Stack Tecnológica

| Área            | Tecnologia        |
| --------------- | ----------------- |
| Linguagem       | Java 21           |
| Build           | Maven             |
| Test Framework  | JUnit 5           |
| Assertions      | AssertJ           |
| Web Automation  | Playwright        |
| REST API        | REST Assured      |
| SOAP            | Apache HttpClient |
| Banco de Dados  | SQL Server + JDBC |
| JSON            | Jackson           |
| Massa de Testes | Datafaker         |
| Excel           | Apache POI        |
| Boilerplate     | Lombok            |
| Reporting       | Allure            |

---

# ✦ Capacidades Atuais

O framework já possui suporte para:

* automação Web com Playwright;
* Page Objects;
* testes REST;
* infraestrutura para SOAP;
* builders de requests SOAP;
* parsing de responses SOAP;
* SQL Server via JDBC;
* Repository Pattern;
* testes Data-Driven com Excel;
* geração de dados com Datafaker;
* regras de cálculo financeiro;
* modelagem de cenários de pagamento;
* fixtures reutilizáveis;
* testes parametrizados;
* separação entre testes locais e integração;
* evidências e relatórios com Allure.

---

# ✦ Roadmap

A plataforma está sendo construída de forma incremental.

```text
✓ Fundação do framework
✓ Automação Web
✓ Infraestrutura REST
✓ Infraestrutura SOAP
✓ Abstração de banco de dados
✓ Regras financeiras de pagamento
✓ Modelagem dos cenários
✓ Fixtures financeiras
✓ Isolamento dos testes de integração

→ Validação com SQL Server real
→ Seleção dinâmica de massas financeiras
→ Execução de pagamentos
→ Snapshot pré-batch
→ Processamento Batch
→ Validação pós-batch
→ Validação de parcelamento
→ Validação de quitação
→ Expansão da regressão financeira
→ Pipeline CI/CD
```

---

# ✦ Próxima Evolução: Validação Pós-Batch

Uma das evoluções planejadas é permitir que a plataforma capture o estado financeiro de uma conta antes do processamento:

```text
Conta
  ↓
Snapshot Pré-Batch
  ↓
Execução da operação
  ↓
Batch
  ↓
Snapshot Pós-Batch
  ↓
Comparação
  ↓
Validação financeira
```

Com isso, a automação poderá validar não apenas se uma operação foi executada, mas se **o resultado financeiro produzido pelo processamento está correto**.

---

# ✦ Quality Engineering na prática

Para mim, automação de testes não deve ser apenas uma sequência de comandos executados contra uma aplicação.

Ela precisa ajudar a responder perguntas importantes:

```text
A regra de negócio foi respeitada?

O cálculo está correto?

O dado persistido corresponde à operação realizada?

O processamento alterou exatamente o que deveria?

Se algo falhar, conseguimos identificar onde e por quê?
```

É essa visão que estou aplicando na construção deste projeto.

Quero que o framework continue evoluindo não apenas em quantidade de testes, mas principalmente em **qualidade de arquitetura, confiabilidade e entendimento do negócio**.

Porque uma boa automação não é aquela que simplesmente executa muitos testes.

**É aquela que gera confiança no produto que está sendo entregue.**

---

<div align="center">

## Vanessa Lima

### QA Automation • Quality Engineering

*Transformando regras de negócio em validações automatizadas, confiáveis e escaláveis.*

</div>
