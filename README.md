# bitnami-redis-ca-cert-shenanigans

In order to use TLS, Redis needs

> a CA certificate bundle file or path to be used as a trusted root when validating certificates [[source](https://redis.io/docs/latest/operate/oss_and_stack/management/security/encryption/#:~:text=In%20order%20to%20support%20TLS%2C%20Redis%20must%20be%20configured%20with%20a%20X.509%20certificate%20and%20a%20private%20key.%20In%20addition%2C%20it%20is%20necessary%20to%20specify%20a%20CA%20certificate%20bundle%20file%20or%20path%20to%20be%20used%20as%20a%20trusted%20root%20when%20validating%20certificates)]

`redis-cli` is a little more flexible in that it will:

> If neither cacert nor cacertdir are specified, the default system-wide trusted root certs configuration will apply. [[source](https://valkey.io/topics/cli/#:~:text=If%20neither%20cacert%20nor%20cacertdir%20are%20specified%2C%20the%20default%20system%2Dwide%20trusted%20root%20certs%20configuration%20will%20apply.) (Disclaimer: these are `valkey` docs, but `redis-cli --help` says the same thing]

As such, `bitnami/redis` also insists upon being given a CA certificate.

This presents a problem when working with `cert-manager` which doesn't always include a `ca.crt` when issuing certificates:

> Additionally, if the Certificate Authority is known, the corresponding CA certificate will be stored in the secret with key ca.crt. For example, with the ACME issuer, the CA is not known and ca.crt will not exist in acme-crt-secret. [[source](https://cert-manager.io/v1.1-docs/concepts/certificate/#:~:text=Additionally%2C%20if%20the%20Certificate%20Authority%20is%20known%2C%20the%20corresponding%20CA%20certificate%20will%20be%20stored%20in%20the%20secret%20with%20key%20ca.crt.%20For%20example%2C%20with%20the%20ACME%20issuer%2C%20the%20CA%20is%20not%20known%20and%20ca.crt%20will%20not%20exist%20in%20acme%2Dcrt%2Dsecret.)]

This leads to trouble with `bitnami/redis` because I'm left with two options:
1. Create a custom secret from the `cert-manager` secret that also includes a manually retrieved copy of [the appropriate root certificate](https://crt.sh/?id=9314791).
2. Hack a way around.

Option 1 doesn't work great for a CA like Let's Encrypt which issues certificates with a TTL of 90 days.

Option 2 is the route I've been forced to go, but the path forward is brittle and hacky.

This repository documents the hack I'm using and would like a better solution for.

## How to run kustomize to reify `bitnami/redis` chart

```bash
kustomize build \
  --enable-alpha-plugins \
  --enable-helm \
  --load-restrictor LoadRestrictionsNone \
  .
```

or

```bash
./scripts/kustomize.sh
```
