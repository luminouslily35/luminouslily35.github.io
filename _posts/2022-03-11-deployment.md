---
title: Deploying the Pokédex
date: 2022-03-10 13:21:00 -0800
categories: [projects]
tags: [ci-cd, web-dev]
---
Although I have worked, debugged, and tested my source code at my past internships, I have never actually participated in the process of shipping a software product. This is why I have decided to tackle part 11 of the full-stack web development series, in order to learn more about this surprisingly challenging concept. You can learn more here: <https://fullstackopen.com/en/part11>

## Branches and pull requests

These two ideas are fundamental in the success of a software project with a developer team of size > 1. Because the main (or master) branch must be kept as polished and bug-free as possible, developers are expected to work on their own branches (to fix bugs, add a new feature, etc.) then create a *pull request* that needs to be approved by another human prior to the code changes to be integrated with the main branch.

## Github Actions

GitHub Actions is a service provided by GitHub that automates your development workflow, including building, testing, and deploying. GitHub Actions is triggered by GitHub events, such as a push to a certain branch (musually `main`), the opening of a pull request, and even a schedule that dictates when the build job is to occur. GitHub Actions executes your scripts, which are `.yml` or `.yaml` files in the `.github/workflows`{: /filepath} directory, which has to be syntaxed properly. Here is one example of a build script that I used for the forked Pokédex repository:

```yaml
name: Deployment pipeline

on:
  push:
    branches:
      - master
  pull_request:
    branches: [master]
    types: [opened, synchronize]

jobs:
  simple_deployment_pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: npm install
        run: npm install
      - name: lint
        run: npm run lint
      - name: build
        run: npm run build
      - name: test
        run: npm test
      - name: e2e tests
        uses: cypress-io/github-action@v2
        with:
          command: npm run test:e2e
          start: npm run start-prod
          wait-on: http://localhost:5000
      - name: deploy
        if: ${ { github.event_name == 'push' } }
        uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${ { secrets.HEROKU_API_KEY } }
          heroku_app_name: pokemon-fanatic
          heroku_email: "<email goes here>"
          healthcheck: "https://<heroku url goes here>/health"
          checkstring: "ok"
          rollbackonhealthcheckfailed: true
```
{:file='.github/workflows/pipeline.yml }

This particular script is triggered by a push to the `master` branch, as well as the opening of a pull request from some other branch to `master`. When triggered, GitHub Actions executes a collection of tasks known as jobs. This script only has one job: `simple_deployment_pipeline`, but can have more.

Please note that much like Python, yaml files depend on indents for proper execution.

`simple_deployment_pipeline` consists of many steps, but first it needs to instantiate an operating system container. Here, it is the latest version of `ubuntu`. After an instance of `ubuntu` is created, this is what happens next:

* `actions/checkout` checks out the repository (if you do not perform this step, GitHub Actions won't have any files to operate on!)

* `actions/setup-node` sets up node.js, version 16

* `npm install` installs the dependencies listed under `packages.json`

* `npm run lint` runs the linter (the lint command must be configured in `packages.json`)

* `npm run build` builds the software

* `npm test` runs tests (here, Jest was testing React components)

* `cypress-io/github-action` executes end-to-end tests, or integration tests. The ubuntu instance will launch Cypress under port 5000, and then will carry out the tests in the `cypress/` folder. (Note: I actually had some trouble with Cypress testing when I was using a GitHub Actions mocking library called `act`, so I commented it out. `act` is run locally on your computer as a Docker container, and is useful when your build script grows large, so that you can test that the script works before pushing it to GitHub.)

* `heroku-deploy` deploys your web application to Heroku, and if configured, also performs a health check by sending a GET request to `/health`. If the server responds with `ok`, then everything is fine; otherwise the software is rolled back to the previous version. You require a `HEROKU_API_KEY` that must be set in GitHub under the Secrets page in Settings, which is used to authenticate GitHub Actions to allow deployment.

  * Also, note that the `deploy` step only occurs if the GitHub event that triggered this workflow was `push` (in other words, a pull request would not cause the app to deploy to Heroku.)

## Conclusion

By forking this Pokedéx repository and adding my own build scripts for GitHub Actions, I was able to learn several key aspects of the continuous integration / continuous deployment process.

1. The GitHub Actions cloud compute is not particularly high spec: it only has two cores, 7GB RAM, and 14 GB of SSD space. [^1] Therefore, running the entire pipeline can be time consuming, particularly when installing `node_modules`.

2. While automating the process of building and testing the software is highly convenient, the developer must also build, lint, test, and deploy on their own machine to verify that each of these steps actually works.

3. Though this particular build file was relatively simple (albeit with many steps), scripts *can* get complicated. This is why one should ideally use `act` (as mentioned earlier) or run the script on a dummy GitHub repository before pushing to the actual repo.

You can see my forked Pokédex repository [here](https://github.com/luminouslily35/full-stack-open-pokedex/)!

(By the way, my favorite Pokemon is Espeon.)

### Citations

[^1]: https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners
