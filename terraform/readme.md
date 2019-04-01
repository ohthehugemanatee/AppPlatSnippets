# A collection of useful terraform templates

### More verbose logging

```
TF_LOG=1 tf <command>
```

### Using helm provider after deploying AKS

In the same template, after the `resource "azurerm_kubernetes_cluster"`, you can directly install helm and install charts like this ([source](https://www.terraform.io/docs/providers/helm/release.html)):

```
provider helm {
  kubernetes {
    host                   = "${azurerm_kubernetes_cluster.k8s.kube_config.0.host}"
    client_certificate     = "${base64decode(azurerm_kubernetes_cluster.k8s.kube_config.0.client_certificate)}"
    client_key             = "${base64decode(azurerm_kubernetes_cluster.k8s.kube_config.0.client_key)}"
    cluster_ca_certificate = "${base64decode(azurerm_kubernetes_cluster.k8s.kube_config.0.cluster_ca_certificate)}"
  }
  service_account="${kubernetes_service_account.helm_service_account.metadata.0.name}"
  namespace = "${kubernetes_namespace.helm_namespace.metadata.0.name}"
  install_tiller  = true
  # TF does not use latest version of Helm which has a fix for this
  override        = ["spec.template.spec.automountserviceaccounttoken=true"]
}

data "helm_repository" "stable" {
    name = "stable"
    url  = "https://kubernetes-charts.storage.googleapis.com"
}

resource "helm_release" "example" {
  name       = "my-redis-release"
  repository = "{data.helm_repository.stable.metadata.0.name}"
  chart      = "redis"
  version    = "6.0.1"

  values = [
    "${file("values.yaml")}"
  ]

  set {
    name  = "cluster.enabled"
    value = "true"
  }
}
```

### AKS+Kubent+CustomVNET in Terraform

The only way possible is to use the `local_exec` resource as indicated in this [issue](https://github.com/Azure/AKS/issues/400#issuecomment-395584964).
