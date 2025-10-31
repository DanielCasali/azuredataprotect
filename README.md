# IBM Storage Data Protect on Azure how-to
This guide will help you install IBM Defender Data Protect in Azure and it aims to get a Backup endpoint in Azure created by Cohesity that connect seamlessly to the IBM Defender DMS Console.


# Create a basic linux install and prepare for setup
We will install the IBM Defender Data Protect endpoint using setup tools created by Cohesity. The tools provided are based on linux and a small instance will be enough, I am using 1 vCPU and 3.5 GB of memory. I also use Azure spot discount since it is a PoC and I don't mind to restart the VM in case it comes down. this Server will not be in the Data Path.

# Configure the Setup instance for the install
The install tool will expect to use the cohesity user by default and the `/home/cohesity` path as the install directory.
Create the user and change to the user:
```
[root@linux ~]# useradd cohesity
[root@linux ~]# su - cohesity
[cohesity@linux ~]$
```



# Download the Assets from Fix Central
The Assets are available for Download on FixCentral, we will download the version 2.0.16, so after you login to it:
1) Search for IBM Storage Defender Select Intalled Version: All and Platform All -> Click Continue 
2) Select Browse for Fixes -> Click Continue 
3) Open the Show Contained Fixes sub-bullet under group_: STGDEF_2.0.16
4) Select:
   -> STGDEF_2.0.16_5 for the Azure Cluster VHD image `cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd (34.03 GB)`
5) Also Select:
   -> STGDEF_2.0.16_15 for the Azure setup tools `installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz (134.59 MB)`
6) Scroll down to the bottom of the Page and Click continue
I personally use the HTTPS download method and I copy the download link and use wget or curl to download the Asset:

```
[cohesity@linux ~]$ wget https://<the link created>/cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd
--2025-10-31 15:14:19--  https://<the link created>/cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd
Resolving <Fully Qualified Domain Name> (<Fully Qualified Domain Name>)... <IP>
Connecting to <Fully Qualified Domain Name> (<Fully Qualified Domain Name>)|<IP>|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 36537279488 (34G)
Saving to: ‘cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd’

            cohesity-azur   0%[                                      ] 296.00K  1.33MB/s
```
Wait for it to finish or if there is bandwidth feel free to open another session and download the setup tools (mine did not as I consumed the 100MB/s I had).
```
[cohesity@linux ~]$ wget https://<the link created>/installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
--2025-10-31 15:14:19--  https://<the link created>/installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
Resolving <Fully Qualified Domain Name> (<Fully Qualified Domain Name>)... <IP>
Connecting to <Fully Qualified Domain Name> (<Fully Qualified Domain Name>)|<IP>|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 36537279488 (34G)
Saving to: ‘installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz’

            installer-coh   1%[                                      ] 2096.00K  2.02MB/s
```

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
