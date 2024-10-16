# Week 2 — Distributed Tracing


## HoneyComb
When creating a new dataset in Honeycomb it will provide all these **installation instructions** but will outline these here anyway:

You'll need to grab the API key from your honeycomb account and export them like so (gp is for gitpod so that it will be available again when relaunching):

```sh
export HONEYCOMB_API_KEY=""
gp env HONEYCOMB_API_KEY=""
```

Add the following Env Vars to `backend-flask` in docker compose file

```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "backend-flask"
OTEL_EXPORTER_OTLP_PROTOCOL: "http/protobuf"
```

We'll add the following libraries to our `requirements.txt`

```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

We'll install these dependencies:

```sh
pip install -r requirements.txt
```

Add the following code blocks to `app.py`

```py
# Honeycomb ------------------
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```


```py
# Honeycomb ------------------
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

```py
# Honeycomb ------------------
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

Add the following to your `.gitpod.yml` so you can open these ports by default:
```
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```

## X-Ray

### Instrument AWS X-Ray for Flask

Add to the `requirements.txt`

```py
aws-xray-sdk
```

Install python dpendencies

```sh
pip install -r requirements.txt
```

Add to `app.py`

```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)


# X-Ray move this under the "app" definition --------
XRayMiddleware(app, xray_recorder)
```

### Setup AWS X-Ray Resources

Create `aws/json/xray.json` and add:

```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

Then run the command to create an xray group:

```sh
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```
This should create a group found at CloudWatch -> Settings -> Groups. It will group traces together by filter expression. We don't need to do this but helpful when you have several services.

Then create a sampling rule using the JSON we created earlier (there is a default one, but this one will save us cost):
```sh
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
The created sampling rule can be found in Cloudwatch -> Settings -> X-Ray traces -> Sampling rules


### Add Daemon Service to Docker Compose

```yml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

We need to add these two env vars to our backend-flask in our `docker-compose.yml` file

```yml
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```


### Check for traces
- Run docker-compose up to run all the services
- Check the logs for the daemon and the backend flask for any errors
- If all is good, go to a random API endpoint like this one: https://4567-hngu-awsbootcampcrudd-6b60d74yf2o.ws-us115.gitpod.io/api/activities/home
- Check the logs again to make sure data is sent
- Then go to AWS and check in X-Ray -> Traces for sample traces

### Check service data for last 10 minutes

```sh
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```


## CloudWatch Logs


Add to the `requirements.txt`

```
watchtower
```

```sh
pip install -r requirements.txt
```


In `app.py`

```
# CloudWatch Logs ------
import watchtower
import logging
from time import strftime
```

```py
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
```

```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

We'll log something in an API endpoint. You can pass in `LOGGER` via function call.
```py
LOGGER.info('Hello Cloudwatch! from  /api/activities/home')
```

Set the env var in your backend-flask for `docker-compose.yml`

```yml
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

> passing AWS_REGION doesn't seems to get picked up by boto3 so pass default region instead


## Rollbar

https://rollbar.com/

Create a new project in Rollbar called `Cruddur`

Add to `requirements.txt`


```
blinker
rollbar
```

We need to set our access token (get the token from Rollbar)

```sh
export ROLLBAR_ACCESS_TOKEN="[ROLLBAR_TOKEN]"
gp env ROLLBAR_ACCESS_TOKEN="[ROLLBAR_TOKEN]"
```

Add to backend-flask for `docker-compose.yml`

```yml
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```

Import for Rollbar in app.py

```py
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```

```py
# Rollbar add after app init ----- 
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```

We'll add an endpoint just for testing rollbar to `app.py`

```py
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```

[Rollbar Flask Example](https://github.com/rollbar/rollbar-flask-example/blob/master/hello.py)


## [Note] Changes to Rollbar

During the original bootcamp cohort, there was a newer version of flask.
This resulted in rollback implementation breaking due to a change is the flask api.

If you notice rollbar is not tracking, utilize the code from this file:

https://github.com/omenking/aws-bootcamp-cruddur-2023/blob/week-x/backend-flask/lib/rollbar.py

# My Notes
## Distributed Tracing
- A trace, in the context of the BE contains information about a request and what happened to that request as it goes through your backend. A distributed trace can track how the request was handled as it goes through many BE services and datastores.
- A trace contains one or more spans, where a span contains a start and end timestamp and how long that span took to finish. It also can contain what service handled that span.
- Honeycomb has a paid service called a service map that can visualize what services talk to all other services. This is only if you have a lot of services so no need to have this in this small app.
- Intrumentation is the code that you install that produces this trace information
- Amazon X-Ray is very hard to integrate so we choose Honeycomb
- When you 

## Honeycomb
- when you first login, the app creates a default env called test and each API key is scoped to an environment
- when you export a variable like `export TEST=1` the export command makes it available in the current shell, and any subshells. If you don't then it will only be available in the current shell
- Once you have setup your API key, we need to set the `OTEL_` environment variables (Open Telemetry)
- Open telemetry is an open source standard for specifying the schema for sending instrumentation. The standard was created so that one can migrate from one observability service to another
- The `${}` will take environment variables from the shell where `docker-compose up` is run

## Observability
- 3 pillars: metrics, traces, logs

## AWS X-Ray
- Segments/subsegments are just spans

## AWS Cognito
- User pool: for creating authentication for users of your applications (sign in, sign up, manage user data)
- Idenitity pool: for managing authorization/access to AWS services or resources

### Application best practices
- use industry standard authentication and authorization (SAML, OpenID, OAuth 2.0, etc)
- User management: create, modify, delete users
- User access management: add, modify, revoke roles
- Role based access
- encryption of API calls
- token lifecycle management
