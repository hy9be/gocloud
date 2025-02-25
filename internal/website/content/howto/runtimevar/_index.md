---
title: "Runtime Configuration"
date: 2019-07-11T12:00:00-07:00
lastmod: 2020-12-23T12:00:00-07:00
showInSidenav: true
toc: true
---

The [`runtimevar` package][] provides an easy and portable way to watch runtime
configuration variables. This guide shows how to work with runtime configuration
variables using the Go CDK.

<!--more-->

Subpackages contain driver implementations of runtimevar for various services,
including Cloud and on-prem solutions. You can develop your application locally
using [`filevar`][] or [`constantvar`][], then deploy it to multiple Cloud
providers with minimal initialization reconfiguration.

[`runtimevar` package]: https://godoc.org/gocloud.dev/runtimevar
[`filevar`]: https://godoc.org/gocloud.dev/runtimevar/filevar
[`constantvar`]: https://godoc.org/gocloud.dev/runtimevar/constantvar

## Opening a Variable {#opening}

The first step in watching a variable is to instantiate a portable
[`*runtimevar.Variable`][] for your service.

The easiest way to do so is to use [`runtimevar.OpenVariable`][] and a service-specific URL pointing
to the variable, making sure you ["blank import"][] the driver package to link
it in.

```go
import (
	"github.com/hy9be/gocloud/runtimevar"
	_ "github.com/hy9be/gocloud/runtimevar/<driver>"
)
...
v, err := runtimevar.OpenVariable(context.Background(), "<driver-url>")
if err != nil {
    return fmt.Errorf("could not open variable: %v", err)
}
defer v.Close()
// v is a *runtimevar.Variable; see usage below
...
```

See [Concepts: URLs][] for general background and the [guide below][]
for URL usage for each supported service.

Alternatively, if you need fine-grained control
over the connection settings, you can call the constructor function in the
driver package directly (like `etcdvar.OpenVariable`).

```go
import "github.com/hy9be/gocloud/runtimevar/<driver>"
...
v, err := <driver>.OpenVariable(...)
...
```

You may find the [`wire` package][] useful for managing your initialization code
when switching between different backing services.

See the [guide below][] for constructor usage for each supported service.

When opening the variable, you can provide a [decoder][] parameter (either as a
[query parameter][] for URLs, or explicitly to the constructor) to specify
whether the raw value stored in the variable is interpreted as a `string`, a
`[]byte`, or as JSON. Here's an example of using a JSON encoder:

{{< goexample src="github.com/hy9be/gocloud/runtimevar.Example_jsonDecoder" imports="0" >}}

[`*runtimevar.Variable`]: https://godoc.org/gocloud.dev/runtimevar#Variable
[`runtimevar.OpenVariable`]: https://godoc.org/gocloud.dev/runtimevar#OpenVariable
["blank import"]: https://golang.org/doc/effective_go.html#blank_import
[Concepts: URLs]: {{< ref "/concepts/urls.md" >}}
[decoder]: https://godoc.org/gocloud.dev/runtimevar#Decoder
[guide below]: {{< ref "#services" >}}
[query parameter]: https://godoc.org/gocloud.dev/runtimevar#DecoderByName
[`wire` package]: http://github.com/google/wire

## Using a Variable {#using}

Once you have opened a `runtimevar.Variable` for the provider you want, you can
use it portably.

### Latest {#latest}

The easiest way to a `Variable` is to use the [`Variable.Latest`][] method. It
returns the latest good [`Snapshot`][] of the variable value, blocking if no
good value has *ever* been detected. The dynamic type of `Snapshot.Value`
depends on the decoder you provided when creating the `Variable`.

{{< goexample src="github.com/hy9be/gocloud/runtimevar.ExampleVariable_Latest" imports="0" >}}

To avoid blocking, you can pass an already-`Done` context. You can also use
[`Variable.CheckHealth`][], which reports as healthy when `Latest` will
return a value without blocking.

[`Variable.Latest`]: https://godoc.org/gocloud.dev/runtimevar#Variable.Latest
[`Variable.CheckHealth`]: https://godoc.org/gocloud.dev/runtimevar#Variable.CheckHealth
[`Snapshot`]: https://godoc.org/gocloud.dev/runtimevar#Snapshot

### Watch {#watch}

`Variable` also has a [`Watch`][] method for obtaining the value of a variable;
it has different semantics than `Latest` and may be useful in some scenarios. We
recommend starting with `Latest` as it's conceptually simpler to work with.

