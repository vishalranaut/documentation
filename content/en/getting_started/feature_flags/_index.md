---
title: Getting Started with Feature Flags
description: Manage feature delivery with integrated observability, real-time metrics, and OpenFeature-compatible gradual rollouts.
further_reading:
    - link: '/feature_flags/client/'
      tag: 'Documentation'
      text: 'Client-Side SDKs'
    - link: '/feature_flags/server/'
      tag: 'Documentation'
      text: 'Server-Side SDKs'
    - link: 'https://www.datadoghq.com/blog/feature-flags/'
      tag: 'Blog'
      text: 'Ship features faster and safer with Datadog Feature Flags'
    - link: 'https://www.datadoghq.com/blog/experimental-data-datadog/'
      tag: 'Blog'
      text: 'How to bridge speed and quality in experiments through unified data'
    - link: 'https://www.datadoghq.com/blog/datadog-feature-flags-cloud-resilience/'
      tag: 'Blog'
      text: 'How Datadog Feature Flags is resilient to cloud provider failures'
    - link: "https://www.datadoghq.com/blog/guardrail-metrics"
      tag: "Blog"
      text: "Make use of guardrail metrics and stop babysitting your releases"
site_support_id: getting_started_feature_flags
---

## Overview

Datadog feature flags offer a powerful, integrated way to manage feature delivery, with built-in observability and seamless integration across the platform.

- **Real-time metrics:** Understand who's receiving each variant, as well as how your flag impacts the health & performance of your application—all in real time.

- **Supports common flag types:** Use Boolean, string, integer, numeric (float/double), or JSON variants. JavaScript SDKs use `getNumberValue()` for both integer and numeric variants, while Java, Swift, Kotlin, and Python expose separate integer and floating-point evaluation methods.

- **Built for experimentation:** Target specific audiences for A/B tests, roll out features gradually with canary releases, and automatically roll back when regressions are detected.

- **OpenFeature compatible:** Built on the OpenFeature standard, ensuring compatibility with existing OpenFeature implementations and providing a vendor-neutral approach to feature flag management.

## Feature Flags SDKs

This guide uses the JavaScript browser SDK as an example. You can integrate Datadog Feature Flags into any application using one of the following SDKs:

### Client-side SDKs

{{< partial name="feature_flags/feature_flags_client.html" >}}

### Server-side SDKs

{{< partial name="feature_flags/feature_flags_server.html" >}}

## Configure your environments

Your organization likely already has pre-configured environments for Development, Staging, and Production. If you need to configure these or any other environments, navigate to the [{{< ui >}}Environments{{< /ui >}}][3] page to create tag queries for each environment. You can also identify which environment should be considered a Production environment.

{{< img src="getting_started/feature_flags/environments-list.png" alt="Environments list" style="width:100%;" >}}

## Create your first feature flag

### Step 1: Import and initialize the SDK

Choose the SDK that matches where the flag is evaluated and initialize the Datadog Feature Flags provider.

{{< tabs >}}
{{% tab "JavaScript browser" %}}

Install `@datadog/openfeature-browser`, `@openfeature/web-sdk`, and `@openfeature/core` as dependencies in your project:

{{< code-block lang="bash" >}}
yarn add @datadog/openfeature-browser @openfeature/web-sdk @openfeature/core
{{< /code-block >}}

Then, add the following to your project to initialize the SDK:

Note: Browser Feature Flags are currently not supported on GovCloud sites.

{{< code-block lang="javascript" >}}
import { DatadogProvider } from '@datadog/openfeature-browser';
import { OpenFeature } from '@openfeature/web-sdk';

// Initialize the provider
const provider = new DatadogProvider({
    // Required client-side Datadog credentials
    applicationId: '<APPLICATION_ID>',
    clientToken: '<CLIENT_TOKEN>',
    site: '{{< region-param key="dd_site" code="true" >}}',
    env: '<YOUR_ENV>', // Same environment normally passed to the RUM SDK
    service: '<SERVICE_NAME>',
    version: '1.0.0'
});

// Set the provider
await OpenFeature.setProviderAndWait(provider);
{{< /code-block >}}

<div class="alert alert-info">The browser SDK emits three independent telemetry streams, all enabled by default. <code>enableExposureLogging</code> sends per-evaluation exposure events to the exposures intake. <code>enableFlagEvaluationTracking</code> sends aggregated evaluation telemetry to the flag-evaluation intake. <code>enableRumFeatureFlagTracking</code> attaches flag evaluations to RUM events and is the setting that can affect RUM usage. Disable only the stream you do not need.</div>

{{% /tab %}}
{{% tab "Node.js server" %}}

Install `dd-trace` and the OpenFeature server SDK:

{{< code-block lang="bash" >}}
npm install dd-trace @openfeature/server-sdk
{{< /code-block >}}

