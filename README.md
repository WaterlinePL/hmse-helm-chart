# HMSE helm chart

This is the Kubernetes branch of Hydrus-Modflow-Synergy-Engine frontend.

## Installation/Deploying HMSE

In order to deploy the Kubernetes version of HMSE:

#### 1. Deploy NFS provisioner
Deploy NFS provisioner (such as [nfs-ganesha-server-and-external-provisioner](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner/tree/master/charts/nfs-server-provisioner)).
It is important to configure NFS provisioning with ReadWriteMany access mode and set it as default StorageClass. For
the referenced provisioner desired setup can be achieved as follows:

---
./values.yaml

```yaml
persistence:
  accessMode: ReadWriteMany
  size: 10Gi    # suggested not going below 2Gi
storageClass:
  defaultClass: true
```

bash:

```
helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
helm install nfs-provisioner nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner --values ./values.yaml
```

---

#### 2. Configure values
Configure values present in
   the [values.yaml](https://github.com/WaterlinePL/hmse-helm-chart/blob/main/hmse/values.yaml):

**Required:**

* airflow.airflow.variables:
    * update preprocessing value `s3-secret-name` (if created S3 secret from previous point under different name)
    * hmseWebserver.s3:
        * region - S3 region
        * bucket - S3 bucket for storing project data
        * interface - S3 interface type (S3/MinIO)
        * secret - S3 secret details
          * accessKey - S3 access key
          * secretKey - S3 secret key
          * endpoint - S3 interface endpoint (e.g. used by MinIO)

**Optional:**

* airflow.airflow.users:
    * username (for security reasons)
    * password (for security reasons)
* hmseWebserver.ingress.prefix - prefix used for ingress if needed

For more configuration options reference values.yaml files present in charts and the charts themselves
[here](https://github.com/WaterlinePL/hmse-helm-chart/blob/main/hmse/).

#### 3. Deploy HMSE chart
Add repo with HMSE chart to helm:
```
helm repo add hmse-repo https://waterlinepl.github.io/hmse-helm-chart/charts/
```

Deploy HMSE chart in the k8s cluster:
```
helm install hmse hmse-repo/hmse -n <desired namespace> --values <your Values.yaml file>
```

**Important!**
Note that this setup is simplified, as Airflow should be configured with secrets for its components and may not be safe 
to use in a production environment. 

## Development

### Creating new chart version
Edit Chart.yaml in `hmse` directory and update chart version. Then execute commands:
```
helm package hmse -d charts/
helm repo index charts/
```
After pushing to main branch the new version will be deployed automatically.