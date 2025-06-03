# Cushions Repo

Monorepo for TAKE2 stack + applications!
Based on this [repository](https://github.com/Couchsurfing/cushions)

## Getting Started

### Prerequisites

- Guide assumes native Mac OS `zsh` shell for now. `bash` support for user shell is not fully tested!
    - Some setup scripts specifically target `~/.zshrc` to setup environment.
    - Even though `zsh` is the default Mac OS shell, you may still need to create the config file: `touch ~/.zshrc`.
- Install Brew (if you already haven't) - [brew/homebrew package manager](https://brew.sh/)
- Install GIT `brew install git`
- Configure git (if not already configured) !!!must be accurate and unique to you!!!
    - `git config --global user.name`
    - `git config --global user.email`
- Set up [SSH Keys with your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- Clone the Cushions repository to your local (`git clone git@github.com:Couchsurfing/cushions.git`)
- Install [nvm](https://github.com/nvm-sh/nvm). You can run `nvm install` at the root of this repo to get the correct Node environment. Once installed, you can run `nvm use` to ensure you are using the correct Node version.
- Setup pnpm
    - Install `pnpm` globally with `npm i -g pnpm`. Almost all commands that you would normally run with `npm` can be run with `pnpm` in this repo.
    - Run `pnpm setup` - don't forget to source! `source ~/.zshrc`
    - Configure the couchsurfing npm registries by running `bash ./support/local/setup-npm-registries.sh`. You will need to open a new terminal for the following steps or `source ~/.zshrc`
- Install [Visual Studio Code](https://code.visualstudio.com/download)
    - Open VS Code, and Accept + install 'suggested' extensions via the notification in VS Code, which are configured by `.vscode/extensions.json` in the project.
- Install `helm` using brew `brew install helm` to use the platform in a Kubernetes environment.
- Install Terraform using brew `brew tap hashicorp/tap && brew install hashicorp/tap/terraform`
- Install GCP SDK `brew install --cask google-cloud-sdk`
- Login to GCP SDK `gcloud auth application-default login` using your @couchsurfing.com email
- Install Docker using [the section below in this README](#Docker-setup).
- Download [secrets.local.key](https://couchsurfingorg.atlassian.net/wiki/download/attachments/5734402/secrets.local.key?api=v2) & [secrets.ci.key](https://couchsurfingorg.atlassian.net/wiki/download/attachments/5734402/secrets.ci.key?api=v2) & [secrets.temp-staging.key](https://couchsurfingorg.atlassian.net/wiki/download/attachments/5734402/secrets.temp-staging.key?api=v2) - the pnpm start command will copy them to the right location

#### Mobile App Setup Requirements

Before you can `pnpm start` all the apps you need to do the following steps in the mobile README (follow at least to `Running The App` section)
[Mobile App README](apps/mobile/README.md). This includes long steps of downloading + installing XCode + Android Studio. If you receive an error related to NX not being found then you may need to run `pnpm install` prior to running the mobile app setup. Don't forget to continue back with the quick start once mobile app setup is complete.

### Quick Start

1. Install dependencies with `pnpm install`.
2. Start services with `pnpm start`. You may also start all tooling services as well with `pnpm start:full` to not start optional tooling services including only running slim service e2e tests with `pnpm test:e2e:slim`

See more details: [local commands](https://couchsurfingorg.atlassian.net/wiki/spaces/TGAS/pages/78741505/Nx+and+Local+Development+Commands)

The `pnpm start` command will attempt to create host entries in your `/etc/hosts`, create the initial env vars for all service, generate local SSL Certs, and then build and launch all the services in docker. It will then launch `https://local.couchsurfing.dev` in a browser.

If the browser is giving you an invalid certificate error, close and reopen your browser. The SSL Certs are self-signed and added to your local keychain, but the browser may not pick up the new certs until it is restarted.

If you are using Firefox, you may see a warning about the SSL Certs saying "MOZILLA_PKIX_ERROR_SELF_SIGNED_CERT". You can add an exception to proceed to the site.

#### Commit and push

There are commit hooks that run on commit and push by husky that use the following commands

1. `pnpm prepare:commit`
2. `pnpm prepare:push`

Prepare commit is focused on generation, file formatting and basic compilation.

Prepare push is focused on linting and tests

#### e2e

The pull request workflow will run tests, linting, and other quality checks but also ensure code generation has been executed. In order to mimic what happens on e2e it boils down to these commands.

1. `pnpm prepare:commit`
2. `pnpm prepare:push`
3. `pnpm test:e2e`

### Development

This repo uses [Prettier](https://prettier.io/) for code formatting. Just run `pnpm run format` to format the entire repo. Additionally, Eslint is configured for the repo, the linter
can be run using `pnpm run lint`.

Code should be written with [Typescript](https://www.typescriptlang.org/) when possible. Some tooling may require `.js` files instead, but all React, NodeJS, and scripts can leverage Typescript.

The `pnpm start` script will start the app in slim mode and re-initialize all data sources. `pnpm upgrade:slim` will just process any changes and apply pending migrations but will not touch the current installed k8s namespace.

To start all debug ports and the full application run `pnpm start:full` and to apply changes to ongoing changes to the deployed k8s application use `pnpm upgrade:full`

### Docker setup

- Log in to docker registry `gcloud auth configure-docker gcr.io`
- Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Configure the below settings
- Additionally: Give docker desktop full disk access through security -> full disk access -> docker (toggle)

This project uses Docker and Kubernetes. To see a visual representation of how the services are logically grouped together, see [Local-Development-Environment](https://couchsurfingorg.atlassian.net/wiki/spaces/TC/pages/150405134/High-Level+Components+for+Take2).

**IMPORTANT SETTINGS**

1. Virtual Machine Options - Docker VMM
2. Memory 32 GB MAX (24GB if only using pnpm start:slim)
3. CPU 8 cores
4. Swap 4GB
5. Disk 350GB
6. Resource saver - off
7. Enable - Kubernetes (k8s)
    - This will install key dependencies such as the `kubectl` CLI utility.
8. Disable - Do NOT use containerd for pulling and storing images
9. Disable - Send usage statistics
10. Disable - Show CLI hints
11. Disable - SBOM Indexing (background indexing should automatically be disabled)

**For Local:** Enable [Kubernetes](https://kubernetes.io) in _Docker Desktop -> Settings -> Kubernetes_ so you can run your local environment in a Kubernetes environment. This will install key dependencies such as the `kubectl` CLI utility.

To test and run docker locally the same as the ci environment simply run `pnpm start:docker:ci`.

### Secrets

If you find the need to introduce new secrets into the monorepo (passwords, tokens, API keys, etc.), you can use the `make-secret.js` support script in conjunction with the local/CI keys to commit them securely in the `support/infra/application-config/infra.*.yaml` definition files:

```sh
# Generate the value for all environments
$ echo "this^is^secret" | node ./support/scripts/make-secret.js

# Generate the value for the `local` environment:
$ echo "this^is^secret" | node ./support/scripts/make-secret.js --key ~/.couchsurfing/secrets.local.key
SECRET;1yS9Rr3zM0Jsc3XFcRomRE7sEbanphWMj7IAcTGpQZD9KA==

# Generate the value for the `ci` environment:
$ echo "this^is^secret" | node ./support/scripts/make-secret.js --key ~/.couchsurfing/secrets.ci.key
SECRET;Fh7HjiKz...
```

Should you find the need for the plain-text version of a secret, you can use the same script to decrypt an encrypted string:

```sh
# Just be sure you specify the appropriate matching key!
$ echo "SECRET;1yS9Rr3zM0Jsc3XFcRomRE7sEbanphWMj7IAcTGpQZD9KA==" | node ./support/scripts/make-secret.js --key ~/.couchsurfing/secrets.local.key -d
this^is^secret
```

### Testing

You can run linters, formatters, and tests (local) via package scripts: `pnpm lint`, `pnpm format`, and `pnpm test`.

To run e2e tests with Cypress, you can run `pnpm test:e2e:slim` or `pnpm test:e2e` (which will also run coverage).

To run unit tests for affected (changed ) projects with coverage, you can run `pnpm test:affected:coverage` alternatively you can run all tests with coverage `pnpm test:all:coverage`

We currently make use of Husky for pre-commit hooks. This will run the linter and formatter on all staged files before committing along with api and dart generation.

### e2e coverage

By default coverage will always be collected locally while the services are running in docker.

`pnpm test:e2e` will measure and report coverage along with if it will fail a build at current levels

### Breaking changes BFF

We are using the CI/CD (Github Actions) pipelines to generate `openAPI spec` documents and a `breaking changes` detection tool whenever there is a `PR -> main` created.
Leveraging 2 open source libs for this:

1. [openapi-trpc](https://github.com/dtinth/openapi-trpc)

2. [openapi-changes](https://pb33f.io/openapi-changes/)

`openapi-trpc` lib serves to generate `openAPI v3.0` compatible spec documents from a `tRPC` API.

`openapi-changes` lib is the most complete opensource detecting changes tool for openAPI spec files, the current implementation is set to generate a summary of the changes between 2 spec files it also detects breaking changes.

**NOTE:**
At this early in the development process it is likely that we introduce many breaking changes to the `BFF API` because of the constant restructuring of the infrastructure as new requirements are created, so as for now the current Breaking changes `job` is not required to complete a PR.

### postgres db migrations

See [Migrations doc](/docs/migrations.md).

### Creating new libs

There are helper utilities built in to generate new Typescript libs:

It will prompt you for name, subdirectory, and org.

To create a new lib under domains named `test`, you would answer `test`, `domains`, and `@surf`.

```sh
pnpm exec nx generate tooling/nx:ts-lib
```

To create a new React lib:

```sh
pnpm exec nx generate tooling/nx:react-lib
```

To create new nestjs app:

```sh
pnpm exec nx g tooling/nx:service-app
```

### OpenApi documentation

The OpenApi documentation for each service is accessible locally at `https://local-service.couchsurfing.dev:8080/{service_name}/docs` where `{service_name}` is the value of the related `project.json` file `name` field.
ex. `https://local-service.couchsurfing.dev:8080/location-service/docs`

### Connecting to PostgreSQL databases

To connect to one of the service PostgreSQL database locally, first tunnel into the appropriate database inside the Kubernetes cluster to make it accessible on `localhost`: `pnpm nx run infra:k8s-tunnel {service}-pgdb-service 5432:5432 zsh`

Once established, use the following parameters in your SQL client:
Url: `jdbc:postgresql://localhost:5432/{db_name}`

- the `db_name` can be found in ./apps/{service}/src/app/migrations/initdb.sql
- the username can be found in ./support/infra/application-config/infra.local.yaml, `credentials:{service}-pg:username`.
- the password can be found in the ./dist/infra/local/infra.local.env generated file, look for `CS_{SERVICE}_PG_PASSWORD` environment variable

You can `exit` out of the tunneled shell process to disconnect.

### Logging, Traces, and Metrics

Connect to grafana to search logs, metrics, and distributed tracing at `http://localhost:3000` a complete view of the observability approach can be found here `https://couchsurfingorg.atlassian.net/wiki/spaces/Technology/pages/498958337/Observability`