Enable the provider with environment variables:

{{< code-block lang="bash" >}}
# Required: Enable the feature flags provider
DD_EXPERIMENTAL_FLAGGING_PROVIDER_ENABLED=true

# Optional: Enable flag evaluation metrics
DD_METRICS_OTEL_ENABLED=true
{{< /code-block >}}

Or enable the provider in code:

{{< code-block lang="javascript" >}}
import { OpenFeature } from '@openfeature/server-sdk'
import tracer from 'dd-trace';

tracer.init({
  experimental: {
    flaggingProvider: {
      enabled: true,
    }
  }
});

OpenFeature.setProvider(tracer.openfeature);
{{< /code-block >}}

{{% /tab %}}
{{% tab "Java" %}}

Add the OpenFeature SDK and Datadog OpenFeature provider dependencies:

{{< code-block lang="groovy" filename="build.gradle" >}}
dependencies {
    // OpenFeature SDK for flag evaluation
    implementation 'dev.openfeature:sdk:1.18.2'

    // Datadog OpenFeature Provider
    implementation 'com.datadoghq:dd-openfeature:1.57.0'
}
{{< /code-block >}}

Enable the provider and start your application with the Java tracer:

{{< code-block lang="bash" >}}
# Required: Enable the feature flagging provider
# The EXPERIMENTAL_ prefix is historical; the provider is no longer experimental.
export DD_EXPERIMENTAL_FLAGGING_PROVIDER_ENABLED=true

# Optional: Enable flag evaluation metrics
export DD_METRICS_OTEL_ENABLED=true

java -javaagent:path/to/dd-java-agent.jar -jar your-application.jar
{{< /code-block >}}

Register the Datadog OpenFeature provider:

{{< code-block lang="java" >}}
import dev.openfeature.sdk.OpenFeatureAPI;
import dev.openfeature.sdk.Client;
import datadog.trace.api.openfeature.Provider;

OpenFeatureAPI api = OpenFeatureAPI.getInstance();
api.setProviderAndWait(new Provider());
Client client = api.getClient("my-app");
{{< /code-block >}}

{{% /tab %}}
{{% tab "Python" %}}

Enable the provider with environment variables:

{{< code-block lang="bash" >}}
# Required: Enable the feature flags provider
export DD_EXPERIMENTAL_FLAGGING_PROVIDER_ENABLED=true

# Optional: Enable flag evaluation metrics
export DD_METRICS_OTEL_ENABLED=true
{{< /code-block >}}

Install the Datadog Python SDK and OpenFeature SDK:

{{< code-block lang="bash" >}}
pip install ddtrace openfeature-sdk
{{< /code-block >}}

Register the Datadog OpenFeature provider:

{{< code-block lang="python" >}}
from ddtrace import tracer
from openfeature import api
from ddtrace.openfeature import DataDogProvider

# Initialize the tracer (required for Remote Configuration)
tracer.configure()

# Create and register the Datadog provider
provider = DataDogProvider()
api.set_provider(provider)

# Create an OpenFeature client
client = api.get_client()
{{< /code-block >}}

{{% /tab %}}
{{< /tabs >}}

### Credentials at a glance

| Credential | Used by | Where it goes | Sensitive? |
| --- | --- | --- | --- |
| Client token | Browser, mobile, and game SDKs | Client application configuration | Public-shipping token |
| Application ID | Browser and RUM-backed client SDKs | Client application configuration | Public-shipping identifier |
| API key | Datadog Agent for server-side Remote Configuration | Agent configuration only | Secret |

Do not put API keys in browser, mobile, or game applications.

See the platform-specific SDK documentation linked above for more configuration details. For more information on creating client tokens and application IDs, see [API and Application Keys][4].

### Step 2: Create a feature flag

Go to [{{< ui >}}Create Feature Flag{{< /ui >}}][2] in Datadog and configure the following:

- {{< ui >}}Name and key{{< /ui >}}: The flag's display name and the key referenced in code
- {{< ui >}}Variant type{{< /ui >}}: The data type for the flag variants (Boolean, string, integer, numeric (float/double), or JSON)

    **Note**: The {{< ui >}}flag key{{< /ui >}} and {{< ui >}}variant type{{< /ui >}} cannot be modified after creation.

    JavaScript SDKs use `getNumberValue()` for both integer and numeric variants. Java, Swift, Kotlin, and Python expose integer and floating-point methods separately.

- {{< ui >}}Variant values{{< /ui >}}: The possible values the flag can return (you can add these later)
- {{< ui >}}Distribution channels{{< /ui >}}: Which types of SDKs receive this flag's configuration (client-side, server-side, or both)

<div class="alert alert-warning">
  {{< ui >}}Flag keys{{< /ui >}}, {{< ui >}}variant keys{{< /ui >}}, and {{< ui >}}variant values{{< /ui >}} should be considered public when sent to client SDKs.
