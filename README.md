# IBM Storage Data Protect on Azure how-to
This guide will help you install IBM Defender Data Protect in Azure and it aims to get a Backup endpoint in Azure created by Cohesity that connect seamlessly to the IBM Defender DMS Console.


# Create a basic linux install and prepare for setup
We will install the IBM Defender Data Protect endpoint using setup tools created by Cohesity. The tools provided are based on linux and a small linux instance will 


# Download the Assets from Fix Central
The Assets are available for Download on FixCentral, after you login to Fixcentral:
1) 

-> 
-> 

```
az quota update --resource-name standardDSv2Family --scope /subscriptions/<subscription-id>/providers/Microsoft.Compute/locations/eastus --limit-object value=20
```

```
{
  "azure_application_id" : "<Your Application ID>",
  "azure_application_key" : "<Your Application key>",
  "azure_tenant_id" : "<Your Tenant ID>",
  "azure_subscription_id" : "<Your Subscription ID>",
  "azure_is_subscription_us_gov_cloud" : false,
  "cohesity_azure_cluster_resource_group_name" : "defender",
  "cohesity_azure_cluster_storage_account_prefix" : "defender",
  "cohesity_azure_vm_name_prefix" : "defender",
  "cohesity_azure_virtual_network_resource_group_name" : "support_group",
  "cohesity_azure_virtual_network_name" : "support-vnet",
  "cohesity_azure_virtual_network_subnet_name" : "default",
  "cohesity_azure_cluster_location": "eastus",
  "cohesity_azure_host_name": "defender",
  "cohesity_azure_domain_name": "fusion.guru",
  "cohesity_azure_ntp_servers": "2.almalinux.pool.ntp.org",
  "cohesity_azure_dns_server": "8.8.8.8",
  "cohesity_setup_tool_dir_full_path": "/home/cohesity/software",
  "cohesity_setup_templates_dir_full_path": "/home/cohesity/software/conf",
  "cohesity_azure_vhd_file_path": "/home/cohesity/cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd",
  "cohesity_azure_num_vms": 1,
  "cohesity_azure_vm_ip_addresses": "10.0.0.7",
  "cohesity_azure_vm_type": "Standard_DS5_v2",
  "cohesity_azure_num_vms_per_storage_account": 8,
  "cohesity_azure_num_vms_per_resource_group": 64,
  "cohesity_azure_enable_cluster_encryption": false,
  "cohesity_azure_ce_in_existing_resource_group": false,
  "cohesity_azure_cluster_name": "azurecluster",
  "cohesity_azure_cluster_size": "small"
}
```








```
sudo ./cohesity_azure_setup startup_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=chsty1761258242326
sudo ./cohesity_azure_setup shutdown_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=chsty1761258242326



./iris_cli --output=prettyjson --username=admin --password=admin --server=10.0.0.7 --skip_password_prompt=true --skip_force_password_change=true cluster cloud-create name=azurecluster node-ips=10.0.0.7 subnet-gateway=10.0.0.1 subnet-mask=255.255.255.0 dns-server-ips=8.8.8.8 ntp-servers=2.almalinux.pool.ntp.org domain-names=fusion.guru hostname=cohesity metadata-fault-tolerance=0 enable-software-encryption=false cluster-size=small
```
