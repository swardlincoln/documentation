---
title: Google Cloud Run
kind: documentation
further_reading:

- link: 'https://www.datadoghq.com/blog/collect-traces-logs-from-cloud-run-with-datadog/'
  tag: 'Blog'
  text: 'Collect traces, logs, and custom metrics from GCR services'

---

## Overview

Google Cloud Run is a fully managed serverless platform for deploying and scaling container-based applications. Datadog
provides monitoring and log collection for Cloud Run through the [GCP integration][1]. Datadog also provides a solution,
now in public beta, for instrumenting your Cloud Run applications with a purpose-built Agent to enable tracing, custom
metrics, and direct log collection.

  <div class="alert alert-warning">This feature is in public beta. You can provide feedback through a <a href="https://forms.gle/HSiDGnTPvDvbzDAQA">feedback form</a>, or through your standard support channels. During the beta period, Cloud Run monitoring and APM tracing are available without a direct cost. Existing APM customers may incur increased span ingestion and volume costs. </div>

## Getting started

### Prerequisites

Make sure you have a [Datadog API Key][10] and are using a programming language [supported by a Datadog tracing library][2].

### Deploy a sample application in one click

To deploy a sample application without the need to follow the rest of the guide, you can use on of [these examples][12]. The installation process will ask the required details such as the Datadog API Key. Note that these applications send data to the default website `datadoghq.com`.

### Instrument your application

To build your container with Datadog instrumentation you can follow one of these two methods, depending if you are using a Dockerfile or buildpack.

#### Instrument using Dockerfile

You can instrument your application by adding the following lines to your Dockerfile. You may need to adjust these examples depending on your existing Dockerfile setup.

{{< programming-lang-wrapper langs="go,python,nodejs,java,dotnet,ruby" >}}
{{< programming-lang lang="go" >}}

```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-go
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["/path/to/your-go-binary"]
```

