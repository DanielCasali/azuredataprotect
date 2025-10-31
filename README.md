# IBM Storage Data Protect on Azure how-to
This guide will help you install IBM Defender Data Protect in Azure and it aims to get a Backup endpoint in Azure created by Cohesity that connect seamlessly to the IBM Defender DMS Console.
I thought I would have to create a Terraform for making the install easy but turns out it is so straightforward with the setup tool that I don't think it is worth the effort for one VM and an application ID.


## Create a basic linux install and prepare for setup
We will install the IBM Defender Data Protect endpoint using setup tools created by Cohesity. The tools provided are based on linux and a small instance will be enough, I am using 1 vCPU and 3.5 GB of memory. I also use Azure spot discount since it is a PoC and I don't mind to restart the VM in case it comes down. this Server will not be in the Data Path.

## Configure the Setup instance for the install
The install tool will expect to use the cohesity user by default and the `/home/cohesity` path as the install directory.
Create the user and change to the user:
```
[root@linux ~]# useradd cohesity
[root@linux ~]# usermod -g wheel cohesity
[root@linux ~]# su - cohesity
[cohesity@linux ~]$
```

## Download the Assets from Fix Central
The Assets are available for Download on FixCentral, we will download the version 2.0.16, so after you login to it:
1) Search for IBM Storage Defender Select Intalled Version: All and Platform All -> Click Continue 
2) Select Browse for Fixes -> Click Continue 
3) Open the Show Contained Fixes sub-bullet under group_: STGDEF_2.0.16
4) Select:
   -> STGDEF_2.0.16_5 for the Azure Cluster VHD image `cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd (34.03 GB)`
5) Also Select:
   -> STGDEF_2.0.16_15 for the Azure setup tools `installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz (134.59 MB)`
6) Scroll down to the bottom of the Page and Click continue
I personally use the HTTPS download method and I copy the download link and use wget or curl to download the asset.
7) You can download it directly on the Linux box.
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
8) Wait for it to finish or if there is bandwidth feel free to open another session and download the setup tools (mine did not as I consumed the 100MB/s I had).
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

9) Check if the youhave the files:
```
[cohesity@linux ~]$ ls -la
total 35958228
drwx------. 3 cohesity cohesity        4096 Oct 31 15:25 .
drwxr-xr-x. 5 root     root              53 Oct 31 14:29 ..
-rw-------. 1 cohesity cohesity         517 Oct 31 15:23 .bash_history
-rw-r--r--. 1 cohesity cohesity          18 Apr 30  2024 .bash_logout
-rw-r--r--. 1 cohesity cohesity         174 Oct 31 15:25 .bash_profile
-rw-r--r--. 1 cohesity cohesity         492 Apr 30  2024 .bashrc
-rw-r--r--. 1 cohesity cohesity 36537279488 Oct 31 15:21 cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd
-rw-r--r--. 1 cohesity cohesity   141131248 Oct 31 15:21 installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
```

9) Decompress the setup tool:
```
[cohesity@linux ~]$ tar -xf installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
[cohesity@linux ~]$ 
```

10) Run the installer
For some reason it does not get the path right (it adds some /home/cohesity/bin/ but the software so not there so I link it):
```
[cohesity@linux ~]$ sudo ./install.sh
/etc/bashrc: line 12: BASHRCSOURCED: unbound variable
[cohesity@linux ~]$ echo $PATH
/home/cohesity/.local/bin:/home/cohesity/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
[cohesity@linux ~]$ ls
7.2.2_u2_release-20250718_86f4ecd0
cohesity-azure-7.2.2_u2_release-20250718_86f4ecd0.vhd
cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
installer-cohesity_azure_setup-7.2.2_u2_release-20250718_86f4ecd0.tar.gz
install.sh
software
```
11) As you can see there is no bin, so I link the 7.2.2_u2_release-20250718_86f4ecd0 directory to the bin and iris_cli should be working as a it is in PATH:
```
[cohesity@linux ~]$ ln -s 7.2.2_u2_release-20250718_86f4ecd0 bin
[cohesity@linux ~]$ iris_cli
 Username:
^C
```
If you are stuck on the Username: prompt we are not using it yet, you may Control+C to interrupt the iris_cli program you will get a error dump but do not worry.

