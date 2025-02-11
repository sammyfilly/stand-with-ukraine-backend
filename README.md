<p align="center">
  <a href="https://www.bigcommerce.com/apps/stand-with-ukraine/">
    <picture aria-label="Stand with Ukraine App logo">
      <img src="https://s3.amazonaws.com/integrated-apps/nbxgitdu/mvcrlqjk.png" height="128">
    </picture>
    <h1 align="center">Stand with Ukraine App</h1>
    <h3 align="center">Backend Components</h3>
  </a>
</p>

<p align="center">
  <a aria-label="Coverage" href="https://coveralls.io/github/bigcommerce/stand-with-ukraine-backend?branch=main">
    <img src="https://img.shields.io/coveralls/github/bigcommerce/stand-with-ukraine-backend/main?style=for-the-badge&labelColor=005BBB">
  </a>
  <a aria-label="Build Status for Server" href="https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/server.yaml">
    <img alt="" src="https://img.shields.io/github/actions/workflow/status/bigcommerce/stand-with-ukraine-backend/server.yaml?branch=main&label=build%3Aserver&style=for-the-badge&labelColor=005BBB&logo=Rust">
  </a>
  <a aria-label="Build Status for Exporter" href="https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/exporter.yaml">
    <img alt="" src="https://img.shields.io/github/actions/workflow/status/bigcommerce/stand-with-ukraine-backend/exporter.yaml?branch=main&label=build%3Aexporter&style=for-the-badge&labelColor=005BBB&logo=Rust">
  </a>
  <a aria-label="License" href="https://github.com/bigcommerce/stand-with-ukraine-backend/blob/main/LICENSE.md">
    <img alt="" src="https://img.shields.io/github/license/bigcommerce/stand-with-ukraine-backend?style=for-the-badge&labelColor=005BBB">
  </a>
  <hr/>
</p>

## Overview

[Stand With Ukraine is a BigCommerce application][app_store_link]. It allows merchants to easily add a widget to their storefront with a customized message and list of Charities that the merchant would like their shoppers to visit and support.

## Repositories

- [Frontend Components][frontend_repo]
- [Backend Components][backend_repo]

## Getting Started

- `apps/server` is a rust http server that implements the OAuth callback flow for BigCommerce [Single Click Apps][single_click_apps] and uses the [BigCommerce Management APIs][bigcommerce_api_docs] to inject the widget using the [Script API][script_api]. For information on how to develop apps for BigCommerce stores, see our [Developer Portal][dev_portal].
- `apps/exporter` is an program that exports data from the database and updates a google spreadsheet with the install status and widget removal feedback. It is scheduled to run every hour and allows the product/business teams to access this data without needing database access.

We hope this sample gives you a good reference point for building your next killer app!

### Registering the app with BigCommerce

1. Create a trial store on [BigCommerce][bigcommerce]
2. Go to the [Developer Portal][dev_portal] and log in by going to "My Apps"
3. Click the button "Create an app", enter a name for the new app, and then click "Create"
4. You don't have to fill out all the details for your app right away, but you do need
   to provide some core details in section 4 (Technical). Note that if you are just getting started, you can use `localhost` for your hostname, but ultimately you'll need to host your app on the public Internet.

   - _Auth Callback URL_: `https://<app hostname>/bigcommerce/install`
   - _Load Callback URL_: `https://<app hostname>/bigcommerce/load`
   - _Uninstall Callback URL_: `https://<app hostname>/bigcommerce/uninstall`

5. Enable the following under _OAuth scopes_ which this sample app needs:
   - **Store Information - Read Only** scope is needed to get the store url for providing a preview url
   - **Store Content - Modify** scope is needed to inject the script on the storefront
6. Click `Save & Close` on the top right of the dialog.
7. You'll now see your app in a list in the _My Apps_ section of Developer Portal. Hover over it and click _View Client ID_. You'll need these values in the next step.

### Local Development

- Prerequisites
  - Rust toolchain
    - Recommend using `rustup` to setup `rust`, `cargo`, `fmt`
  - SQLX command
    - Recommend setup using `cargo install sqlx-cli --force --version=0.7.1 --features=postgres,rustls --no-default-features`
  - Docker
    - Recommended setup for `macos` or `linux` is `podman` and creating an alias for docker from podman
  - Editor
    - Recommended setup is `vscode` and the `rust-analyzer` extension.
  - For parsing logging `bunyan` command is helpful.
    - Recommended setup is `cargo install bunyan`
    - Enable log during testing and pass it through bunyan `TEST_LOG=true cargo test | bunyan`