[Sample code for a simple Go application](https://github.com/DataDog/crpb/tree/main/go).

{{< /programming-lang >}}
{{< programming-lang lang="python" >}}

```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# install the python tracing library here or in requirements.txt
RUN pip install --no-cache-dir ddtrace==1.7.3

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-python
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# execute your binary application wrapped in the entrypoint, launched by the Datadog trace library. Adapt this line to your needs
CMD ["ddtrace-run", "python", "app.py"]
```

[Sample code for a simple Python application](https://github.com/DataDog/crpb/tree/main/python).

[1]: /tracing/trace_collection/dd_libraries/python/?tab=containers#instrument-your-application

{{< /programming-lang >}}
{{< programming-lang lang="nodejs" >}}

1. Use the `COPY` instruction to copy the [Datadog `serverless-init` binary][3] into your Docker image.

2. Add the environment variable `--require dd-trace/init`

3. Use the `ENTRYPOINT` instruction to run the `serverless-init` binary as your Docker container is initiated.

4. Use the `CMD` instruction to run your existing application and other required commands as arguments.

You can accomplish those steps by adding the following lines to your Dockerfile. You may need to adjust these examples depending on your existing Dockerfile setup.
```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# install the Datadog js tracing library, either here or in package.json

npm i dd-trace@2.2.0

# enable the Datadog tracing library
ENV NODE_OPTIONS="--require dd-trace/init"

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-python
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["/nodejs/bin/node", "/path/to/your/app.js"]

```

[Sample code for a simple Node.js application](https://github.com/DataDog/crpb/tree/main/js).

{{< /programming-lang >}}
{{< programming-lang lang="java" >}}

```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-python
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["./mvnw", "spring-boot:run"]

```

[Sample code for a simple Java application](https://github.com/DataDog/crpb/tree/main/java).

{{< /programming-lang >}}
{{< programming-lang lang="dotnet" >}}

```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-python
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["dotnet", "helloworld.dll"]

```

{{< /programming-lang >}}
{{< programming-lang lang="ruby" >}}

```
# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# optionally add Datadog tags
ENV DD_SERVICE=datadog-demo-run-python
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["rails", "server", "-b", "0.0.0.0"] (adapt this line to your needs)

```

[Sample code for a simple Ruby application](https://github.com/DataDog/crpb/tree/main/ruby-on-rails).

{{< /programming-lang >}}
{{< /programming-lang-wrapper >}}

#### Instrument using buildpack

[`Pack Buildpacks`][4] provide a convenient way to package your container without using a Dockerfile. This example will use the GCP container registry and Datadog serverless buildpack. Build your application by running the following command:

   ```shell
   pack build --builder=gcr.io/buildpacks/builder \
   --buildpack from=builder \
   --buildpack datadog/serverless-buildpack \
   gcr.io/YOUR_PROJECT/YOUR_APP_NAME
   ```

**Note**: Not compatible with Alpine.

### Push the container image to a registry

Depending on your deployment process, push the image built to a registry. The `gcr.io` prefix in the examples above assumed [Google Cloud Registry][11].

### Deploy the Application with the Datadog Agent

{{< tabs >}}

{{% tab "Cloud Run web UI" %}}

1. Create a secret with Datadog API key. This can be done via [Secret Manager](https://console.cloud.google.com/security/secret-manager) in your GCP console and clicking on **Create secret**. Set a name (for example, `datadog-api-key`) in the **Name** field. Then, paste your Datadog API key in the **Secret value** field.

2. Go to [Cloud Run](https://console.cloud.google.com/run) in your GCP console. and click on **Create service**.

3. Select **Deploy one revision from an existing container image**. Choose your previously built image.

4. Select your invocation authentication method.

5. Reference your previously created secret, named `datadog-api-key` in this guide. Go to the **Container, Networking, Security** section and select the **Secrets** tab. click on **Reference a secret** and choose the secret you created from your Datadog API key. You may need to grant your user access to the secret.

6. Under **Reference method**, select **Exposed as environment variable**.

7. Under the **Environment variables** section, ensure that the name is set to `DD_API_KEY`.

8. Still under **Environment variables**, create two more variables. One named `DD_TRACE_ENABLED` set to `true` to enable tracing. Another named `DD_SITE` containing the Datadog site which will collect the data (if not set, it defaults to `datadoghq.com`). Your `DD_SITE` is {{< region-param key="dd_site" code="true" >}} (ensure the correct SITE is selected on the right of the page).

{{% /tab %}}
{{% tab "gcloud CLI" %}}
For testing purpose, the Datadog API key can be exposed as an environment variable. That is unsafe due to its value being displayed in plaintext. Allowing any external connection to reach the service, this one line command will deploy the service. It expects `DD_API_KEY` to be set as environment variable and a service listening to port 80

```shell
gcloud run deploy APP_NAME --image=gcr.io/YOUR_PROJECT/APP_NAME \
  --port=80 \
  --update-env-vars=DD_API_KEY=$(DD_API_KEY) \
  --update-env-vars=DD_TRACE_ENABLED=true \
  --update-env-vars=DD_SITE='datadoghq.com' \
  --allow-unauthenticated

```

{{% /tab %}}
{{< /tabs >}}

### Results

Once the deploy is completed, you should be able to see metrics and traces of your Cloud Run application in the Datadog UI after at most a few minutes!

## Additional Configurations

- **Tracing:** the Datadog Agent already provides some basic tracing for popular frameworks. Follow the guide for [advanced tracing](2) for more

- **Logs:** If you use [GCP integration][1] your logs are already being collected. Alternatively, you can set the `DD_LOGS_ENABLED` environment variable to `true` to capture application logs through the serverless instrumentation directly.

- **Custom Metrics:** You can submit custom metrics using a [DogStatsd client][7]. Only `DISTRIBUTION` metrics should be used.

- **Environment Variables**

| Variable          | Description                                                             |
|-------------------|-------------------------------------------------------------------------|
| `DD_SITE`         | [Datadog site][8].                                                      |
| `DD_LOGS_ENABLED` | When true, send logs (stdout and stderr) to Datadog. Defaults to false. |
| `DD_SERVICE`      | See [Unified Service Tagging][9].                                       |
| `DD_VERSION`      | See [Unified Service Tagging][9].                                       |
| `DD_ENV`          | See [Unified Service Tagging][9].                                       |
| `DD_SOURCE`       | See [Unified Service Tagging][9].                                       |
| `DD_TAGS`         | See [Unified Service Tagging][9].                                       |

## Troubleshooting

This integration depends on your runtime having a full SSL implementation. If you are using a slim image for Node, you may need to add the following command to your Dockerfile to include certificates.

```
RUN apt-get update && apt-get install -y ca-certificates
```

## Further reading

{{< partial name="whats-next/whats-next.html" >}}


[1]: /integrations/google_cloud_platform/#log-collection

[2]: /tracing/trace_collection/#for-setup-instructions-select-your-language

[3]: https://registry.hub.docker.com/r/datadog/serverless-init

[4]: https://buildpacks.io/docs/tools/pack/

[5]: https://console.cloud.google.com/security/secret-manager

[6]: https://console.cloud.google.com/run

[7]: /metrics/custom_metrics/dogstatsd_metrics_submission/

[8]: /getting_started/site/

[9]: /getting_started/tagging/unified_service_tagging/

[10]: /account_management/api-app-keys/

[11]: https://cloud.google.com/run/docs/deploying.

[12]: https://github.com/DataDog/crpb/tree/main
