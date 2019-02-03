# ocd-secret

The idea is that you can use Helmfile configuration in git to cookie cutter dozens of secrets for your microservices using this single chart.

See https://ocd-scm.github.io/ocd-meta/ for the bigger picture. 

You can use this chart with helmfile to install secrets that can be mounted as environment varibles or files into other k8s objects. To set encode some environment variables that are encrypted into git checkout:

```
releases:
  - name: ocd-builder-credentials
    labels:
      secret: ocd-builder-credentials
    chart: ocd-meta/ocd-secret
    version: ~1.0.0
    values:
      - name: ocd-builder-credentials
      - "secret-env-vars.yaml
```

Then create an encrypted `secret-env-vars.yaml` see the ocd-scm/ocd-meta about how to encrypt. The GPG encrypted data would be something like:

```
env_vars: 
  - key: USERNAME
    value: "hello"
  - key: PASSWORD
    value: "world"
```

If you want to create an scmsecret or other secrets that are mounted as files then you need your secret to be a map where the key are the filenames and the values are the files. For example here is an example:

```
releases:
  - name: {{ requiredEnv "ENV_PREFIX" }}-scmsecret-backoffice
    labels:
      secret: scmsecret-backoffice
    chart: ocd-meta/ocd-secret
    version:  "1.0.0-SNAPSHOT"
    values:
      - name: scmsecret-backoffice
      - "values.yaml.gotmpl"
```

Where `values.yaml.gotmpl` does a base64 encoding of file data:

```
data:
  - key: ssh-privatekey
    value: >-
{{ exec "base64" (list "-w" "0" "scm-private-key") | indent 6 }}
```

The file `scm-private-key` can be git-secret encrypted see the ocd-scm/ocd-meta wiki about encryption. 
