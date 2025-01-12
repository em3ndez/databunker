## Terraform script to create AWS infrastructure for Databunker

Terraform is a powerful tool to manage infrastructure with configuration files rather than through a graphical user interface.

We use Terraform scripts to streamline the installation of the following infrastructure elements:

1. VPC
1. MySQL RDS
1. Elastic Kubernetes Service (EKS)
1. Security groups to allow connectivity
1. Generate random password for secure MySQL RDS access and save it as Kubernetes secret using the following resource path: **databunker-mysql-rds/db-password** 

### ⚡ How to set up everything

Run the following commands:
```
terraform init
terraform apply
```

Make sure to save the database hostname displayed as **rds_hostname** variable.

Same MYSQL RDS **hostname** is printed using the following command:
```
terraform output rds_hostname
```

### ☕ Next steps
1. Set **KUBECONFIG** environment variable to point to a newly generated config file for Kubernetes
1. Create an SSL certificate for Databunker service and save it as Kubernetes secret
1. Add Databunker charts repository using **helm** command
1. Start Databunker process using **helm** command

```
export KUBECONFIG=`pwd`/`ls -1 kubeconfig_*`
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=localhost"
kubectl create secret tls databunkertls --key="tls.key" --cert="tls.crt"
helm repo add databunker https://databunker.org/charts/
helm repo update
helm install databunker databunker/databunker --set mariadb.enabled=false \
  --set externalDatabase.host=MYSQL-RDS-HOST \
  --set externalDatabase.existingSecret=databunker-mysql-rds \
  --set certificates.customCertificate.certificateSecret=databunkertls
```

The **MYSQL-RDS-HOST** is the same as ```terraform output rds_hostname```.


### 🔍 View generated database password
```
terraform output rds_password
```

### ⚙️ Troubleshooting
```
terraform destroy -target aws_eks_cluster.yuli-cluster
terraform destroy -target module.eks.aws_eks_cluster.this\[0\]
terraform destroy
helm uninstall databunker
kubectl get secret databunkertls -o json
kubectl get secret databunker-mysql-rds -o json
```
