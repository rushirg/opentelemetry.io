---
title: Node.JS
aliases: [/docs/js/getting_started/nodejs]
---

This guide will show you how to get started with tracing in Node.js.

- [Example Application](#example-application)
  - [Dependencies](#dependencies)
  - [Code](#code)
- [Tracing](#tracing)
  - [Dependencies](#dependencies-1)
    - [Core Dependencies](#core-dependencies)
    - [Exporter](#exporter)
    - [Instrumentation Modules](#instrumentation-modules)
  - [Setup](#setup)
  - [Run Application](#run-application)
- [Metrics](#metrics)
  - [Dependencies](#dependencies-2)
    - [Core Dependencies](#core-dependencies-1)
    - [Exporter](#exporter-1)
  - [Setup](#setup-1)
  - [Run Application](#run-application-1)

## Example Application

This is a small example application we will monitor in this guide.

### Dependencies

Install dependencies used by the example.

```sh
npm install express
```

### Code

Please save the following code as `app.js`.

```javascript
/* app.js */

const express = require("express");

const PORT = process.env.PORT || "8080";
const app = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(parseInt(PORT, 10), () => {
  console.log(`Listening for requests on http://localhost:${PORT}`);
});
```

Run the application with the following request and open <http://localhost:8080> in your web browser to ensure it is working.

```sh
$ node app.js
Listening for requests on http://localhost:8080
```

## Tracing

### Dependencies

The following dependencies are required to trace a Node.js application.

#### Core Dependencies

These dependencies are required to configure the tracing SDK and create spans.

```shell
npm install @opentelemetry/sdk-node @opentelemetry/api
```

#### Exporter

In the following example, we will use the `ConsoleSpanExporter` which prints all spans to the console.

In order to visualize and analyze your traces, you will need to export them to a tracing backend.
Follow [these instructions](../../exporters) for setting up a backend and exporter.

You may also want to use the `BatchSpanProcessor` to export spans in batches in order to more efficiently use resources.

#### Instrumentation Modules

Many common modules such as the `http` standard library module, `express`, and others can be automatically instrumented using autoinstrumentation modules. To find autoinstrumenatation modules, you can look at the [registry](https://opentelemetry.io/registry/?language=js&component=instrumentation#).

You can also install all instrumentations maintained by the OpenTelemetry authors by using the `@opentelemetry/auto-instrumentations-node` module.

```shell
npm install @opentelemetry/auto-instrumentations-node
```

### Setup

The tracing setup and configuration should be run before your application code. One tool commonly used for this task is the [`-r, --require module`](https://nodejs.org/api/cli.html#cli_r_require_module) flag.

Create a file with a name like `tracing.js` which will contain your tracing setup code.

```javascript
/* tracing.js */

// Require dependencies
const opentelemetry = require("@opentelemetry/sdk-node");
const { getNodeAutoInstrumentations } = require("@opentelemetry/auto-instrumentations-node");

const sdk = new opentelemetry.NodeSDK({
  traceExporter: new opentelemetry.tracing.ConsoleSpanExporter(),
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk.start()
```

### Run Application

Now you can run your application as you normally would, but you can use the `--require` flag to load the tracing code before the application code.

```shell
$ node --require './tracing.js' app.js
Listening for requests on http://localhost:8080
```

Open <http://localhost:8080> in your web browser and reload the page a few times, after a while you should see the spans printed in the console by the `ConsoleSpanExporter`.

<details>
<summary>View example output</summary>

```json
{
  "traceId": "3f1fe6256ea46d19ec3ca97b3409ad6d",
  "parentId": "f0b7b340dd6e08a7",
  "name": "middleware - query",
  "id": "41a27f331c7bfed3",
  "kind": 0,
  "timestamp": 1624982589722992,
  "duration": 417,
  "attributes": {
    "http.route": "/",
    "express.name": "query",
    "express.type": "middleware"
  },
  "status": { "code": 0 },
  "events": []
}
{
  "traceId": "3f1fe6256ea46d19ec3ca97b3409ad6d",
  "parentId": "f0b7b340dd6e08a7",
  "name": "middleware - expressInit",
  "id": "e0ed537a699f652a",
  "kind": 0,
  "timestamp": 1624982589725778,
  "duration": 673,
  "attributes": {
    "http.route": "/",
    "express.name": "expressInit",
    "express.type": "middleware"
  },
  "status": { code: 0 },
  "events": []
}
{
  "traceId": "3f1fe6256ea46d19ec3ca97b3409ad6d",
  "parentId": "f0b7b340dd6e08a7",
  "name": "request handler - /",
  "id": "8614a81e1847b7ef",
  "kind": 0,
  "timestamp": 1624982589726941,
  "duration": 21,
  "attributes": {
    "http.route": "/",
    "express.name": "/",
    "express.type": "request_handler"
  },
  "status": { code: 0 },
  "events": []
}
{
  "traceId": "3f1fe6256ea46d19ec3ca97b3409ad6d",
  "parentId": undefined,
  "name": "GET /",
  "id": "f0b7b340dd6e08a7",
  "kind": 1,
  "timestamp": 1624982589720260,
  "duration": 11380,
  "attributes": {
    "http.url": "http://localhost:8080/",
    "http.host": "localhost:8080",
    "net.host.name": "localhost",
    "http.method": "GET",
    "http.route": "",
    "http.target": "/",
    "http.user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36",
    "http.flavor": "1.1",
    "net.transport": "ip_tcp",
    "net.host.ip": "::1",
    "net.host.port": 8080,
    "net.peer.ip": "::1",
    "net.peer.port": 61520,
    "http.status_code": 304,
    "http.status_text": "NOT MODIFIED"
  },
  "status": { "code": 1 },
  "events": []
}
```

</details>

## Metrics

### Dependencies

The following dependencies are required to collect metrics in your Node.js application.

#### Core Dependencies

These dependencies are required to configure the tracing SDK and create spans.

- `@opentelemetry/metrics`

#### Exporter

In the following example, we will use the `ConsoleMetricExporter` which prints all spans to the console.

In order to visualize and analyze your metrics, you will need to export them to a metrics backend.
Follow [these instructions](../../exporters) for setting up a backend and exporter.

### Setup

You need a `Meter` to create and monitor metrics. A `Meter` in OpenTelemetry is the mechanism used to create and manage metrics, labels, and metric exporters.

Create a file named `monitoring.js` and add the following code:

```javascript
/* monitoring.js */
'use strict';

const { MeterProvider, ConsoleMetricExporter } = require('@opentelemetry/metrics');

const meter = new MeterProvider({
  exporter: new ConsoleMetricExporter(),
  interval: 1000,
}).getMeter('your-meter-name');
```

Now you can require this file from your application code and use the `Meter` to create and manage metrics. The simplest of these metrics is a counter.

Let's create and export from your `monitoring.js` file a middleware function that express can use to count all requests by route. Modify the `monitoring.js` file so it looks like this:

```javascript
/* monitoring.js */
'use strict';

const { MeterProvider, ConsoleMetricExporter } = require('@opentelemetry/metrics');

const meter = new MeterProvider({
  exporter: new ConsoleMetricExporter(),
  interval: 1000,
}).getMeter('your-meter-name');

const requestCount = meter.createCounter("requests", {
  description: "Count all incoming requests"
});

const boundInstruments = new Map();

module.exports.countAllRequests = () => {
  return (req, res, next) => {
    if (!boundInstruments.has(req.path)) {
      const labels = { route: req.path };
      const boundCounter = requestCount.bind(labels);
      boundInstruments.set(req.path, boundCounter);
    }

    boundInstruments.get(req.path).add(1);
    next();
  };
};
```

Now import and use this middleware in your application code `app.js`:

```javascript
/* app.js */
const express = require("express");
const { countAllRequests } = require("./monitoring");
const app = express();
app.use(countAllRequests());
/* ... */
```

Now when you make requests to your service, your meter will count all requests.

**Note**: Creating a new labelSet and binding on every request is not ideal because creating the labelSet can often be an expensive operation. Therefore, the instruments are created and stored in a Map according to the route key.

### Run Application

First, install the dependencies as described above. Here you need to add the following:

```shell
npm install --save @opentelemetry/metrics
```

Now you can run your application:

```shell
$ node app.js
Listening for requests on http://localhost:8080
```

Now, when you open <http://localhost:8080> in your web browser, you should see the metrics printed in the console by the `ConsoleMetricExporter`.

```json
{
  "name": "requests",
  "description": "Count all incoming requests",
  "unit": "1",
  "metricKind": 0,
  "valueType": 1
}
{ "route": "/" }
"value": "1"
```