[`Watch`]: https://godoc.org/gocloud.dev/runtimevar#Variable.Watch

## Other Usage Samples

* [CLI Sample](https://github.com/google/go-cloud/tree/master/samples/gocdk-runtimevar)
* [Guestbook sample](https://gocloud.dev/tutorials/guestbook/)
* [runtimevar package examples](https://godoc.org/gocloud.dev/runtimevar#pkg-examples)

## Supported Services {#services}

### GCP Runtime Configurator {#gcprc}

To open a variable stored in [GCP Runtime Configurator][] via a URL, you can use
the `runtimevar.OpenVariable` function as shown in the example below.

[GCP Runtime Configurator]: https://cloud.google.com/deployment-manager/runtime-configurator/

`runtimevar.OpenVariable` will use Application Default Credentials; if you have
authenticated via [`gcloud auth application-default login`][], it will use those credentials. See
[Application Default Credentials][GCP creds] to learn about authentication
alternatives, including using environment variables.

[GCP creds]: https://cloud.google.com/docs/authentication/production
[`gcloud auth application-default login`]: https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login

{{< goexample
"github.com/hy9be/gocloud/runtimevar/gcpruntimeconfig.Example_openVariableFromURL" >}}

#### GCP Constructor {#gcprc-ctor}

The [`gcpruntimeconfig.OpenVariable`][] constructor opens a Runtime Configurator
variable.

{{< goexample "github.com/hy9be/gocloud/runtimevar/gcpruntimeconfig.ExampleOpenVariable" >}}

[`gcpruntimeconfig.OpenVariable`]: https://godoc.org/gocloud.dev/runtimevar/gcpruntimeconfig#OpenVariable

### GCP Secret Manager {#gcpsm}

To open a variable stored in [GCP Secret Manager][] via a URL, you can use
the `runtimevar.OpenVariable` function as shown in the example below.

[GCP Secret Manager]: https://cloud.google.com/secret-manager

`runtimevar.OpenVariable` will use Application Default Credentials; if you have
authenticated via [`gcloud auth application-default login`][], it will use those credentials. See
[Application Default Credentials][GCP creds] to learn about authentication
alternatives, including using environment variables.

[GCP creds]: https://cloud.google.com/docs/authentication/production
[`gcloud auth application-default login`]: https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login

{{< goexample
"github.com/hy9be/gocloud/runtimevar/gcpsecretmanager.Example_openVariableFromURL" >}}

#### GCP Constructor {#gcpsm-ctor}

The [`gcpsecretmanager.OpenVariable`][] constructor opens a Secret Manager
variable.

{{< goexample "github.com/hy9be/gocloud/runtimevar/gcpsecretmanager.ExampleOpenVariable" >}}

[`gcpsecretmanager.OpenVariable`]: https://godoc.org/gocloud.dev/runtimevar/gcpsecretmanager#OpenVariable

### AWS Parameter Store {#awsps}

To open a variable stored in [AWS Parameter Store][] via a URL, you can use the
`runtimevar.OpenVariable` function as shown in the example below.

[AWS Parameter Store]:
https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html

If you set the "awssdk=v1" query parameter,
`runtimevar.OpenVariable` will create a default AWS Session with the
`SharedConfigEnable` option enabled; if you have authenticated with the AWS CLI,
it will use those credentials. See [AWS Session][] to learn about authentication
alternatives, including using environment variables.

If you set the "awssdk=v2" query parameter, it will instead create an AWS
Config based on the AWS SDK V2; see [AWS V2 Config][] to learn more.

If no "awssdk" query parameter is set, Go CDK will use a default (currently V1).

[AWS Session]: https://docs.aws.amazon.com/sdk-for-go/api/aws/session/
[AWS V2 Config]: https://aws.github.io/aws-sdk-go-v2/docs/configuring-sdk/

{{< goexample
"github.com/hy9be/gocloud/runtimevar/awsparamstore.Example_openVariableFromURL" >}}

#### AWS Constructor {#awsps-ctor}

The [`awsparamstore.OpenVariable`][] constructor opens a Parameter Store
variable.

{{< goexample "github.com/hy9be/gocloud/runtimevar/awsparamstore.ExampleOpenVariable" >}}

[`awsparamstore.OpenVariableV2`][] is similar but uses the AWS SDK V2.

{{< goexample "github.com/hy9be/gocloud/runtimevar/awsparamstore.ExampleOpenVariableV2" >}}

[`awsparamstore.OpenVariable`]:
https://godoc.org/gocloud.dev/runtimevar/awsparamstore#OpenVariable
[`awsparamstore.OpenVariableV2`]:
https://godoc.org/gocloud.dev/runtimevar/awsparamstore#OpenVariableV2

### AWS Secrets Manager {#awssm}

To open a variable stored in [AWS Secrets Manager][] via a URL, you can use the
`runtimevar.OpenVariable` function as shown in the example below.

[AWS Secrets Manager]:
https://aws.amazon.com/secrets-manager

If you set the "awssdk=v1" query parameter,
`runtimevar.OpenVariable` will create a default AWS Session with the
`SharedConfigEnable` option enabled; if you have authenticated with the AWS CLI,
it will use those credentials. See [AWS Session][] to learn about authentication
alternatives, including using environment variables.

If you set the "awssdk=v2" query parameter, it will instead create an AWS
Config based on the AWS SDK V2; see [AWS V2 Config][] to learn more.

If no "awssdk" query parameter is set, Go CDK will use a default (currently V1).

[AWS Session]: https://docs.aws.amazon.com/sdk-for-go/api/aws/session/
[AWS V2 Config]: https://aws.github.io/aws-sdk-go-v2/docs/configuring-sdk/

{{< goexample
"github.com/hy9be/gocloud/runtimevar/awssecretsmanager.Example_openVariableFromURL" >}}

#### AWS Constructor {#awssm-ctor}

The [`awssecretsmanager.OpenVariable`][] constructor opens a Secrets Manager
variable.

{{< goexample "github.com/hy9be/gocloud/runtimevar/awssecretsmanager.ExampleOpenVariable" >}}

[`awssecretsmanager.OpenVariableV2`][] is similar but uses the AWS SDK V2.

{{< goexample "github.com/hy9be/gocloud/runtimevar/awssecretsmanager.ExampleOpenVariableV2" >}}

[`awssecretsmanager.OpenVariable`]:
https://godoc.org/gocloud.dev/runtimevar/awssecretsmanager#OpenVariable
[`awssecretsmanager.OpenVariableV2`]:
https://godoc.org/gocloud.dev/runtimevar/awssecretsmanager#OpenVariableV2

Note that both `secretsmanager:GetSecretValue` and `secretsmanager:DescribeSecret` actions must be allowed in
the caller's IAM policy.

### etcd {#etcd}

*NOTE*: Support for `etcd` has been temporarily dropped due to dependency
issues. See https://github.com/google/go-cloud/issues/2914.

You can use `runtimevar.etcd` in Go CDK version `v0.20.0` or earlier.

### HTTP {#http}

`httpvar` supports watching a variable via an HTTP request. Use
`runtimevar.OpenVariable` with a regular URL starting with `http` or `https`.
`httpvar` will periodically make an HTTP `GET` request to that URL, with the
`decode` URL parameter removed (if present).

{{< goexample "github.com/hy9be/gocloud/runtimevar/httpvar.Example_openVariableFromURL" >}}

#### HTTP Constructor {#http-ctor}

The [`httpvar.OpenVariable`][] constructor opens a variable with a `http.Client`
and a URL.

{{< goexample "github.com/hy9be/gocloud/runtimevar/httpvar.ExampleOpenVariable" >}}

[`httpvar.OpenVariable`]: https://godoc.org/gocloud.dev/runtimevar/httpvar#OpenVariable

### Blob {#blob}

`blobvar` supports watching a variable based on the contents of a
[Go CDK blob][]. Set the environment variable `BLOBVAR_BUCKET_URL` to the URL
of the bucket, and then use `runtimevar.OpenVariable` as shown below.
`blobvar` will periodically re-fetch the contents of the blob.

{{< goexample "github.com/hy9be/gocloud/runtimevar/blobvar.Example_openVariableFromURL" >}}

[Go CDK blob]: https://gocloud.dev/howto/blob/

You can also use [`blobvar.OpenVariable`][].

[`blobvar.OpenVariable`]: https://godoc.org/gocloud.dev/runtimevar/blobvar#OpenVariable

### Local {#local}

You can create an in-memory variable (useful for testing) using `constantvar`:

{{< goexample "github.com/hy9be/gocloud/runtimevar/constantvar.Example_openVariableFromURL" >}}

Alternatively, you can create a variable based on the contents of a file using
`filevar`:

{{< goexample "github.com/hy9be/gocloud/runtimevar/filevar.Example_openVariableFromURL" >}}