1. Clone this repo: `git clone git@github.com:bigcommerce/stand-with-ukraine-backend.git`
2. Change to the repo directory: `cd stand-with-ukraine-backend`
3. Install dependencies with pip: `cargo install`
4. Copy `.env-sample` to `.env`
5. Edit `.env` or `apps/exporter/configuration/base.yaml` / `apps/server/configuration/base.yaml`
   - Set the following to the values obtained from Developer Portal for your app.
     - `bigcommerce.client_id`
     - `bigcommerce.client_secret`
   - Set `application.jwt_secret` to a long random string, such as that generated by your password manager or [random.org](https://random.org).
6. Start the database container `CREATE_LOCAL_DB=TRUE ./scripts/init_db.sh`
7. Run tests: `cargo test`
8. Run app: `cargo run --bin swu-app`

### Hosting the app

In order to install this app in a BigCommerce store, it must be hosted on the public Internet. You can get started in development by simply running `cargo run --bin swu-app` to run it locally, and then use `localhost` in your URLs. Ultimately you will need to host it somewhere to use the app anywhere other than your development system.

### Deploying to Production

This repo is setup to be deployed to Google cloud run using a Docker container. Github Actions tests and builds a docker image if everything passes. The action is also responsible for pushing the image to google cloud artifacts and also running the update deployment command to target the new image.

To configure the production server variables you can use environment variables for the container to override the configuration in the file `apps/server/configuration/base.yaml`
`apps/exporter/configuration/base.yaml`.

For example, to configure of the app hostname you can do one of the following:

1. Edit `application.base_url` in `apps/server/configuration/base.yaml`
   `apps/exporter/configuration/base.yaml`.
2. Set `APP__APPLICATION__BASE_URL` using environment variables from the container platform. Environment variables will override the file configuration.

### Installing the app in your trial store

- Login to your trial store
- Go to the Marketplace and click _My Drafts_. Find the app you just created and click it.
- A details dialog will open. Click _Install_ and the draft app will be installed in your store.

### Architecture - API/Routes

- BigCommerce OAuth Routes. They are responsible for handling the install, load and uninstall requests from a BigCommerce Store
  - `/bigcommerce/install`
  - `/bigcommerce/load`
  - `/bigcommerce/uninstall`
- API Routes
  - `/api/v1/publish`
    - `POST` publish widget to storefront
    - `DELETE` remove widget from storefront
  - `/api/v1/preview`
    - `GET` retrieve the store url for previewing the widget
  - `/api/v1/configuration`
    - `POST` set the configuration of the widget
    - `GET` get the current configuration of the widget
  - `/api/v2/widget-event`
    - `POST` saves a widget event for analytics purposes
  - `/api/v2/charity-event`
    - `POST` saves a charity visited event for analytics purposes

## License

Copyright (c) 2017-present, BigCommerce Pty. Ltd. All rights reserved

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
documentation files (the "Software"), to deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[app_store_link]: https://www.bigcommerce.com/apps/stand-with-ukraine/ "BigCommerce App Store - Stand with Ukraine"
[bigcommerce]: https://www.bigcommerce.com/
[single_click_apps]: https://developer.bigcommerce.com/api/#building-oauth-apps
[bigcommerce_api_docs]: https://developer.bigcommerce.com/docs
[dev_portal]: https://developer.bigcommerce.com
[script_api]: https://developer.bigcommerce.com/docs/ZG9jOjIyMDYyMg-scripts-api
[server_action]: https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/server.yaml
[server_action_badge]: https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/server.yaml/badge.svg
[exporter_action]: https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/exporter.yaml
[exporter_action_badge]: https://github.com/bigcommerce/stand-with-ukraine-backend/actions/workflows/exporter.yaml/badge.svg
[coverage_badge]: https://coveralls.io/repos/github/bigcommerce/stand-with-ukraine-backend/badge.svg?branch=main
[coverage]: https://coveralls.io/github/bigcommerce/stand-with-ukraine-backend?branch=main
[frontend_repo]: https://github.com/bigcommerce/stand-with-ukraine-frontend
[backend_repo]: https://github.com/bigcommerce/stand-with-ukraine-backend
