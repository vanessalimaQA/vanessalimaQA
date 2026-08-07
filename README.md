<div align="center">

# Financial Validation Platform

### Automação de Testes & Quality Engineering aplicada a fluxos financeiros

**Java 21 · Playwright · REST Assured · SOAP · SQL Server · JUnit 5 · Maven · Allure**

Framework de automação desenvolvido para validar operações financeiras em múltiplas camadas, combinando **Web, APIs, serviços SOAP, banco de dados, regras de negócio e processamento batch** em uma arquitetura orientada a domínio e preparada para evolução.

**Vanessa Lima**
QA Automation | Quality Engineering

</div>

---

## O problema que este projeto resolve

Em sistemas financeiros, validar apenas a interface ou o retorno de uma API não é suficiente.

Uma operação pode ser aceita na entrada e ainda produzir um resultado financeiro incorreto depois do processamento.

Um pagamento, por exemplo, pode atravessar:

```text
Massa financeira
      │
      ▼
Web / REST / SOAP
      │
      ▼
Regra de negócio
      │
      ▼
Persistência
      │
      ▼
Processamento Batch
      │
      ▼
Estado financeiro resultante
```

Por isso, este projeto não foi estruturado como uma coleção de scripts E2E.

A proposta é construir uma **plataforma de validação financeira** capaz de verificar o comportamento da operação em diferentes pontos do fluxo e manter as regras de negócio independentes do canal utilizado para executá-las.

---

## Visão da solução

A arquitetura foi dividida para responder quatro perguntas diferentes:

| Pergunta                             | Responsabilidade        |
| ------------------------------------ | ----------------------- |
| Qual cenário quero validar?          | `PaymentScenario`       |
| Qual é a regra financeira?           | Business Layer          |
| Como a operação será executada?      | Integration Layer       |
| O resultado persistido está correto? | Repository + Validators |

Isso evita concentrar regra de negócio, SQL, chamadas externas e assertions dentro do mesmo teste.

```text
                        ┌─────────────────────┐
                        │      Test Layer     │
                        └──────────┬──────────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │        Flow         │
                        │   Orquestração      │
                        └──────────┬──────────┘
                                   │
                                   ▼
                 ┌─────────────────────────────────┐
                 │         Business Layer          │
                 │                                 │
                 │ Scenario → Resolver → Calculator│
                 │          → Context              │
                 └───────────────┬─────────────────┘
                                 │
                                 ▼
                       PaymentInstruction
                                 │
                  ┌──────────────┼──────────────┐
                  ▼              ▼              ▼
                REST            SOAP            Web
                  │              │              │
                  └──────────────┼──────────────┘
                                 ▼
                              Sistema
                                 │
                                 ▼
                            SQL Server
                                 │
                                 ▼
                           Validators
```

### Decisão central da arquitetura

`PaymentInstruction` não conhece Playwright, REST Assured, SOAP ou SQL.

Ela representa **o que deve ser executado**, não **como será executado**.

Isso permite que a regra financeira continue reutilizável mesmo quando o mecanismo de integração muda.

---

## Cenário implementado: pagamentos de fatura

O primeiro domínio financeiro modelado pela plataforma é pagamento de fatura.

Foram implementados seis comportamentos:

| Cenário         | Comportamento                                  |
| --------------- | ---------------------------------------------- |
| `BELOW_MINIMUM` | pagamento abaixo do mínimo                     |
| `MINIMUM`       | pagamento exatamente no mínimo                 |
| `PARTIAL_20`    | pagamento parcial dentro da faixa mínimo/total |
| `PARTIAL_60`    | segundo ponto da faixa parcial                 |
| `TOTAL`         | pagamento integral                             |
| `ABOVE_TOTAL`   | pagamento superior ao saldo da fatura          |

A intenção não é criar massas com valores fixos.

O valor é calculado a partir dos dados reais da fatura.

### Exemplo

Para:

```text
Total  = R$ 250,00
Mínimo = R$ 200,00
```

temos:

```text
Faixa disponível = Total - Mínimo
                  = 250 - 200
                  = 50
```

O cenário `PARTIAL_20` produz:

```text
200 + (50 × 20%) = R$ 210,00
```

Enquanto `PARTIAL_60` produz:

```text
200 + (50 × 60%) = R$ 230,00
```

Consequentemente, a mesma regra funciona para diferentes faturas sem depender de valores mágicos inseridos nos testes.

Todos os cálculos monetários utilizam `BigDecimal`.

---

## Como uma regra vira um teste

