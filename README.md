# Bornali Test Automation

A Cypress + Cucumber BDD test automation framework for end-to-end validation of web application workflows.

This project follows a modular architecture where each functional area is separated into its own folders for:

- feature files
- step definitions
- UI locators
- test data
- shared configuration

It is designed to be easy to scale, maintain, and reuse across multiple business modules and test flows.

## Project Overview

- Type: Cypress E2E + Cucumber BDD
- Reporting: Allure
- Browser: Chrome
- Test style: Behavior-driven development (Gherkin)
- Main purpose: automate UI workflows and validate expected results against business rules

## Technology Stack

- Cypress: ^12.17.4
- @badeball/cypress-cucumber-preprocessor: ^15.0.0
- @bahmutov/cypress-esbuild-preprocessor: ^2.1.5
- @shelex/cypress-allure-plugin: ^2.41.2
- cypress-xpath: ^2.0.1
- cypress-file-upload: ^5.0.8
- allure-commandline: ^2.34.1
- multiple-cucumber-html-reporter: ^3.5.0
- Node.js + npm

## Architecture Summary

The framework follows a layered structure:

1. Feature layer
   - Human-readable BDD scenarios in `.feature` files
   - Located in `cypress/e2e/features/`

2. Step definition layer
   - Maps Gherkin steps to Cypress actions and assertions
   - Located in `cypress/e2e/step_definitions/`

3. Locator layer
   - Centralizes all UI selectors (XPath/CSS)
   - Located in `cypress/Locators/`

4. Data layer
   - Holds test inputs and expected values
   - Located in `cypress/e2e/data/` and `cypress/data/`

5. Support layer
   - Global Cypress setup, plugins, commands, and shared utilities
   - Located in `cypress/support/`

6. Configuration layer
   - Cypress runtime, Cucumber integration, and reporting settings
   - Root files: `cypress.config.js`, `.cypress-cucumber-preprocessorrc.json`, `package.json`

## Project Structure

```text
bornali-tests/
├── .cypress-cucumber-preprocessorrc.json
├── .gitignore
├── .gitlab-ci.yml
├── README.md
├── cypress.config.js
├── dockerfile
├── jsconfig.json
├── package.json
├── package-lock.json
├── cypress/
│   ├── Locators/
│   │   ├── Admin/
│   │   ├── Configuration/
│   │   ├── Cad_Maker/
│   │   └── 2nd_Release/
│   ├── data/
│   │   └── data.js
│   ├── downloads/
│   ├── e2e/
│   │   ├── data/
│   │   │   └── Cad_Maker/
│   │   ├── features/
│   │   │   ├── Admin/
│   │   │   ├── Configuration/
│   │   │   ├── Cad_Maker/
│   │   │   └── 2nd_Release/
│   │   └── step_definitions/
│   │       ├── Admin/
│   │       ├── Configuration/
│   │       ├── Cad_Maker/
│   │       └── 2nd_Release/
│   ├── fixtures/
│   ├── screenshots/
│   ├── support/
│   │   ├── commands.js
│   │   ├── dateFormatter.js
│   │   └── e2e.js
│   └── videos/
├── allure-results/
├── allure-report/
├── cucumber-html-report.js
└── AUTOMATION_ARCHITECTURE_SUMMARY.md
```

## How the Files Connect

The main execution flow is:

```text
Feature File (.feature)
  -> Step definition (.js)
     -> Locator file (.js)
     -> Data file (.js)
     -> Cypress executes UI interaction/assertion
     -> Allure reports results
```

Example pattern:

- `RF_Report_Limit.feature` defines a scenario
- `RF_Report_Limit-StepDefinations.js` contains the matching code for the steps
- `RF_Report_Limit-locators.js` contains the XPath selectors used by those steps
- `RF_Report_Limit.js` contains expected values for assertions
- `cypress/data/data.js` provides shared values such as URLs and common constants

This separation keeps the test logic reusable and maintainable.

## Setup and Installation

### Prerequisites

- Node.js installed
- npm installed
- VS Code recommended
- Chrome browser installed
- Git installed

### Install Dependencies

```bash
npm install
```

### Initial Setup (if starting from scratch)

```bash
npm init -y
npm install cypress@12.17.4
npm install --save-dev cypress-cucumber-preprocessor
```

## Available Scripts

From `package.json`:

```json
"scripts": {
  "test": "cypress open --e2e --browser chrome",
  "cypress:execution": "cypress run --spec cypress/e2e/features/*",
  "cypress:execution-tags": "cypress run --env tags=@mobile",
  "allure:execution-headless": "cypress run --env allure=true",
  "allure:execution-headed": "cypress open --e2e --browser chrome --env allure=true",
  "allure:clear": "rimraf allure-results allure-report cypress/screenshots || true",
  "allure:report": "allure generate allure-results --clean -o allure-report",
  "allure:history": "mv -f allure-report/history allure-results/history && rm -r allure-report || true"
}
```

## Run Tests

### Open Cypress UI

```bash
npm test
```

### Run all feature files headless

```bash
npm run cypress:execution
```

### Run by tag

```bash
npm run cypress:execution-tags
```

### Run with Allure reporting

```bash
npm run allure:execution-headless
```

### Generate HTML report

```bash
npm run allure:report
```

## Cucumber and Cypress Configuration

### `cypress.config.js`

This file configures:

- Cypress project settings
- feature spec patterns
- viewport size
- Chrome security settings
- Allure integration
- Cucumber preprocessor setup

### `.cypress-cucumber-preprocessorrc.json`

This file tells the Cucumber preprocessor where to look for step definition files so feature steps can be automatically matched.

## Shared Conventions Used in This Project

The project follows common reusable patterns:

- Feature folder per business domain
- Step definition file names match feature behavior or module name
- Locator file names are separate from step files
- Test data is externalized from logic
- Shared URL and config values are centralized
- Assertions compare UI values to expected values from data files
- Reporting is integrated at the framework level

## Best Practices Followed in This Repo

- Keep feature files business-readable
- Keep automated logic in step definitions
- Keep locators centralized instead of hardcoding inline selectors
- Keep data separate from execution logic
- Reuse shared global values from `cypress/data/data.js`
- Keep modules independent and maintainable
- Use descriptive naming conventions for locators and step definitions

## CI/CD

This project includes GitLab CI compatibility through `.gitlab-ci.yml` and uses a Docker-based browser environment for test execution.

Typical CI flow:

- install dependencies
- run Cypress in headless Chrome
- capture results
- generate Allure report or publish test artifacts

## Reporting

This project supports Allure reports and stores generated artifact outputs in:

- `allure-results/`
- `allure-report/`

## Notes

This project is a good structural template for building a similar automation codebase, but the domain-specific test cases, URLs, modules, and selectors should be adapted to the target application rather than copied blindly.

The key value of this repository is its architectural pattern and modular test organization, not the business-specific contents.

## Recommended Usage for New Projects

If you are creating a new automation project based on this structure:

1. Keep the same folder pattern
2. Create a module-based directory structure
3. Add feature files first
4. Add matching step definitions
5. Add locators and data files
6. Use shared config values from a common file
7. Run smoke tests before scaling the suite

## Summary

This repository demonstrates a clean, scalable, and maintainable Cypress + Cucumber automation architecture. The most important design principles are:

- separation of concerns
- modular structure by business domain
- centralized selectors
- externalized test data
- shared framework configuration
- structured reporting and CI integration

## License

ISC
