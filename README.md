# SPM Jaeger example

This repository is a simple configuration example to test Jaeger with the SPM (Service Performance Monitoring) feature enabled. It is basically a copy of the monitor example in the [jaeger repository](https://github.com/jaegertracing/jaeger/tree/main/docker-compose/monitor).

## How to run

1. Clone this repository
2. Run `docker-compose up -d`
3. Open the Jaeger UI at http://localhost:16686

You should see the Jaeger UI with no data. To start generating traces and metrics, you will need to connect an instrumented application to the otel-collector. The collector is running and configured to receive data using the opentelemetry GRPC exporter on port 4317.

Example of a simple node application that can be used to generate traces and metrics:
    
```ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-grpc';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-grpc';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { diag, DiagConsoleLogger, DiagLogLevel } from '@opentelemetry/api';
import express from 'express';

diag.setLogger(new DiagConsoleLogger(), DiagLogLevel.INFO);

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'your-service-name',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0',
  }),
  traceExporter: new OTLPTraceExporter({
    url: 'http://localhost:4317',
  }),
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({
      url: 'http://localhost:4317',
    }),
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

const app = express();

app.get('/ping', (req, res) => {
  res.send('Pong!');
});

app.listen(3000, () => {
  console.log('Example app listening on port 3000!');
});
```

The above code should generate traces when you access the `/ping` endpoint.
