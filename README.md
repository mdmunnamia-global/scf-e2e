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

## Execution Flow

This project does not implement a backend application or a stand-alone runtime service. It is a browser automation framework that drives a running web application. The execution flow is therefore: Cypress loads the framework, reads a `.feature` file, matches each Gherkin step to a JavaScript step definition, and then interacts with the application through selectors and assertions.

### 1. Where execution starts

Execution starts in the Cypress runtime, configured by `cypress.config.js`.

The real startup flow is:

1. Cypress loads `cypress.config.js`
2. `setupNodeEvents` is registered
3. The Cucumber preprocessor is initialized
4. Allure reporting is attached
5. Cypress scans the configured `specPattern` entries to find `.feature` files
6. Each file is parsed and mapped to matching step definitions

The root configuration sets the search pattern to include files like:

- `cypress/e2e/features/**/*.feature`
- `cypress/e2e/features/Admin/*.feature`
- `cypress/e2e/features/Configuration/*.feature`
- `cypress/e2e/features/Cad_Maker/*.feature`
- `cypress/e2e/features/2nd_Release/*.feature`

This is the first point where the project decides what will run.

### 2. Which file/module is executed first

The first executable unit is the feature file selected by Cypress from the `specPattern` list.

Example:

- `cypress/e2e/features/Cad_Maker/Limit_RF.feature`

This file contains the scenario flow, including the `Background` and `Scenario` steps. The `Background` step is usually the first thing that runs before each scenario.

In this project, a common pattern is:

```gherkin
Background: Open Website With Valid url
    Given Open Browser and Visit Website
```

The actual implementation of `Open Browser and Visit Website` is not stored in a single application service; it is implemented in the step definition files under `cypress/e2e/step_definitions/` that match the same Gherkin text. This is how Cucumber links the feature file to the test logic.

### 3. How execution moves from one layer to another

The movement is controlled by the Cucumber preprocessor.

- The feature file defines human-readable steps
- `.cypress-cucumber-preprocessorrc.json` tells Cypress where step definitions live
- Cypress matches the step text to the corresponding JavaScript implementation
- The step definition then calls Cypress commands such as `cy.xpath(...)`, `.type()`, `.click()`, `.should()`, `.scrollIntoView()`
- The locator file provides the exact selector used by that Cypress command
- The data file provides the expected value or input value used by the assertion or interaction

This is the real flow:

```text
Feature step
  -> Cucumber step match
  -> Step definition method
  -> Locator file selector
  -> Cypress UI action/assertion
  -> Result recorded by Allure or the test runner
```

### 4. What each major file/module does during execution

#### `cypress.config.js`

Responsible for:

- loading Cypress configuration
- registering the Cucumber preprocessor
- registering the Allure plugin
- defining the feature file patterns
- setting viewport, browser-level, and timeout behavior

This file is not the application logic itself; it is the test runner bootstrap.

#### `.cypress-cucumber-preprocessorrc.json`

Responsible for:

- telling the Cucumber preprocessor where to search for step definitions
- enabling the mapping between `.feature` steps and JavaScript code

Without this file, the feature steps would not be matched to implementation.

#### `cypress/support/e2e.js`

Responsible for:

- importing support files
- registering `cypress-file-upload`
- registering the Allure plugin
- running a final `after()` hook

In this repo it is a framework-level setup file, not a business logic file.

#### `cypress/support/commands.js`

Responsible for:

- adding reusable custom Cypress commands
- formatting working dates for date-picker flows
- preparing helper logic used by tests

This file is not the app layer; it is a utility layer used by test steps.

#### `cypress/data/data.js`

Responsible for:

- storing shared values like URLs, client data, and cross-module constants
- centralizing common data for many scenarios

This file acts as the shared configuration/data layer for the framework.

#### `cypress/Locators/Cad_Maker/*.js`

Responsible for:

- storing XPath/CSS selectors for page elements
- defining the exact UI targets used by steps

Example from the repo:

- `Limit_RF-locators.js`
- `ODAccount = "//input[@data-testid='loanAccount']"`
- `ModuleSelect = "//select[@title='Select Module']"`
- `SupplierCreditLimit = "//input[contains(@data-testid,'supplierCreditLimit')]"`

These selectors are then consumed by step definitions through `locator.*` references.

#### `cypress/e2e/step_definitions/.../*.js`

Responsible for:

- executing the behavior described in the feature file
- interacting with the browser
- checking UI state
- asserting expected conditions
- using locator definitions and data definitions

Example from the repo:

```javascript
When('Select Module RF',  () => {
    cy.xpath(locator.ModuleSelect)
    .scrollIntoView()
    .should('be.visible')
    .select('reverse-factoring');
});
```

This is where the actual action happens.

#### `cypress/e2e/data/Cad_Maker/*.js`

Responsible for:

- defining expected values and test scenario data
- storing module-specific inputs and result expectations

Example:

- `RF_Report_Limit.js`
- exports `limitReportDataRF`
- contains expected numbers and labels such as sanction limit, utilized limit, and report values

This data is compared against the actual UI output in assertions.

#### `*.feature` files

Responsible for:

- defining the flow as readable business behavior
- describing the order of actions and validations

Example flow from `Limit_RF.feature`:

```gherkin
When Enter Cad_Maker User ID
Then Enter Cad_Maker Password
When Click on Login
Then Click on EOD OK
When Select Module RF
When Click on Limit
Then Click on Create Limit
```

This is the user/business-facing sequence.

