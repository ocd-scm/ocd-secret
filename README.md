# ocd-secret

The idea is that you can use Helmfile configuration in git to cookie cutter dozens of shard secrets for your microservices using this single chart.

See https://ocd-scm.github.io/ocd-meta/ for the bigger picture. 

You can use this chart with helmfile to install secrets that can be mounted as environment varibles or files into other k8s objects. To encode some environment variables that are git-secret encrypted within git using [Helmfile](https://github.com/roboll/helmfile):

```
releases:
  - name: my-secrets
    labels:
      secret: my-secrets
    chart: ocd-meta/ocd-secret
    version: ~1.0.0
    values:
      - name: my-secrets
      - "secret-env-vars.yaml
```

Then create an encrypted `secret-env-vars.yaml` see the ocd-scm/ocd-meta about how to encrypt in a way that OCD webhooks can decrypt. The setting you encrypt would be something like:

```
env_vars: 
  - key: USERNAME
    value: "hello"
  - key: PASSWORD
    value: "world"
```

The secret can then be mounted as env vars into an `ocd-deploy` release with:

```
      - secrets:
        - name: my-secrets
```

The list of secret names is then evaluated into a list of `envFrom` settings on a deployment. The `envFrom` setting support adding a prefix to all keys that you can set with:

```
      - secrets:
        - name: my-secrets
          prefix: 'FALLBACK_'
```


Note if you use a single Helm Tiller to manage multiple projects that will have different values e.g., different database logins you need the release name, but not the secret name, to be unique. The OCD environment webhook sets an env var "ENV_PREFIX" based on the project name you can use for this purpose:

```
releases:
  - name: {{ requiredEnv "ENV_PREFIX" }}-my-secrets
    labels:
      secret: my-secrets
    chart: ocd-meta/ocd-secret
    version: ~1.0.0
    values:
      - name: my-secrets
      - "secret-env-vars.yaml
```

If you want to create an scmsecret or other secrets that are mounted as files then you need your secret to be a map where the key are the filenames and the values are the base64 encoded file contents. For example here is an example:

```
releases:
  - name: {{ requiredEnv "ENV_PREFIX" }}-scmsecret-mybuilder
    labels:
      secret: scmsecret-mybuilder
    chart: ocd-meta/ocd-secret
    version:  "1.0.0-SNAPSHOT"
    values:
      - name: scmsecret-mybuilder
      - "values.yaml.gotmpl"
```

Where `values.yaml.gotmpl` does a base64 encoding of file data:

```
data:
  - key: ssh-privatekey
    value: >-
{{ exec "base64" (list "-w" "0" "scm-private-key") | indent 6 }}
```

The file `scm-private-key` can be git-secret encrypted in which case it is in your `.gitignore` and only the GPG encrypted `scm-private-key.secret` will be in git. See the ocd-scm/ocd-meta wiki about encryption. 
