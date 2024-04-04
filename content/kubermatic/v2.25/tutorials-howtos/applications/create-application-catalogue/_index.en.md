+++
title = "Creating An Application Catalogue"
date =  2022-08-03T16:27:43+02:00
weight = 1
+++

This guide targets KKP Admins and details adding Applications to a catalogue, so they can be installed by Cluster Admins.
Create permissions on the KKP master cluster are required to complete it.

Before an Application is available for install, its installation- and metadata need to be added to the KKP Master Cluster. From the master, they will be automatically replicated to all KKP Seed Clusters.

![Example of a populated catalogue](/img/kubermatic/v2.25/applications/default-applications-catalog.png "Example of a populated catalogue")

To organize its catalogue, KKP makes use of a Custom Kubernetes Resource Type called `ApplicationDefinition`. This ensures Kubernetes-native management and full GitOps compatibility.
Additionally this mechanism can be used to ensure that only approved Applications can be deployed into a cluster.

## Creating an ApplicationDefinition

An ApplicationDefinition represents a single Application and contains all of its versions.
Each ApplicationDefinition maps to one Application in the UI.
Currently ApplicationDefinitions need to be created by hand, but we plan to include a UI importer in an upcoming release.

```yaml
# Example of an ApplicationDefinition
apiVersion: apps.kubermatic.k8c.io/v1
kind: ApplicationDefinition
metadata:
  name: prometheus
spec:
  description: Prometheus is a monitoring system and time series database.
  method: helm
  versions:
  - template:
      source:
        helm:
          chartName: prometheus
          chartVersion: 15.10.4
          url: https://prometheus-community.github.io/helm-charts
    version: 2.36.2
  - template:
      source:
        git:
          path: charts/prometheus
          ref:
            branch: master
          remote: https://github.com/prometheus-community/helm-charts
    version: 0.0.0-dev
```

The example defines the application `prometheus` with two versions. To learn more about the ApplicationDefinition's configuration please refer to the [ApplicationDefinition Reference]({{< ref "../../../architecture/concept/kkp-concepts/applications/application-definition" >}})

### Applying the ApplicationDefinition

After creating an ApplicationDefinition file, you can simply apply it using kubectl in the KKP master cluster.

```sh
# Inside KKP master
kubectl apply -f my-appdef.yml
```

A KKP controller will afterwards ensure that your definition is synced to all KKP Seed Clusters. The application automatically be available ont the dashboard.

## ApplicationDefinition Customizations

In addition, it is possible to provide useful metadata about an application. This includes the applications upstream source-code, documentation link, or a logo. These fields will be displayed to ClusterAdmins, when interacting with Applications:

![ApplicationDefinition Customizations](appdef-customizations.png "ApplicationDefinition Customizations")

```yaml
spec:
  description: cert-manager is a Kubernetes addon to automate the management and issuance
    of TLS certificates from various issuing sources.
  documentationURL: "https://cert-manager.io/"
  sourceURL: "https://github.com/cert-manager/cert-manager"
  logoFormat: png
  logo: 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
```

In order to base64 encode the logo, we recommend the usage of the standard base64 package:

```sh
base64 mylogo.svg
```

## Edit Application Catalogue

Please refer to [Add or Remove an  Application Version]({{< ref "../add-remove-application-version/" >}})