O pipeline financeiro implementado atualmente é:

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

### `InvoicePaymentData`

Transporta os dados necessários da fatura.

### `PaymentScenario`

Expressa a intenção do teste em linguagem de domínio.

### `PaymentScenarioResolver`

Converte o cenário solicitado na regra correspondente.

### `PaymentAmountCalculator`

Centraliza os cálculos financeiros e suas restrições.

### `PaymentContext`

Representa o resultado da regra aplicada ao contexto da fatura.

### `PaymentScenarioFlow`

Orquestra a preparação do cenário sem acoplar a regra à infraestrutura.

### `PaymentInstruction`

Entrega uma instrução pronta para ser consumida posteriormente por um mecanismo de execução.

---

## Estratégia de validação em banco

A camada de persistência não é acessada diretamente pelos testes.

O framework utiliza:

```text
Test
  │
  ▼
Flow / Business
  │
  ▼
Repository
  │
  ▼
DatabaseExecutor
  │
  ▼
JDBC
  │
  ▼
SQL Server
```

Para o domínio de faturas, uma das relações utilizadas é:

```text
BoletosEmitidos
      │
      │ Id_Historico
      ▼
HistoricosCorrentes
```

A partir dela, a automação pode trabalhar com informações como:

```text
Id_Conta
Id_Historico
Id_Boleto
saldoAtualFinal
valorMinimoExtrato
```

O repository atual prepara a seleção da fatura mensal utilizando `id_tipoboleto = 7`.

A query permanece encapsulada no repository.

O teste conhece **comportamento**.

O repository conhece **persistência**.

---

## Estratégia de testes

Nem todo teste precisa chegar até a interface.

A suíte foi estruturada para testar cada responsabilidade no nível adequado.

```text
                  E2E
                   ▲
                  / \
                 /   \
                /     \
         Integração    \
              ▲         \
             /           \
      Business / Flow     \
            ▲              \
           /________________\
              Unitários
```

### Testes unitários

Validam componentes como:

* cálculos financeiros;
* contextos;
* factories;
* resolvers;
* builders;
* validators;
* geração de massa.

São rápidos e independentes de infraestrutura.

### Testes parametrizados

Os diferentes cenários financeiros percorrem o mesmo fluxo através de testes parametrizados, reduzindo duplicação e aumentando cobertura comportamental.

### Testes de integração

Cenários que dependem de infraestrutura são identificados explicitamente:

```java
@Tag("integration")
```

A suíte padrão exclui:

```text
external
integration
```

Assim:

```bash
mvn test
```

continua funcionando em uma máquina sem acesso ao SQL Server corporativo.

Isso também cria uma separação clara para futura execução em pipeline.

---

## Integrações suportadas

### Web

Automação construída com **Playwright**, utilizando abstrações para browser, context e Page Objects.

### REST

Estrutura com **REST Assured**, incluindo:

* `ApiClient`
* `ApiConfig`
* `RequestSpecFactory`
* `ResponseSpecFactory`
* requests e responses tipados

### SOAP

Camada própria contendo:

```text
Builder
   ↓
Request
   ↓
SoapClient
   ↓
Response
   ↓
Parser
   ↓
Service
```

Isso evita XML sendo montado diretamente dentro dos testes.

### Database

Infraestrutura JDBC separada em:

```text
DatabaseConfig
DatabaseConnection
DatabaseExecutor
DatabaseAccessor
RowMapper
Repositories
```

---

## Organização do código

```text
com.automation
│
├── business        # regras financeiras
├── config          # configuração de ambiente
├── core            # infraestrutura transversal
├── database        # abstração JDBC
├── excel           # Data-Driven Testing
├── factory         # criação de componentes
├── flow            # orquestração de cenários
├── integrations
│   ├── rest
│   └── soap
├── locators
├── model           # modelos de domínio
├── pages           # Page Objects
├── repository      # persistência
├── testdata        # criação e validação de massas
├── utils
└── validator       # validações reutilizáveis
```

A estrutura de testes acompanha as responsabilidades da aplicação:

