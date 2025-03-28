# XCMetrics
This document is to help setup [XCMetrics](https://xcmetrics.io/) on a local machine

## Server

### Install Prerequisites
* Install [docker](https://www.docker.com)

### Server

Follow the steps [here](https://github.com/spotify/XCMetrics/blob/main/docs/Run%20the%20Backend%20Locally.md) to run the server locally.

* Download `docker-compose-local.yml`
* Run: `docker-compose -f ~/Downloads/XCMetrics/docker-compose-local.yml up` _(update the path to the file in my case `docker-compose-local.yml` is in `~/Downloads/XCMetrics/`)_
* Validate server setup: `curl -I http://localhost:8080/hello`
    * Expected response:
        ```
        HTTP/1.1 200 OK
        content-type: text/plain; charset=utf-8
        content-length: 13
        connection: keep-alive
        date: Fri, 14 Jan 2022 00:22:44 GMT
        ```
    * Had to update docker resources Swap to 2 GB and Memory to 4 GB.

### Client

Follow the steps [here](https://github.com/spotify/XCMetrics/blob/main/docs/Getting%20Started.md).

* Download `XCMetrics-macOS` from [XCMetrics releases](https://github.com/spotify/XCMetrics/releases)
* Add the following to Your_iOS_APP by going to Edit scheme > Build > Post-actions and add:
    _Update the path in my case `XCMetricsLauncher` and `XCMetrics` are in `~/Downloads/XCMetrics/`_
    ```
    #Capture console logs (Post-actions are not logged)
    exec > ~/Downloads/XCMetrics/error.txt 2>&1
    echo "Starting XCMetrics collection"
    ~/Downloads/XCMetrics/XCMetricsLauncher ~/Downloads/XCMetrics/XCMetrics --name "Your_iOS_APP" --buildDir ${BUILD_DIR} --serviceURL http://localhost:8080/v1/metrics
    echo "Done XCMetrics collection"
    ```

Make sure to select `Your_iOS_APP` target in provide build setting for section.

#### Additional flag added:
* https://github.com/MobileNativeFoundation/XCLogParser#parse-command
* https://www.onswiftwings.com/posts/build-time-optimization-part1/ (section `Compiler diagnostic options`)

### GUI

Follow the steps [here](https://xcmetrics.io/docs/backstage-integration.html).

* Create backstage app steps [here](https://backstage.io/docs/getting-started/)
    * `mkdir backstage` in `~/Downloads/XCMetrics`
    * `cd ~/Downloads/XCMetrics/backstage`
    * Run: `npx @backstage/create-app`
        * app name: `xcmetrics-app`
        * for backend select: `PostgreSQL`
           * If option to select `PostgreSQL` is not given do the following:
              * `yarn add pg`
              * Add the following:
                 ```yaml
                 database:
                 ...
                     client: pg
                 ```
    * Add backstage xcmetrics plugin: 
        * `cd xcmetrics-app`
        * `yarn add @backstage/plugin-xcmetrics` (might ask you to run with `-W` option)
    * Update the config `packages/app/src/App.tsx`:
    * Add the following:
        ```ts
        import { XcmetricsPage } from '@backstage/plugin-xcmetrics';
        ```
    * Update the config `app-config.yaml`: 
    * Add the following:
        ```yaml
        proxy:
        ...
        '/xcmetrics':
            target: http://127.0.0.1:8080/v1
        ```
    * Add the following:
        ```yaml
          database:
          client: pg
          connection:
        ...
            host: localhost
            port: 5432
            user: xcmetrics-dev
            password: xcmetrics-dev
        ```
    * Run the app: `cd xcmetrics-app && yarn dev`
    * Goto http://localhost:3000/xcmetrics
        * Run Xcode build to see new entry
