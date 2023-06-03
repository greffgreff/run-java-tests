# Run Java Tests Action

Run tests from a java project as part of your workflows.  

Useful scenarios for such an action include running smoke tests with a cucumber project and is useful in preventing actions from taking place given failed cucumber tests.

Below is one such example, where a cucumber tests are ran before a docker image is created. A repository containing a java project is specified under `repo` that contains the cucumber tests in question. Optionally, one can specify a branch name under the `branch` property to target specific tests for instance. 

```yml
name: Exemple Pipeline

on:
  release:
    types:
      - created
      
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: "7.0.x"
      - name: Restore packages
        run: dotnet restore
      - name: Build
        run: dotnet build --configuration Release --no-restore
      - name: Test
        run: dotnet test --no-restore --verbosity normal

  regression-tests:
    needs: build

    runs-on: ubuntu-latest
    steps:
        - name: Run Cucumber Regression
          uses: greffgreff/run-java-tests-action@master
          with:
            repo: https://github.com/some-organization/some-java-project-with-tests.git
            branch: dev/pricing-integration
  
  docker:
    needs: regression-tests

    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        name: Check out code
      - name: Build & push Docker image
        uses: mr-smithers-excellent/docker-build-push@v6
        with:
          ...
```
