# Short

[![Build Status](https://ci.time4hacks.com/api/badges/byliuyang/short/status.svg)](https://ci.time4hacks.com/byliuyang/short)
[![codecov](https://codecov.io/gh/byliuyang/short/branch/master/graph/badge.svg)](https://codecov.io/gh/byliuyang/short)
[![Maintainability](https://api.codeclimate.com/v1/badges/408644627586328ddd6c/maintainability)](https://codeclimate.com/github/byliuyang/short/maintainability)
[![Go Report Card](https://goreportcard.com/badge/github.com/byliuyang/short)](https://goreportcard.com/report/github.com/byliuyang/short)
[![Open Source Love](https://badges.frapsoft.com/os/mit/mit.svg?v=102)](https://github.com/byliuyang/short)

![Demo](promo/marquee.png)

## Preview

![Demo](doc/demo.gif)

## Get `s/` Chrome extension

Install it from [Chrome Web Store](https://short-d.com/r/ext) or build it
from [source](https://short-d.com/r/ext-code)

## Getting Started

### Accessing the source code

```bash
git clone https://github.com/byliuyang/short.git
```

### Prerequisites

- Docker v19.03.2

### Configure environmental variables

1. Create `.env` file at project root with the following content:

   ```env
   DOCKERHUB_USERNAME=local
   DB_USER=your_db_user
   DB_PASSWORD=your_db_password
   DB_NAME=your_db_name
   RECAPTCHA_SECRET=your_recaptcha_secret
   GITHUB_CLIENT_ID=your_Github_client_id
   GITHUB_CLIENT_SECRET= your_Github_client_secret
   JWT_SECRET= your_JWT_secret
   WEB_FRONTEND_URL=http://localhost:3000
   WEB_PORT=3000
   HTTP_API_PORT=80
   GRAPHQL_API_PORT=8080
   ```
   
1. Update `DB_USER`, `DB_PASSWORD`, `DB_NAME`, and `JWT_SECRET` with your own
   configurations.

### Create reCAPTCHA account

1. Sign up at [ReCAPTCHA](https://short-d.com/r/recaptcha) with the
   following configurations:

   | Field           | Value          |
   |-----------------|----------------|
   | Label           | `Short`        |
   | reCAPTCHA type  | `reCAPTCHAv3`  |
   | Domains         | `localhost`    |

1. Replace the value of `RECAPTCHA_SECRET` in the `.env` file with `SECRET KEY`.
1. Replace the value of `REACT_APP_RECAPTCHA_SITE_KEY` in
   `frontend/.env.development` file with `SITE_KEY`.

### Create Github OAuth application

1. Create a new OAuth app at
   [Github Developers](https://short-d.com/r/ghdev) with the
   following configurations:
   
   | Field                      | Value                                            |
   |----------------------------|--------------------------------------------------|
   | Application Name           | `Short`                                          |
   | Homepage URL               | `http://localhost`                               |
   | Application description    | `URL shortening service written in Go and React` |
   | Authorization callback URL | `http://localhost/oauth/github/sign-in/callback` |

1. Replace the value of `GITHUB_CLIENT_ID` in the `.env` file with `Client ID`.
1. Replace the value of `GITHUB_CLIENT_SECRET` in the `.env` file with
   `Client Secret`.

### Generate static assets

Run the following commands at project root:

```bash
docker build -t frontend-build -f Dockerfile-build .
docker run -v $(pwd)/frontend/build:/app/build frontend-build:latest
```

### Build frontend & backend docker images

```bash
docker build -t short-frontend:latest -f frontend/Dockerfile frontend
docker build -t short-backend:latest -f backend/Dockerfile backend
```

### Launch App

```bash
docker-compose up
```

Visit [http://localhost:3000](http://localhost:3000)

## Development

### Dependencies

- Go v1.13.1
- Node.js v12.12.0
- Yarn v1.19.1
- Postgresql v12.0 ( or use [ElephantSQL](https://short-d.com/r/sql) instead )

### Backend

1. Create `.env` under `backend` directory with the following content:

   ```env
   DB_HOST=your_db_host
   DB_PORT=your_db_port
   DB_USER=your_db_user
   DB_PASSWORD=your_db_password
   DB_NAME=your_db_name
   RECAPTCHA_SECRET=your_recaptcha_secret
   GITHUB_CLIENT_ID=your_Github_client_id
   GITHUB_CLIENT_SECRET=your_Github_client_secret
   JWT_SECRET=your_JWT_secret
   WEB_FRONTEND_URL=http://localhost:3000
   ```
   
1. Update `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`,
   `RECAPTCHA_SECRET`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `JWT_SECRET`
   with your own configurations.

1. Launch backend server

   ```bash
   cd backend
   ./scripts/dev
   ```

1. Remember to install developers tools before start coding:

   ```bash
   ./scripts/tools
   ```

### Frontend

Remember to update `REACT_APP_RECAPTCHA_SITE_KEY` in `frontend/.env.development`.

1. Launch frontend server

   ```bash
   cd frontend
   ./scripts/dev
   ```
   
1. Visit [http://localhost:3000](http://localhost:3000)

## System Design

### App Level Architecture

Short backend is built on top of
[Uncle Bob's Clean Architecture](https://api.short-d.com/r/ca), the central
objective of which is separation of concerns.

![Clean Architecture](doc/eng/architecture/clean-architecture.jpg)

It enables the developers to modify a single component of the system at a time
while leaving the rest unchanged. This minizes the amount of changes have to
be made in order to support new requirements as the system grows. Clean
Architecture also improves the testability of system, which in turn saves
precious time when creating automated tests.

### Service Level Archtecture

Short adopts [Microservices Architecture](https://api.short-d.com/r/ms) to
organize dependent services around business capabilities and to enable
independent deployment of each service.

![Microservice Architecture](doc/eng/architecture/microservices.jpg)

### Dependency Management

Short leverages class design, package cohesion, and package coupling princiapls
from C++ world to manage its internal dependencies.

#### Class Design

| Principal                                                        | Description                                                            |
|------------------------------------------------------------------|------------------------------------------------------------------------|
| [Single Responsibility Principle](https://api.short-d.com/r/srp) | A class should have one, and only one, reason to change.               |
| [Open Closed Principle](https://api.short-d.com/r/ocp)           | You should be able to extend a classes behavior, without modifying it. |
| [Liskov Substitution Principle](https://api.short-d.com/r/lsp)   | Derived classes must be substitutable for their base classes.          |
| [Interface Segregation Principle](https://api.short-d.com/r/isp) | Make fine grained interfaces that are client specific.                 |
| [Dependency Inversion Principle](https://api.short-d.com/r/dip)  | Depend on abstractions, not on concretions.                            |

#### Package Cohesion

| Principal                                                            | Description                                           |
|----------------------------------------------------------------------|-------------------------------------------------------|
| [Release Reuse Equivalency Principle](https://api.short-d.com/r/rep) | The granule of reuse is the granule of release.       |
| [The Common Closure Principle](https://api.short-d.com/r/ccp)        | Classes that change together are packaged together.   |
| [The Common Reuse Principle](https://api.short-d.com/r/crp)          | Classes that are used together are packaged together. |

#### Package Coupling

| Principal                                                       | Description                                           |
|-----------------------------------------------------------------|-------------------------------------------------------|
| [Acyclic Dependencies Principle](https://api.short-d.com/r/adp) | The dependency graph of packages must have no cycles. |
| [Stable Dependencies Principle](https://api.short-d.com/r/sdp)  | Depend in the direction of stability.                 |
| [Stable Abstractions Principle](https://api.short-d.com/r/sap)  | Abstractness increases with stability.                |

### Dependency Injection

Short produces flexible and loosely coupled code, by explicitly providing
components with all of the dependencies they need.

```go
type Authenticator struct {
  tokenizer          fw.CryptoTokenizer
  timer              fw.Timer
  tokenValidDuration time.Duration
}

func NewAuthenticator(
  tokenizer fw.CryptoTokenizer,
  timer fw.Timer,
  tokenValidDuration time.Duration,
) Authenticator {
  return Authenticator{
    tokenizer:          tokenizer,
    timer:              timer,
    tokenValidDuration: tokenValidDuration,
  }
}
```

Short also simplifies the management of the big block of order-dependent
initialization code with [Wire](https://api.short-d.com/r/wire), a compile time
depedency injection framework by Google.

```go
func InjectGraphQlService(
  name string,
  sqlDB *sql.DB,
  graphqlPath provider.GraphQlPath,
  secret provider.ReCaptchaSecret,
  jwtSecret provider.JwtSecret,
  bufferSize provider.KeyGenBufferSize,
  kgsRPCConfig provider.KgsRPCConfig,
) (mdservice.Service, error) {
  wire.Build(
    wire.Bind(new(fw.GraphQlAPI), new(graphql.Short)),
    wire.Bind(new(url.Retriever), new(url.RetrieverPersist)),
    wire.Bind(new(url.Creator), new(url.CreatorPersist)),
    wire.Bind(new(repo.UserURLRelation), new(db.UserURLRelationSQL)),
    wire.Bind(new(repo.URL), new(*db.URLSql)),
    wire.Bind(new(keygen.KeyGenerator), new(keygen.Remote)),
    wire.Bind(new(service.KeyFetcher), new(kgs.RPC)),

    observabilitySet,
    authSet,

    mdservice.New,
    provider.NewGraphGophers,
    mdhttp.NewClient,
    mdrequest.NewHTTP,
    mdtimer.NewTimer,

    db.NewURLSql,
    db.NewUserURLRelationSQL,
    provider.NewRemote,
    url.NewRetrieverPersist,
    url.NewCreatorPersist,
    provider.NewKgsRPC,
    provider.NewReCaptchaService,
    requester.NewVerifier,
    graphql.NewShort,
  )
  return mdservice.Service{}, nil
}
```

### Database Modeling

![Entity Relation Diagram](doc/eng/db/er-v1.png)

## Deployment

Merging from `master` branch to `production` branch on Github will automatically
deploy the latest code to the production server. This is called continuous
delivery in the DevOps world.

![Continuous Delivery](doc/eng/devops/continuous-delivery.png)

In the future, when there are enough automated tests, we may migrate to
continuous deployment instead.

![Continuous Deployment](doc/eng/devops/continuous-deployment.png)

## Tools We Use

- [Drone](https://short-d.com/r/ci): Continuous integration
  written in Go
- [Code Climate](https://short-d.com/r/cs): Automated code
  review
- [ElephantSQL](https://www.elephantsql.com): Managed PostgreSQL service.

## Contributing

When contributing to this repository, please first discuss the change you wish
to make via [Slack channel](https://short-d.com/r/slack) with the owner
of this repository before making a change.

Please open a draft pull request when you are working on an issue so that the
owner knows it is in progress. The owner may take over or reassign the issue if no
body replies after ten days assigned to you.

### Pull Request Process

1. Update the README.md with details of changes to the interface, this includes
   new environment variables, exposed ports, useful file locations and container
   parameters.
1. You may merge the Pull Request in once you have the sign-off of code owner,
   or if you do not have permission to do that, you may request the code owner
   to merge it for you.

### Code of Conduct

- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

### Discussions

Please join this [Slack channel](https://short-d.com/r/slack) to
discuss bugs, dev environment setup, tooling, and coding best practices.

## Author

Harry Liu - *Initial work* - [byliuyang](https://short-d.com/r/ghharry)

## License

This project is maintained under MIT license