## Fix the quota for your instances.
The instance quota is normally not that high if you start for the first time, so you might want to use the az CLI to configure a higher quota, it will depend on your cluster but for this one VM. Be aware you need to Login first with `az login`.
```
az quota update --resource-name standardDSv2Family --scope /subscriptions/<subscription-id>/providers/Microsoft.Compute/locations/eastus --limit-object value=20
```

## Install the cluster.

First you need to ensure your params.json file is fine for use, here I have an example that I used with public DNS and NTP and IP address 10.0.0.7 on eastus location, be sure to change this on your parameters file too. This file is located at `/home/cohesity/software/params.json`.
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
  "cohesity_azure_domain_name": "<Your Domain>",
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

There are other fields that are possible and come on the example that you can use but I didn't:
```
  "cohesity_azure_apps_subnet": "CUSTOMER_VALUE",  -> Extra subnet to be backed up
  "cohesity_azure_apps_subnet_mask": "CUSTOMER_VALUE", -> Extra subnet mask to be backed up
  "cohesity_azure_cluster_vhd_storage_account": "CUSTOMER_VALUE", -> if you don't give this an storage account will be created for you.
  "azure_instance_deletion_protection": false, -> I just did not use this, I don't have details on what this changes.
  "cohesity_azure_vault_account_name": "CUSTOMER_VALUE (Please provide external target details in case of xlarge clusters only)", -> External vault is for xlarge
  "cohesity_azure_vault_container_name": "CUSTOMER_VALUE", -> External vault is for xlarge
  "cohesity_azure_vault_access_key": "CUSTOMER_VALUE", -> External vault is for xlarge
  "cohesity_azure_vault_ad_auth_client_id": "CUSTOMER_VALUE (Exactly one of access_key or client_id to be provided)", -> External vault is for xlarge
  "cohesity_azure_tags": -> JSON key/value pair array for tags for management

```
Make sure to be on the /home/cohesity/software/ directory:
```
[cohesity@linux ~]$ cd software
[cohesity@linux software]$ pwd
/home/cohesity/software
[cohesity@linux software]$
```

You can first validate if the installation parameters are fine:
```
[cohesity@linux software]$ sudo ./cohesity_azure_setup validate -cohesity_azure_setup_params_file=/home/cohesity/software/params.json
.
.
.
I1031 16:27:40.296535  3218 login_op.cc:362] Refreshed the access_token in connector_context
I1031 16:27:40.296833  3218 azure_connector_base_op.cc:210] Acquiring semaphore.
I1031 16:27:40.296914  3218 azure_connector_base_op.cc:197] Semaphore acquired.
I1031 16:27:40.296972  3218 azure_cluster_manager.cc:152] Progress: Sending Throttled Curl Rpc
I1031 16:27:40.604641  3218 azure_cluster_manager.cc:152] Progress: Got response for the Curl Rpc
I1031 16:27:40.604871  3218 azure_cluster_manager.cc:152] Progress: Got final response for the Curl Rpc
I1031 16:27:40.605414  3217 azure_tool.cc:1310] Successfully validated Azure credentials.
[cohesity@linux software]$
```

# The rest of the Documentation will come later.


Then Just continue with install:
```
sudo ./cohesity_azure_setup create_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json
```

If the creation has issues
```
./iris_cli --output=prettyjson --username=admin --password=admin --server=10.0.0.7 --skip_password_prompt=true --skip_force_password_change=true cluster cloud-create name=azurecluster node-ips=10.0.0.7 subnet-gateway=10.0.0.1 subnet-mask=255.255.255.0 dns-server-ips=8.8.8.8 ntp-servers=2.almalinux.pool.ntp.org domain-names=fusion.guru hostname=cohesity metadata-fault-tolerance=0 enable-software-encryption=false cluster-size=small
```
Maybe add this?
```
 -default_cluster_password string
    	Default cluster password to use.
  -default_cluster_username string
```

You can control startup and shutdown of the cluster from this place:
```
sudo cohesity_azure_setup startup_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=defender1761258242326
sudo cohesity_azure_setup shutdown_cluster -cohesity_azure_setup_params_file=/home/cohesity/software/params.json -cohesity_azure_resource_group=defender1761258242326


```
