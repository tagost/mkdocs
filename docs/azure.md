### Implementación de un conjunto de escalado en Azure Portal

code cloud-init.yaml

`cloud-config`

package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /var/www/html/index.html
    content: |
        Hello world from Virtual Machine Scale Set !
runcmd:
  - service nginx restart
  
  
  az group create \
  --location westus \
  --name scalesetrg
  
  az vmss create \
  --resource-group scalesetrg \
  --name webServerScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.yaml \
  --admin-username azureuser \
  --generate-ssh-keys
  
#MONITOREO DEL LOAD BALANCER
az network lb probe create \
  --lb-name webServerScaleSetLB \
  --resource-group scalesetrg \
  --name webServerHealth \
  --port 80 \
  --protocol Http \
  --path /

az network lb rule create \
  --resource-group scalesetrg \
  --name webServerLoadBalancerRuleWeb \
  --lb-name webServerScaleSetLB \
  --probe-name webServerHealth \
  --backend-pool-name webServerScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp


#escalado manual
az vmss scale \
    --name MyVMScaleSet \
    --resource-group MyResourceGroup \
    --new-capacity 6


 az group delete \
      --name scalesetrg \
      --yes