</div>

{{< img src="getting_started/feature_flags/create-feature-flags.png" alt="Create Feature Flag" style="width:100%;" >}}

### Step 3: Evaluate the flag and write feature code

In your application code, use the SDK to evaluate the flag and gate the new feature.

<div class="alert alert-warning">Datadog Feature Flags requires evaluation context attributes to be flat primitive values: strings, numbers, and Booleans. Do not pass nested objects or arrays; they are not supported and can cause exposure data to be dropped.</div>

{{< tabs >}}
{{% tab "JavaScript browser" %}}

{{< code-block lang="javascript" >}}
import { OpenFeature } from '@openfeature/web-sdk';

const client = OpenFeature.getClient();

// If applicable, set relevant attributes on the client's global context
// (e.g. org id, user email)
await OpenFeature.setContext({
    org_id: 2,
    user_id: 'user-123',
    email: 'user@example.com',
    targetingKey: 'user-123'
});

// This is what the SDK returns if the flag is disabled in
// the current environment
const fallback = false;

const showFeature = await client.getBooleanValue('show-new-feature', fallback);
if (showFeature) {
    // Feature code here
}
{{< /code-block >}}

{{% /tab %}}
{{% tab "Node.js server" %}}

{{< code-block lang="javascript" >}}
const evaluationContext = {
  targetingKey: req.session?.userID ?? 'unknown',
  companyID: req.session?.companyID
};

const isNewCheckoutEnabled = await client.getBooleanValue(
    'new-checkout-flow', // flag key
    false, // default value
    evaluationContext, // context
);

if (isNewCheckoutEnabled) {
    showNewCheckoutFlow();
} else {
    showLegacyCheckout();
}
{{< /code-block >}}

{{% /tab %}}
{{% tab "Java" %}}

{{< code-block lang="java" >}}
import dev.openfeature.sdk.EvaluationContext;
import dev.openfeature.sdk.MutableContext;

EvaluationContext context = new MutableContext("user-123")
    .add("email", "user@example.com")
    .add("tier", "premium");

boolean enabled = client.getBooleanValue("checkout.new", false, context);

if (enabled) {
    // New checkout flow
} else {
    // Old checkout flow
}
{{< /code-block >}}

{{% /tab %}}
{{% tab "Python" %}}

{{< code-block lang="python" >}}
from openfeature.evaluation_context import EvaluationContext

eval_ctx = EvaluationContext(
    targeting_key="user-123",
    attributes={
        "email": "user@example.com",
        "tier": "premium"
    }
)

enabled = client.get_boolean_value("new-checkout-flow", False, eval_ctx)

if enabled:
    show_new_checkout()
else:
    show_legacy_checkout()
{{< /code-block >}}

{{% /tab %}}
{{< /tabs >}}

After you've completed this step, redeploy the application to pick up these changes. Additional usage examples can be found in the platform-specific SDK pages linked above.

### Step 4: Define targeting rules and enable the feature flag

Now that the application is ready to check the value of your flag, you can start adding targeting rules. Targeting rules enable you to define where or to whom to serve different variants of your feature.

Go to {{< ui >}}Feature Flags{{< /ui >}}, select your flag, select the environment whose rules you want to modify, and click {{< ui >}}Edit Targeting Rules{{< /ui >}}.

{{< img src="getting_started/feature_flags/ff-targeting-rules-and-rollouts.png" alt="Targeting Rules & Rollouts" style="width:100%;" >}}

### Step 5: Publish the rules in your environments

After saving changes to the targeting rules, publish those rules by enabling your flag in the environment of your choice.

<div class="alert alert-info">
As a general best practice, changes should be rolled out in a Staging environment before rolling out in Production.
</div>

Toggle your selected environment to {{< ui >}}Enabled{{< /ui >}}.

{{< img src="getting_started/feature_flags/publish-targeting-rules.png" alt="Publish targeting rules" style="width:100%;" >}}

The flag serves your targeting rules in this environment. You can continue to edit these targeting rules to control where the variants are served.

### Step 6: Monitor your rollout

Monitor the feature rollout from the feature flag details page, which provides real-time exposure tracking and metrics such as {{< ui >}}error rate{{< /ui >}} and {{< ui >}}page load time{{< /ui >}}. As you incrementally release the feature with the flag, view the {{< ui >}}Real-Time Metric Overview{{< /ui >}} panel in the Datadog UI to see how the feature impacts application performance.

{{< img src="getting_started/feature_flags/real-time-flag-metrics.png" alt="Real-time flag metrics panel" style="width:100%;" >}}

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[2]: https://app.datadoghq.com/feature-flags/create
[3]: https://app.datadoghq.com/feature-flags/settings/environments
[4]: https://docs.datadoghq.com/account_management/api-app-keys/#client-tokens
