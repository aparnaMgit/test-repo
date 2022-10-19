# mfd-shell
___
A micro-frontend (a.k.a micro-app) representing the main entry point
of Cloudbeds' micro-frontend architecture, which is based on Webpack Module Federation (WMF).

As the first app loaded, it serves as the base container establishing
which WMF remotes can be consumed in our micro-frontend.

## Overview

A React application written in Typescript responsible for:

- Loading remote entry descriptor files representing which remote micro-apps can be consumed
- Integrating _mfd-common_ in the global `window` namespace
- Authenticating the current user
  (not implemented yet, currently it happens in
  [mfd-core](https://bitbucket.org/cloudbeds/myfrontdesk-front/src/master/src/modules/TwoFactorAuth/))

## Getting started

#### Important Note:

- In most development scenarios it is not necessary to run the _mfd-shell_.
  Each micro-app should be able to run separately in isolation.

- As this micro-app represents the base container of our micro-frontend, it will only
  show an error page at this point in _localhost_ scenario when other micro-apps
  that it consumes are not also running in a locally hosted capacity.

### Requirements

- [node.js](https://nodejs.org) ^16 (LTS)
- npm ^8

### Installation

Install dependencies for the _mfd-shell_ micro-app and for the _mock server_,
that is an [Express](https://expressjs.com/) app for local web development:

```sh
npm i
cd mockServer
npm i
```


### Configuration

Please read [this document in Confluence](https://cloudbeds.atlassian.net/wiki/spaces/AD/pages/2347237424/MFD+FE+Environment+Configuration)
to get some background information on how _.env_ files are treated
and what is the global configuration that myfrontdesk have.

The `.env.defaults` is used in Webpack configuration (`webpack/`)
as well as in the application source code (`src/`).

This configuration should serve for most development purposes, no modifications
or creation of a separate `.env` file is necessary. If you find it necessary
to overwrite this file be kind to other developers and create your own `.env` file,
that is listed in `.gitignore`


#### Variables:

- `DEPLOYMENT_ENV` - deployment environment (`dev`, `stage`, `prod`)
- `REMOTE_ENTRIES_URL` - targets base URL for remote entry JSON files.
  By default it targets your _mockServer_'s API URI to fetch `.json` files
  that define the current location of WMF remotes (a.k.a. remote entry config files).
  Thus, it is necessary to run the _mockServer_ alongside the _mfd-shell_.

#### Variables for `development` mode only:

- set custom `devtool` option for source maps
  (see [webpack devtool](https://webpack.js.org/configuration/devtool/) page for possible values)
- specific to webpack DevServer:
  - add `devServerHttps=false` to change server mode to HTTP
  - set custom `devServerHost` address if necessary
  - set custom `devServerPort` in case if the default port `9081` is not available or if you need to run multiple dev
    servers simultaneously

## Development

### Scripts

The `scripts` node in `src/package.json` contains common commands for running various
services after installation.

| Name               | Description                                                                                                              |
|--------------------|--------------------------------------------------------------------------------------------------------------------------|
|                    | **FOR PRODUCTION**                                                                                                       |
| `build`            | create a microfrontend bundle with [webpack](https://webpack.js.org/concepts/)                                           |
|                    | **FOR DEVELOPMENT**                                                                                                      |
| `start`            | continuously serve `src/index.ts` entry point with [webpack DevServer](https://webpack.js.org/configuration/dev-server/) |
| `start:live`       | serve bundle with Live Reload functionality                                                                              |
| `start:mockServer` | start an [Express](https://expressjs.com/) mock server                                                                   |
| `make-types`       | create types for consumption in other microfrontends                                                                     |
|                    | **CODE QUALITY**                                                                                                         |
| `lint`             | perform linting using [ESLint](https://eslint.org/)                                                                      |
| `lint:ts`          | perform type checking using [tsc](https://www.typescriptlang.org/tsconfig/#noEmit)                                       |
|                    | **UNIT TESTS**                                                                                                                  |
| `test`             | run tests using [Jest](https://jestjs.io/)                                                                               |

### Steps to serve locally built micro-apps:

1. Create `remotes/dev` folder (it is in `.gitignore`)

2. Create `.json` files and point URLs to your locally served applications.
   (Look into files in `dev-example` folder for an example.)
   - `mfd-common-remote-entry.json` - contains URL to the _mfd-common_ micro-app's remote entry point.
   - `remote-entries.json` - contains remote entry URLs of every other micro-app

    **Note:** There's no need to specify all micro-app URLs. By default, they are falling back
    to the URLs in `dev-ga` folder that points to the Dev build of microfrontends
    with latest changes in their `main`/`master` branches.

3. Start the **mockServer**
   ```sh
   npm run start:mockServer
   ```
   If you open http://localhost:4480 in your browser, the message
   _"Hello from an Express mock server."_  should appear.


5. Start the **mfd-shell** micro-app's dev server
   ```sh
   npm start
   ```
