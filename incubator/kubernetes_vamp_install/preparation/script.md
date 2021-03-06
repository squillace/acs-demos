# Azure Subscription

You will need an active azure subscription. Before proceeding with
this script ensure that you are logged in using `az login`. Assuming
you started this script with the `prep.sh` script this is a safe
assumption to make.

# Environment Setup

Since we will be creating an ACS cluster it is important that we first
setup the environment to be unique to you, otherwise we will get
naming conflicts between people running the tutorials. 

If you don't already have a local config file let's start by copying
the default config into a local config file.

```
if [ ! -f ../env.local.json ]; then cp --no-clobber ../env.json ../env.local.json; else echo "You already have a config"; fi
```

You MUST ensure the `ACS_DNS_PREFIX` is something world unique and you
`MAY` change the other settings. Once complete return here. You can
check the current contents with `cat`:

```
cat ../env.local.json
```

# Dependencies

There are a few dependencies that we must have installed for this
tutorial/demo to work:

```
sudo pip3 install virtualenv
```

# Ensuring we have a clean cluster

It's always wise to ensure that a demo starts in a clean state. To
that end we will delete any existing cluster and SSH infromation that
exists using this configuration. Don't worry if this command returns a
"could not be found" error. It just means you didn't have anything
dangling after the last demo.

```
az group delete --name $ACS_RESOURCE_GROUP --yes
sudo ssh-keygen -f "/root/.ssh/known_hosts" -R [$ACS_DNS_PREFIXmgmt.$ACS_REGION.cloudapp.azure.com]:2200
```

# Creating a Cluster

We can now create a resource group that will contain all the Azure resouces deployed by ACS.

```
az group create --name $ACS_RESOURCE_GROUP --location $ACS_REGION
```

Results:

```
{
  "id": "/subscriptions/135f79ed-bb93-4372-91f6-7b5f1c57dd81/resourceGroups/acs-kubernetes-vamp-demo",
  "location": "eastus",
  "managedBy": null,
  "name": "acs-kubernetes-vamp-demo",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

Now, we can create the cluster

```
az acs create --orchestrator-type=kubernetes --name $ACS_CLUSTER_NAME --resource-group $ACS_RESOURCE_GROUP --dns-prefix $ACS_DNS_PREFIX --generate-ssh-keys
```

Results:

```
{
  "id": "/subscriptions/135f79ed-bb93-4372-91f6-7b5f1c57dd81/resourceGroups/acs-k8s-spark-demo/providers/Microsoft.Resources/deployments/azurecli1496363170.3581209",
  "name": "azurecli1496363170.3581209",
  "properties": {
    "correlationId": "d9ac5c3f-83ec-4d30-861f-5c331f8ac40b",
    "debugSetting": null,
    "dependencies": [],
    "mode": "Incremental",
    "outputs": null,
    "parameters": {
      "clientSecret": {
        "type": "SecureString"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.ContainerService",
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiVersions": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "containerServices"
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "template": null,
    "templateLink": null,
    "timestamp": "2017-06-02T00:32:36.607132+00:00"
  },
  "resourceGroup": "acs-kubernetes-vamp-demo"
}
```

## Install the Kubernetes CLI

In order to manage this instance of ACS we will need the Kubernetes command line interface,
fortunately the Azure CLI makes it easy to install it.

```
sudo az acs kubernetes install-cli
```

Results:

```
Downloading client to /usr/local/bin/kubectl from https://storage.googleapis.com/kubernetes-release/release/v1.6.4/bin/linux/amd64/kubectl
```

## Initial Cluster Setup

First, we will load the master Kubernetes cluster configuration locally.

```
az acs kubernetes get-credentials --resource-group=$ACS_RESOURCE_GROUP --name=$ACS_CLUSTER_NAME
```

Results:

```
[No output on success]
```

Next, we will use the Kubernetes command line interface to ensure the list of machines in the cluster are available.

```
kubectl get nodes
```

Results:

```
NAME                    STATUS                     AGE       VERSION
k8s-agent-b90bd903-0    Ready                      1h        v1.5.3
k8s-agent-b90bd903-1    Ready                      1h        v1.5.3
k8s-agent-b90bd903-2    Ready                      1h        v1.5.3
k8s-master-b90bd903-0   Ready,SchedulingDisabled   1h        v1.5.3
```
