 # Site-to-Site VPN Connection Between Azure and AWS

 ## Architecture Diagram:

<img width="985" height="333" alt="image" src="https://github.com/user-attachments/assets/597252be-f9c6-4d17-9d72-c269d93eb642" />

 ### Step 1: Configuring Azure

 1. Crate a resource group on Azure to deploy the resources on that
 ```bash
 Resource Group  : rg-azure-aws
 Region          : East US
 ```
 2. Create Virtual Network (VNet).
     
    Portal → Virtual networks → Create
 ```bash
 Resource Group Name       : rg-azure-aws
 Region                    : East-US
 VNet Name                 : vnet-azure
 VNet IPv4 Address Space   : 172.10.0.0/16
 Subnet Name               : subnet-01
 Subnet IPv4 Address Space : 172.10.1.0/24
 ```
 3. Create VPN Gateway (Azure).
    
    Portal → Virtual network gateways → Create
 ```bash
 VPN Gateway Name          : vpn-azure-aws
 Region                    : East-US
 Gateway Type              : VPN
 VPN type                  : Route-based
 SKU                       : VpnGw1
 Generation                : Generation 1
 Virtual Network           : vnet-azure
 Public IP Address         : pip-vpn-azure-aws
 Public IP Address Type    : Basic
 Assignment                : Dynamic
 Enable active-active mode : Disabled
 Configure BGP             : Disabled
 ```
 ### Step 2: AWS Configuration

 4. Create the Virtual Private Cloud (VPC) in AWS.
    
    VPC → Your VPCs → Create VPC
 ```bash
 Name                      : my-vpc-01
 IPv4 CIDR                 : 10.10.0.0/16
 ```
 5. Create a subnet inside the VPC (Virtual Network).
    
    VPC → Subnets → Create subnet
 ```bash
 Name                      : my-subnet-01
 VPC Name                  : my-vpc-01
 Subnet IPv4 CIDR          : 10.10.1.0/24
 AZ                        : Any
 ```
 6. Create a customer gateway pointing to the Public IP Address of Azure VPN Gateway.
    
    VPC → Customer Gateways → Create customer gateway
 ```bash
Name            : cgw-aws-azure
IP address      : Azure VPN Gateway Public IPAzurePIP
Routing         : Static
 Rest keep everything as default
 ```
 7. Create the Virtual Private Gateway then attach to the VPC.
    
    VPC → Virtual Private Gateways → Create
 ```bash
 Name               : vgw-aws-azure
 Actions → Attach to VPC → my-vpc-01
 ```
 8. Create a site-to-site VPN Connection.
    
    VPC → Site-to-Site VPN Connections → Create
 ```bash
 Name                      : vpn-aws-azure
 Target gateway type       : Virtual private gateway (Select your Virtual private gateway created)
 Customer gateway          : Existing (Select your VCustomer gateway created)
 Routing options           : Static
 Static IP prefixes        : 172.10.1.0/24
 Leave rest of them as default
 ```
 9. Download the configuration file.
     
    VPN Connections → vpn-aws-azure → Download configuration
 ```bash
 Vendor                    : Generic
 Platform                  : Generic
 Software                  : Vendor Agnostic
 IKE Version               : ikev1
 In this configuration file you will note that there are the Shared Keys and the Public Ip Address for each of one of the two IPSec tunnels created by AWS.
 ```
 ### Part 3: Connecting Azure ↔ AWS

 10. Create the Local Network Gateway in Azure.
     
     Portal → Local network gateways → Create
     
 ```bash
 Name                  : lng-azure-aws
 Resource Group Name   : rg-azure-aws
 Region                : East-US
 IP address            : Get the Outside IP address from the configuration file downloaded(VPG)
 Address Space(s)      : 10.10.0.0/16
 ```
 11. Create the connection on the Virtual Network Gateway in Azure.
     
     VPN Gateway → vpn-azure-aws → Connections → Add
 ```bash
 Name                  : connection-azure-aws
 Connection Type       : Site-to-Site
 Local Network Gateway : Select the Local Network Gateway which you created.
 Shared Key            : Get the Pre-Shared Key from Aws the configuration file downloaded.
 Wait till the Connection Status changes to - Connected
 In the same way, check in AWS Console wheather the 1st tunnel of Virtual Private Gateway UP.
 ```
 12. Create Internet Gateway and Attach it to VPC in AWS.
     
     VPC → Internet Gateways → Create
 ```bash
 Name                  : my-internet-gateway
 ```
 13. Now let's edit the route table associated with our VPC.
     
     VPC → Route Tables → Select main route table
 ```bash
 Add the route to Azure subnet through the Virtual Private Gateway
 Destination          : 172.10.1.0/24
 Target               : Virtual Private Gateway that we created.
 also add,
 Destination          : 0.0.0.0/0
 Target               : Internet Gateway that we created.
 ```
14. Create VMs in both Azure and AWS and Test the connection
 ```bash
   Test for Azure 
   ping          : 10.10.1.x
   Test for AWS
   ping          : 172.10.1.x
 ```
15. Create SG in Aws test the connection.
    
    SG → Added route → ICMP for ping and Target is Azure Vnet CIDR.


