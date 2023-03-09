# Week 2 — Distributed Tracing

During Jessica Kerr’s session:

Distributed Tracing is a method used in software development to understand how a request is processed in a distributed system. It involves instrumenting code to track the request's path through the system, enabling developers to identify issues like bottlenecks and errors. This helps improve system performance and reliability, and is a critical tool in modern DevOps practices. 

Instrumenting Honeycomb with Open Telemetry (OTEL)
Honeycomb is a cloud-based observability platform that helps software engineers debug and understand their applications in real-time. It provides high-resolution tracing, profiling, and monitoring capabilities that allow developers to quickly identify and resolve performance issues, errors, and other problems in their software applications.
Honeycomb's main features include:
1.	Tracing: Honeycomb allows users to trace every request through a system in real-time, capturing all of the data about the request and the system's response.
2.	Visualization: Honeycomb provides a range of visualizations, including heatmaps, histograms, and scatter plots, that allow users to easily analyze and understand their data. We implemented this during the livestream session.
3.	Collaboration: Honeycomb allows teams to collaborate on debugging and troubleshooting by sharing data and insights across the organization.
4.	Integrations: Honeycomb integrates with a wide range of tools and platforms, including Kubernetes, AWS, and GitHub, to provide a seamless experience for developers.

### Adding OTEL to the backend-flask


When creating a new dataset in Honeycomb it will provide all these installation insturctions


In my `requirements.txt` file, I added the following packages to instrument the app with OTEL.
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

Then I installed the packages:

```sh
pip install -r requirements.txt
```

In the `app.py` file, I created and initialized tracer and flask instrumentation to send data to Honeycomb.

```py
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```


```py
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

```py
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
Then I created spans to describe what is happening in your application. For example, this could be a HTTP handler, a long running operation, or a database fetch. Spans are created in a parent-child pattern, so each time you create a new span, the current active span is used as its parent.
```py
with tracer.start_as_current_span("home-activities-mock-data"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())
```

I added the following environment variables to `backend-flask` in my docker compose file.

```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
