# IBM Storage Data Protect on Azure how-to
This guide will help you install IBM Defender Data Protect in Azure and it aims to get a Backup endpoint in Azure that connect seamlessly to the 


# Create a basic linux install and prepare for setup
We will install the IBM Defender Data Protect using setup tools crea

```
sudo ./cohesity_azure_setup startup_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=chsty1761258242326
sudo ./cohesity_azure_setup shutdown_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=chsty1761258242326



./iris_cli --output=prettyjson --username=admin --password=C0h3s1tyUs3r! --server=10.0.0.7 --skip_password_prompt=true --skip_force_password_change=true cluster cloud-create name=cluster-1761258242206 node-ips=10.0.0.7 subnet-gateway=10.0.0.1 subnet-mask=255.255.255.0 dns-server-ips=8.8.8.8 ntp-servers=2.almalinux.pool.ntp.org domain-names=fusion.guru hostname=cohesity metadata-fault-tolerance=0 enable-software-encryption=false cluster-size=small
```
