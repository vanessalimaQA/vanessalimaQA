# 🚀 QA Automation Framework

<p align="center">

Framework de automação de testes desenvolvido em **Java**, utilizando **Playwright**, **JUnit 5** e arquitetura em camadas para automação Web, integração com serviços SOAP e validação de regras de negócio.

</p>

<p align="center">

![Java](https://img.shields.io/badge/Java-21-orange?style=for-the-badge&logo=openjdk)
![Playwright](https://img.shields.io/badge/Playwright-Automation-45ba4b?style=for-the-badge&logo=playwright)
![JUnit5](https://img.shields.io/badge/JUnit-5-red?style=for-the-badge)
![Maven](https://img.shields.io/badge/Maven-Build-blue?style=for-the-badge&logo=apachemaven)
![SOAP](https://img.shields.io/badge/API-SOAP-purple?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-Repository-black?style=for-the-badge&logo=github)

</p>

---

# 📌 Sobre o Projeto

Este projeto foi desenvolvido com o objetivo de demonstrar a construção de um framework de automação seguindo princípios de organização, reutilização de código e separação de responsabilidades.

A arquitetura foi projetada para permitir a evolução contínua do framework, suportando automação Web, integração com Web Services SOAP, leitura de XML, validações de regras de negócio e acesso ao banco de dados.

Além da automação, o projeto também serve como laboratório para estudo de boas práticas de desenvolvimento aplicadas à Engenharia de Qualidade.

---

# 🎯 Objetivos

- Automatizar cenários Web utilizando Playwright;
- Organizar testes utilizando Page Object Model;
- Implementar arquitetura em camadas;
- Facilitar reutilização de componentes;
- Integrar testes com serviços SOAP;
- Preparar o framework para validações em banco de dados;
- Evoluir continuamente a arquitetura.

---

# 🏗 Arquitetura

```
                 TESTS
                    │
                    ▼
                 FLOW LAYER
                    │
      ┌─────────────┴─────────────┐
      ▼                           ▼
   PAGE OBJECTS              SERVICES
      │                           │
      ▼                           ▼
 PLAYWRIGHT                 SOAP CLIENT
                                  │
                                  ▼
                               BUILDER
                                  │
                                  ▼
                               PARSER
                                  │
                                  ▼
                              RESPONSE
                                  │
                                  ▼
                             VALIDATIONS

                    DATABASE
                        │
                        ▼
                   REPOSITORY
                        │
                        ▼
                 DATABASE EXECUTOR
                        │
                        ▼
                    SQL SERVER
```

---

# 📂 Estrutura

```
src
│
├── builder
├── client
├── config
├── database
├── factory
├── flow
├── locators
├── model
├── pages
├── parser
├── repository
├── request
├── response
├── service
├── utils
└── validator
```

---

# 🚀 Tecnologias

- Java 21
- Playwright
- JUnit 5
- AssertJ
- Maven
- SOAP
- XML
- SQL Server
- Git
- GitHub

---

# ✨ Recursos do Framework

- Automação Web
- Integração SOAP
- XML Builder
- XML Parser
- Repository Pattern
- Page Object Model
- Flow Layer
- Configuração por Properties
- Captura Automática de Evidências
- Organização em Camadas

---

# 🧪 Testes

O framework contempla diferentes tipos de testes, incluindo:

- Testes Web
- Testes Unitários
- Testes de Builders
- Testes de Services
- Testes de Validators
- Testes de Parser

---

# ▶ Executando

Executar toda a suíte

```bash
mvn clean test
```

Executar somente um teste

```bash
mvn -Dtest=QuitacaoWebTest test
```

---

# 📸 Evidências

Durante a execução dos testes Web, o framework realiza automaticamente a captura de screenshots para auxiliar na análise dos resultados.

---

# 🚧 Roadmap

## Implementado

- ✔ Playwright
- ✔ SOAP
- ✔ XML Builder
- ✔ XML Parser
- ✔ Repository Pattern
- ✔ Validator
- ✔ Captura de Evidências
- ✔ Estrutura em Camadas

## Próximas Evoluções

- REST Assured
- Allure Reports
- GitHub Actions
- Docker
- Apache POI
- Faker
- Jackson
- Execução Paralela

---

# 💡 Boas Práticas Aplicadas

- Page Object Model
- Single Responsibility Principle
- Reutilização de Componentes
- Builder Pattern
- Repository Pattern
- Organização em Camadas
- Configuração Externa
- Baixo Acoplamento

---

# 👩‍💻 Sobre a Autora

## Vanessa Lima

QA Automation Engineer em formação, dedicada ao desenvolvimento de soluções de automação de testes utilizando Java e boas práticas de Engenharia de Software.

### Áreas de Interesse

- Automação Web
- Automação de APIs
- Engenharia de Qualidade
- Arquitetura de Frameworks
- Testes de Integração
- Testes de Regressão

### Tecnologias

Java • Playwright • Selenium • SOAP • SQL Server • Maven • Git

---

# 📫 Contato

**LinkedIn**

https://www.linkedin.com/in/vanessa-ads/

**GitHub**

https://github.com/vanessalimaQA

---

⭐ Este projeto faz parte da evolução contínua do meu portfólio em Quality Assurance e Automação de Testes.
