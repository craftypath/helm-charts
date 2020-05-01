# SOPS Operator

This chart creates a [SOPS Operator](https://github.com/craftypath/sops-operator) deployment on a [Kubernetes](http://kubernetes.io)
cluster using the [Helm](https://helm.sh) package manager.

## Configuration

The following table lists the configurable parameters of the SOPS Operator chart and their default values.

Parameter | Description | Default
--- | --- | ---
`image.repository` | Docker image repository | `craftypath/sops-operator`
`image.tag` | Overrides the image tag whose default is the chart version | `The chart's version`
`image.pullPolicy` | Docker image pull policy | `IfNotPresent`
`replicas` | The number of replicas to run | `1`
`imagePullSecrets`| Image pull secrets | `[]`
`nameOverride` | Overrides the app name | ``
`fullnameOverride` | Overrides the full name for generating resources names | ``
`rbac.create` | Specifies whether RBAC resources should be created | `false`
`rbac.clusterScoped` | Custom RBAC rules for the master pod. If not specified, the `cluster-admin` ClusterRole is used | `true`
`serviceAccount.create` | Specifies whether a service account should be created | `true`
`serviceAccount.name` | If not set and ´serviceAcount.create` is true, a name is generated using the fullname template | `""`
`deployment.labels` | Additional labels for the deployment | `{}`
`deployment.annotations` | Annotations for the deployment | `{}`
`pod.labels` | Additional labels for the pod | `{}`
`pod.annotations` | Annotations for the pod | `{}`
`podSecurityContext` | The security context for the container | `{fsGroup: 1000, runAsNonRoot: true, runAsUser: 1000, runAsGroup: 1000}`
`containerSecurityContext` | The security context for the pod | `{readOnlyRootFilesystem: true, privileged: false, allowPrivilegeEscalation: false}`
`resources` | Pod resource requests and limits | `{}`
`nodeSelector` | Node labels for pod assignment | `{}`
`tolerations` | Node taints to tolerate | `[]`
`affinity` | Pod affinity| `{}`
`watchNamespace`| A comma-separated list of namespaces to watch. Leave empty to watch all namespaces. Ignored if `rbac.clusterScoped` is set to `false` | ``
`secret.existingSecret` | The name of an existing secret to use | ``
`secret.labels` | Additional labels for the secret | ``
`secret.annotations` | Annotations for the seret | ``
`secret.stringData` | If set, a secret is created that should configure access to the cloud provider's KMS | `{}`
`secret.mountPath` | If set, the secret is mounted at this path. If left empty, environment variables are created from the secret. | `""`
`env` | A list of additional environment variables | `[]`

## Configuring Cloud Provider Credentials

The SOPS Operator Helm chart allows for fairly flexible credentials configuration.

The following examples show how SOPS Operator can be configured for AWS, Azure, and GCP using credentials configured via secret.
It should also be possible to use managed identities, but this has not been tested yet.
Whatever SOPS supports should be possible.
Please refer to the [SOPS documentation](https://github.com/mozilla/sops) to find out what is required for your cloud provider.

### AWS

**Using Environment Variables**

If `secret.mountPath` is not set, environment variables are created from the secret.

```yaml
secret:
  stringData:
    AWS_SECRET_ACCESS_KEY=AKIA0123456789ABCDEF
    AWS_SECRET_ACCESS_KEY=random+access+key+0123456789abcdef+12345
```

**Using a Credentials File**

If `secret.mountPath` is set, the secret is mounted to the specified path.
If necessary, environment variables, such as `AWS_DEFAULT_REGION` or `AWS_SHARED_CREDENTIALS_FILE`, can be configured.

```yaml
secret:
  mountPath: /home/sops-operator/.aws
  stringData:
    credentials: |
      [default]
      aws_access_key_id = AKIA0123456789ABCDEF
      aws_secret_access_key = random+access+key+0123456789abcdef+12345

env:
  - name: AWS_DEFAULT_REGION
    value: eu-central-1
```

### Azure

```yaml
secret:
  stringData:
    AZURE_TENANT_ID: 12345678-1234-5678-1234-12345678
    AZURE_CLIENT_ID: 12345678-1234-5678-1234-1234567890ab
    AZURE_CLIENT_SECRET: 12345678-1234-5678-1234-1234567890ab
```

### GCP

```yaml
secret:
  mountPath: /home/sops-operator/.google/credentials.json
  stringData: |
    {
      "type": "service_account",
      "project_id": "myproject",
      "private_key_id": "1234567890123456789012345678901234567890",
      "private_key": "-----BEGIN PRIVATE KEY....==\n-----END PRIVATE KEY-----\n",
      "client_email": "sops-operator@myproject.iam.gserviceaccount.com",
      "client_id": "123456789012345678901",
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/sops-operator@myproject.iam.gserviceaccount.com"
    }

env:
  - name: GOOGLE_APPLICATION_CREDENTIALS
    value: /home/sops-operator/.google/credentials.json
```