### 5. How different components communicate with each other

The communication pattern in this codebase is very explicit and hierarchical:

- Feature files send business intent
- Step definitions interpret that intent
- Step definitions ask the locator layer for selectors
- Step definitions act on the browser using Cypress commands
- Step definitions read the data layer for expected values
- The running app responds in the DOM
- Cypress checks the DOM for assertions and reports pass/fail

This is not a service-to-service call chain. It is a browser-driven workflow in which the code communicates with the application through the DOM and page state.

### 6. How data flows through the system

The data path is simple and clear:

1. Static/global values are stored in `cypress/data/data.js`
2. Feature scenarios express the required flow and user actions
3. Step definitions use the flow to populate inputs, click elements, and navigate pages
4. Data values may be hardcoded in the step definition or pulled from the module data file
5. The UI receives those values and alters page state
6. Assertions compare actual DOM text/value against expected data from `cypress/e2e/data/...`

Example from the repo:

- `RF_Report_Limit.js` contains expected values like:
  - `sanctionLimit: "10,00,000.00"`
  - `utilizedLimit: "0.00"`
  - `availableLimit: "10,00,000.00"`
- A step definition then reads that value and asserts it against a visible UI element

This means the data moves like this:

```text
data file -> step definition -> DOM input/action -> application state -> DOM output -> assertion -> pass/fail result
```

### 7. Where business logic, validation, API calls, database operations, auth/authorization, and error handling take place

This is important: in this codebase, the repository itself does not contain the application’s business logic, API layer, database layer, or auth service implementation.

The actual project is a UI automation layer for an external application.

What is present in this repo:

- UI interaction logic in step definitions
- selector logic in locator files
- validation logic via Cypress assertions (`.should(...)`, `.and(...)`)
- data-driven expected results in data files
- browser-level workflow orchestration in feature files

What is not present in this repo:

- application backend business logic
- server-side API implementations
- database schema or repository code
- authentication/authorization service code
- custom error handling middleware on the server

These are handled by the target web application itself, not by this automation repository. The automation repository simply checks whether the application behaves as expected in the browser.

Therefore:

- business validation is performed by the application and verified in the automation layer
- API calls are made by the application under test, not by this repo directly
- database operations happen in the external app, not in the test project
- authentication/authorization are part of the application flow being automated
- error handling in this repo is mostly through Cypress test failures and assertion messages

### 8. What happens after each step and which component handles the next step

After each step, the next action depends on the application state and the next Gherkin step.

Example from `Limit_RF.feature`:

```gherkin
When Enter Cad_Maker User ID
Then Enter Cad_Maker Password
When Click on Login
Then Click on EOD OK
When Select Module RF
```

Execution sequence:

1. Feature step `When Enter Cad_Maker User ID`
   - Cucumber finds the step definition
   - Step definition locates the `UserID` selector
   - Cypress types the value into the UI
   - Browser state updates

2. Next feature step `Then Enter Cad_Maker Password`
   - The same process repeats with a different locator and different input

3. `When Click on Login`
   - Step definition locates the login button and triggers click
   - The app processes the login request
   - The next page loads

4. `Then Click on EOD OK`
   - The step definition calls the selector for the relevant confirmation element
   - The app continues to the next screen

5. `When Select Module RF`
   - The step definition selects the module from a dropdown
   - The rest of the scenario continues from that page state

There is no hidden queue or service bus here. The state transition is purely driven by the browser DOM and the next step in the feature file.

### 9. End-to-end execution sequence (practical example)

A practical flow in this repo is the RF Limit scenario.

#### Example: `Limit_RF.feature`

```gherkin
Background: Open Website With Valid url
    Given Open Browser and Visit Website

Scenario: Verify that Create Limit RF with different bank
    When Enter Cad_Maker User ID
    Then Enter Cad_Maker Password
    When Click on Login
    Then Click on EOD OK
    When Select Module RF
    When Click on Limit
    Then Click on Create Limit
    When Reload the page
    When Input RF Negative1 OD Account
    Then Click on Tick Mark
    Then Click on Successful OK
```

#### What happens in the actual runtime

1. Cypress starts from `cypress.config.js`
2. It finds the feature file from `specPattern`
3. It matches each step to the corresponding definition in the step definition files
4. The first step runs: `Open Browser and Visit Website`
5. The browser opens the configured website from the shared data config
6. The user ID step locates the login field using a selector from `Limit_RF-locators.js`
7. Password is entered with the next step
8. Login is clicked
9. The app handles the login and loads the next page
10. EOD confirmation is clicked
11. The module is selected from the dropdown
12. The limit page opens and the flow continues
13. Inputs are entered into fields such as account number, financing rate, supplier details, etc.
14. Assertions verify page state, validation messages, or expected data values
15. The test passes or fails depending on whether the DOM matches the expected output

This is the full browsing sequence: feature step → browser action → DOM state → next step → next UI action.

### 10. Final execution model in one sentence

The codebase executes in a strict browser-driven sequence: configuration bootstraps Cypress, feature files define the business flow, step definitions implement that flow, locator files identify the UI elements, data files provide runtime values and expected outputs, and the browser/application under test responds through DOM state and UI behavior while Cypress validates each result.

This is the core execution model of the project and the correct mental model for understanding how the codebase works.

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

### Cypress runtime configuration

This file configures:

- Cypress project settings
- feature spec patterns
- viewport size
- Chrome security settings
- Allure integration
- Cucumber preprocessor setup

### Cucumber step-definition registration

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
