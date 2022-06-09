TODO
- Wat doe je als een test faalt? Maak dat duidelijk in de repo
- Unit testing is niet nodig als er geen buitengewoon ingewikkelde logica is
- Contract Testing/Integration Testing is veel belangrijker
- Frontend testing wel


# Testing Plan

To assure quality of both code and design various forms of testing can be applied. In this chapter I will be going over the different test types I will be using in this project. For each I will detail how and why I plan to use them. Note that I will not specify the exact tests here.

## Functional Tests

Functional tests focus on the question: Does the code do what it should do?

### Unit testing

The first and most granular form of testing is unit testing. Unit testing is involves testing individual functions within the code. To do this we mock (or stub) any dependencies they have so we can test only what the function does without any external factors impacting it.

I have decided not to use unit tests within the current scope of the project due to having very almost no logic that is worth testing - most of the application is relatively basic CRUD operations or operations that require a wider scope to test - and will thus be tested via integration and system tests. Thus, although it would be possible to write unit tests, they wouldn't contribute much to overall quality of the end product.

### Integration testing

Integration tests focus on the interfaces and interactions between components. The scope of the tests can vary based on testing method and phase - some integration tests might be limited to the interaction between a few classes within a single application, while some span multiple micro-services or include the database (src: [atlassian](https://www.atlassian.com/continuous-delivery/software-testing/types-of-software-testing)).

For FOTL I will be using a form of Big Bang Testing - a strategy where all components/modules are tested together. Specifically i will be testing the controller-repository-database stack. This will be done both locally and as part of my CI pipeline, by deploying a test database and running the tests against that. These tests will ensure that any changes to the APIs are tested.

Big Bang testing is a viable strategy mainly due to the scale of the project. I have only a few small systems which makes it a convenient strategy that is simple to implement while minimizing the downside - the difficulty of fault localization.

## Contract testing

Maybe? https://docs.pact.io/

## Acceptance Tests

Acceptance test focus on the question: Does the application do what it should do?

### End-to-End testing

End-to-end tests verify that the complete system does what it's supposed to do and does it correctly by simulating user behavior. In the case of FOTL this will be done with Selenium. After functional tests have passed the pipeline will deploy the service to the test environment - an azure webapp - and then the Selenium tests will be run against that. Tests will focus on happy flow and "dangerous" unhappy flows that require proper handling due to for example security - eg attempting to log in with an incorrect password.

### Acceptance testing

Acceptance tests will be executed manually on a user-story basis. These will encompass the acceptance criteria of the user story as well as testing usability of the newly added functionality.
