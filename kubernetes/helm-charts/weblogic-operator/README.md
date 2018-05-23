# Weblogic Operator Helm Chart
This chart creates a Weblogic operator on a [Kubernetes](https://kubernetes.io/) cluster using the 
[Helm](https://github.com/kubernetes/helm) package manager.

## Prerequisites
- Kubernetes 1.7.5+ with Beta APIs enabled

- Install [Helm](https://github.com/kubernetes/helm#install)

- Initialize Helm by installing Tiller, the server portion of Helm, to your Kubernetes cluster

```bash
helm init
```

## Creating a Namespace 
We recommend that you create a namespace to run your WebLogic Operator in.

To create a namespace named `my-namespace`:

```bash
kubectl create namespace my-namespace
```

## Clone the git weblogic-operator Repository
Clone the git weblogic-operator repo:

```bash
git clone https://github.com/oracle/weblogic-kubernetes-operator.git
```

## Creating Certificates and Keys
Change directory to the cloned git weblogic-operator repo:

```bash
cd weblogic-operator/kubernetes/helm-charts
```

To create an Operator that does not allow external clients to call the Operator's
REST API, use:
```bash
create-helm-certificates.sh NAMESPACE
```

To create an operator that allows external clients to call the Operator's REST API using
TLS/SSL security on an SSL port that uses a generated self-signed X5.09 certificate as
its identity, use:
```bash
create-helm-certificates.sh NAMESPACE SUBJECT_ALTERNATIVE_NAMES
```

To create an operator that allows external clients to call the Operator's REST API using
TLS/SSL security on an SSL port that uses a customer supplied X.509 certificate as its
identity, use:
```bash
create-helm-certificates.sh NAMESPACE CERTFILE KEYFILE
```

The parameters are as follows: 

- `NAMESPACE` (required) is the Namespace to run the Operator in.
- `SUBJECT_ALTERNATIVE_NAMES` (optional) is the list subject alternative names that will
be put in the generated self-signed certificate that is used as the Operator's identity,
for example: DNS:myhost,DNS:localhost,IP:127.0.0.1.  It should be set to the DNS
hostname and/or IP address of the Kubernetes Master/API Server that the external client
uses.  If the client is outside of a firewall, then make sure to use the externally
visible hostname of the Kubernetes Server.
- `CERTFILE` (optional) is the pathname of a PEM file containing the X.509 certificate
that you want to use as the Operator's identity.
- `KEYFILE` (optional) is the pathname of a PEM file containing the private key that
matches `CERTFILE`.

This script will generate a version of values.yaml with the encoded values for certificates and keys
required by the Weblogic Operator.  You can then change the generated values.yaml file as needed.

## Installing the Chart

To install the chart with the release `my-release` and namespace `my-namespace`:

```bash
helm install weblogic-operator --name my-release --namespace my-namespace 
```
The command deploys weblogic-operator on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists
the parameters that can be configured during installation.

> Note: if you do not pass the --name flag, a release name will be auto-generated. You can view releases by running helm list (or helm ls, for short).


## Uninstalling the Chart
To uninstall/delete the `my-release` deployment:

```bash
helm delete --purge my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration 
|  Key                           |  Description                      |  Default              |
| -------------------------------|-----------------------------------|-----------------------|
| elkIntegrationEnabled | Controls whether or not ELK integration is enabled | false |
| externalDebugHttpPort | The node port that should be allocated for the Kubernetes cluster for the operator's Java remote debug server. This parameter is required if 'remoteDebugNodePortEnabled' is true | 30999 |
| externalOperatorCert | The customer supplied certificate to use for the external operator REST https interface.  The value must be a string containing a base64 encoded PEM certificate. Value can be generated by the create-helm-certificates.sh script. This value is optional | |
| externalOperatorKey | The customer supplied private key to use for the external operator REST https interface.  The value must be a string containing a base64 encoded PEM key. Value can be generated by the create-helm-certificates.sh script. This value is optional | |
| externalRestHttpsPort | The node port that should be allocated for the external operator REST https interface. This value is required if the 'externalOperatorCert' and 'externalOperatorKey' values are set. | 31001 |
| image | The docker image containing the operator code | wlsldi-v2.docker.oraclecorp.com/weblogic-operator:latest |
| imagePullPolicy | The image pull policy for the operator docker image | IfNotPresent |
| internalOperatorCert | Internal certificate for Weblogic operator. Value is generated by the create-helm-certificates.sh script | |
| internalOperatorKey | Internal key for Weblogic operator. Value is generated by the create-helm-certificates.sh script | |
| internalDebugHttpPort | The port number inside the Kubernetes cluster for the operator's Java remote debug server. This parameter is required if 'remoteDebugNodePortEnabled' is true | 30999 |
| javaLoggingLevel | The level of Java logging that should be enabled in the operator. Valid values are: "SEVERE", "WARNING", "INFO", "CONFIG", "FINE", "FINER", and "FINEST" | INFO |
| remoteDebugNodePortEnabled | Controls whether or not the operator will start a Java remote debug server on the provided port and will suspend execution until a remote debugger has attached. | false |
| serviceAccount | The name of the service account that the operator will use to make requests to the Kubernetes API server | weblogic-operator |
| targetNamespaces | A comma separated list of target namespaces the operator manages. Each listed namespace must be created prior to install of this helm chart | default |
 
Specify parameters to override default values using the `--set key=value[,key=value]` argument to helm install. For example:

```bash
helm install weblogic-operator --name my-release --namespace my-namespace --set elkIntegrationEnabled=true
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example:

```bash
helm install weblogic-operator --name my-release --namespace my-namespace --values values.yaml
```