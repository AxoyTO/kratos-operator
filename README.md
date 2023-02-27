# Charmed Ory Kratos

## Description

This repository hosts the Kubernetes Python Operator for Ory Kratos -  an API-first identity and user management system.
For more details, visit https://www.ory.sh/docs/kratos/ory-kratos-intro

## Usage

The Kratos Operator may be deployed using the Juju command line as follows:

```bash
juju deploy postgresql-k8s --channel edge --trust
juju deploy kratos
juju relate kratos postgresql-k8s
```

To set the smtp connection uri, do:

```bash
juju config kratos smtp_connection_uri={smtp_connection_uri}
```

### Interacting with Kratos API

Below are two examples of the API. Visit [Ory](https://www.ory.sh/docs/kratos/reference/api) to see full API specification.

#### Create Identity
```bash
curl <kratos-service-ip>:4434/identities \
--request POST -sL \
--header "Content-Type: application/json" \
--data '{
  "schema_id": "default",
  "traits": {
    "email": "test@example.org"
  }
}'
```
#### Get Identities
```bash
curl <kratos-service-ip>:4434/admin/identities
```
You should be able to see the identity created earlier.

## Relations

### PostgreSQL

This charm requires a relation with [postgresql-k8s-operator](https://github.com/canonical/postgresql-k8s-operator).

### Ingress

The Kratos Operator offers integration with the [traefik-k8s-operator](https://github.com/canonical/traefik-k8s-operator) for ingress. Kratos has two APIs which can be exposed through ingress, the public API and the admin API.

If you have a traefik deployed and configured in your kratos model, to provide ingress to the admin API run:
```console
juju relate traefik-admin kratos:admin-ingress
```

To provide ingress to the public API run:
```console
juju relate traefik-public kratos:public-ingress
```

### External Provider Relation

Kratos can be used as an identity broker. To connect Kratos with an external identity provider you can use the external provider relation. All you need to do is deploy the [kratos-external-idp-integrator](https://charmhub.io/kratos-external-idp-integrator), configure it and relate it to Kratos:

```console
juju deploy kratos-external-provider-integrator
juju config kratos-external-provider-integrator \
    client_id={client_id} \
    client_secret={client_secret} \
    provider={provider}
juju relate kratos-external-provider-integrator kratos
```

Once kratos has registered the provider, you will be able to retrieve the redirect_uri from the integrator by running:
```console
juju run-action {external_provider_integrator_unit_name} get-redirect-uri --wait
```

### Hydra

This charm offers integration with [hydra-operator](https://github.com/canonical/hydra-operator).

## Integration with Hydra and UI

If you have deployed [Login UI charm](https://github.com/canonical/identity-platform-login-ui), you can configure it with kratos by providing its URL.
Note that the UI charm should run behind a proxy.
```console
juju config kratos kratos_ui_url=http://{traefik_public_ip}/{model_name}-{kratos_ui_app_name}
```

In order to integrate kratos with hydra, it needs to be able to access hydra's admin API endpoint.
To enable that, relate the two charms:
```console
juju relate kratos hydra
```

For further guidance on integration on hydra side, visit the [hydra-operator](https://github.com/canonical/hydra-operator#readme) repository.

## OCI Images

The image used by this charm is hosted on [Docker Hub](https://hub.docker.com/r/oryd/kratos) and maintained by Ory.

### Security
Security issues in IAM stack can be reported through [LaunchPad](https://wiki.ubuntu.com/DebuggingSecurity#How%20to%20File). Please do not file GitHub issues about security issues.

## Contributing

Please see the [Juju SDK docs](https://juju.is/docs/sdk) for guidelines on enhancements to this charm following best practice guidelines, and [CONTRIBUTING.md](https://github.com/canonical/kratos-operator/blob/main/CONTRIBUTING.md) for developer guidance.


## License

The Charmed Kratos Operator is free software, distributed under the Apache Software License, version 2.0. See [LICENSE](https://github.com/canonical/kratos-operator/blob/main/LICENSE) for more information.
