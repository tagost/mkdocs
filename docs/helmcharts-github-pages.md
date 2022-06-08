# Crear repositorio helmcharts
```bash
git clone https://github.com/tagost/helmcharts.git

cd helmcharts

helm create testchart

helm install --dry-run debug .
```

## Editar los templetes de acuerdo a tu aplicación
    helm package testchart/

    mkdir charts

    mv testchart-0.1.0.tgz charts/

    helm repo index .

    git add .
    git commit -m "add testchart"
    git push origin main

    helm repo add myrepo https://tagost.github.io/helmcharts/

    helm repo list

    helm repo update

    helm search repo myrepo

    helm install myrepo/testchart 

    helm uninstall testchart

### add dependency 
### Now we will add the dependency section in the Charts.yaml file

cat >> testchart/Chart.yaml <<EOF

dependencies:
  - name: redis
    version: 12.7.x
    repository: https://charts.bitnami.com/bitnami
EOF

helm dependency update ./testchart

helm dependency list ./testchart


helm package testchart/
mv testchart-0.1.0.tgz charts/

helm show values myrepo/testchart