```text
tests
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

## Stack técnica

| Responsabilidade       | Tecnologia        |
| ---------------------- | ----------------- |
| Linguagem              | Java 21           |
| Build & dependências   | Maven             |
| Test Runner            | JUnit 5           |
| Assertions             | AssertJ           |
| Web                    | Playwright        |
| REST                   | REST Assured      |
| SOAP                   | Apache HttpClient |
| Database               | JDBC + SQL Server |
| Serialização JSON      | Jackson           |
| Data-Driven            | Apache POI        |
| Test Data              | Datafaker         |
| Redução de boilerplate | Lombok            |
| Evidências / Reporting | Allure            |

---

## Segurança de configuração

Credenciais não fazem parte do código-fonte.

Arquivos locais de configuração são ignorados pelo Git e o repositório mantém apenas templates:

```text
application.properties.example
users.properties.example
```

Configurações sensíveis também podem ser fornecidas por:

```text
DB_URL
DB_USER
DB_PASSWORD
```

ou propriedades da JVM.

A precedência de configuração permite:

```text
JVM Property
     ↓
Environment Variable
     ↓
Environment Properties
```

Essa abordagem facilita execução local e futura integração com CI/CD sem inserir secrets no código.

---

## Executando o projeto

### Pré-requisitos

```text
Java 21+
Maven 3.9+
```

### Executar a suíte local

```bash
mvn clean test
```

A suíte padrão não depende de acesso ao banco corporativo.

### Compilar os testes de integração

```bash
mvn test-compile
```

Testes externos devem ser executados somente em ambientes autorizados e com as configurações necessárias disponíveis.

---

## Evidências de engenharia presentes no projeto

Este repositório não concentra sua complexidade em um único teste E2E.

As principais decisões implementadas incluem:

* separação entre domínio e infraestrutura;
* modelagem explícita de cenários financeiros;
* cálculo monetário com `BigDecimal`;
* Repository Pattern para persistência;
* abstração JDBC reutilizável;
* clientes independentes para REST e SOAP;
* builders para requests;
* responses tipados;
* factories para criação de contexto;
* fixtures reutilizáveis;
* testes parametrizados;
* validações isoladas;
* configuração por ambiente;
* separação entre testes locais e testes de integração;
* proteção de credenciais;
* estrutura preparada para expansão pós-batch.

---

## Próxima etapa: validação financeira pós-batch

A evolução mais importante da plataforma é a estratégia de snapshots.

```text
Selecionar massa
      │
      ▼
Snapshot financeiro inicial
      │
      ▼
Executar operação
      │
      ▼
Processamento Batch
      │
      ▼
Snapshot financeiro final
      │
      ▼
Comparação
      │
      ▼
Business Assertions
```

O objetivo é permitir validações como:

> dado um estado financeiro conhecido antes do processamento, o estado produzido depois do batch corresponde exatamente às regras esperadas?

Essa estratégia permitirá evoluir a plataforma para cenários de:

* pagamento de fatura;
* parcelamento;
* refinanciamento;
* quitação;
* cálculos pós-processamento;
* regressão financeira.

---

## Roadmap técnico

```text
[✓] Fundação Web
[✓] Infraestrutura REST
[✓] Infraestrutura SOAP
[✓] Abstração JDBC
[✓] Data-Driven Testing
[✓] Geração de massa
[✓] Domínio financeiro
[✓] Cálculos de pagamento
[✓] Cenários parametrizados
[✓] Isolamento de integração

[→] Integração com massa financeira real
[→] Seleção automática de contas
[→] Execução dos pagamentos
[→] Snapshot pré-batch
[→] Snapshot pós-batch
[→] Business Assertions pós-processamento
[→] Validação de parcelamento
[→] Validação de quitação
[→] Expansão da regressão financeira
[→] CI/CD
```

---

## O que este projeto representa

Este projeto nasceu de uma decisão que considero importante na minha evolução em Quality Engineering:

**não quero apenas automatizar passos; quero entender o comportamento que estou validando.**

Por isso, conforme o framework evolui, minha preocupação não está apenas em adicionar novas ferramentas.

Está em responder perguntas melhores:

* Qual regra de negócio estamos protegendo?
* Qual dado prova que ela funcionou?
* Em qual camada essa validação deveria acontecer?
* O teste realmente precisa de interface?
* Como separar falha de infraestrutura de falha de negócio?
* Como reutilizar uma regra em diferentes canais?
* Como tornar uma validação financeira reproduzível?
* Como descobrir rapidamente onde uma regressão aconteceu?

Para mim, maturidade em automação começa quando deixamos de medir qualidade pela quantidade de scripts e começamos a construir **confiança sobre o comportamento do produto**.

---

<div align="center">

### Vanessa Lima

**QA Automation · Quality Engineering**

Java · Playwright · API Testing · SQL · Test Architecture

**Automação com propósito: entender o negócio, estruturar a validação e gerar confiança no resultado.**

</div>
