## VPC migration script input:

- [discovery.yaml example](#discoveryyaml-example)

- [Input parameters](#migration-input-parameters)

### discovery.yaml example

```
 # aws:
 #   s3:
 #     account: "955315316192"
 #     role_name: "aviatrix-role-app"
 #     name: "aviatrix-discovery-misgration"
 #     region: "us-east-1"
 terraform:
   terraform_output: "/home/bednarsk/Nike-migration/migration/discovery_output"
   terraform_version: ">= 0.14"
   aviatrix_provider: "= 2.19.5"
   aws_provider: "~> 3.43.0"
   # enable_s3_backend: False              ## generate the basic s3 terraform resource
   # module_source: "../abc"               ## use a module with a different name or location
   # module_name: "abc"                    ## define the name of the module to be generated under terraform_output folder.
   # tf_cloud:                             ## terraform cloud integration
   #   organization: "aviatrix-ps-test"    ## organization is mandatory
   #   workspace_name: "terraform-cli-ws"  ## either workspace_name or tags is allowed
   #   tags: ["abc"]                       ## This information will be written to the cloud block in terraform
 aviatrix:
   controller_ip: "35.81.225.204"
   controller_account: "955315316192"
   # tf_controller_access:                ## Default to "ENV". Available options are "SSM" or "ENV"
   #   mode: "SSM"                        ## When using ENV, the rest of the attributes are not needed.
   #   alias: "us_west_2"
   #   region: "us-west-2"
   #   username: "admin"
   #   password_store: "avx-admin-password"
   #   ssm_role: ""                       ## Default to "".  If defined with a role, generate assume_role statmeent in provider for SSM access, otherwise, no assume_role statement is generated.
   ## onboard an account with aviatrix-role-ec2 and aviatrix-role-app. These roles need to exist
   ## in the controller account
   # ctrl_role_app: "aviatrix-role-app"
   # ctrl_role_ec2: "aviatrix-role-ec2"
   ## Assign separate role to the gateway if required. These roles need to exist in the spoke account
   # gateway_role_app: "aviatrix-role-app"
   # gateway_role_ec2: "aviatrix-role-ec2"
 # alert:
 #   expect_dest_cidrs: ["0.0.0.0/0"]        ## Expected route in the route table. Use quad zero to disable alert.
 #   expect_vpc_prefixes: ["0.0.0.0/0"]    ## prefix in VPC.
 #   check_gateway_existence: True         ## ## Check if a spoke gateway has been deployed
 # config:
 #   import_resources: True                ## Determine whether we should import existing igw and subnets
 #   add_vpc_cidr: True                      ## Determine whether we should generate terraform code for adding VPC cidr.
 #   filter_tgw_attachment_subnet: False   ## Default is False, i.e., include tgw attachment subnet in the migration
 #   allow_vpc_cidrs: ["10.0.0.0/8"]       ## Only allowed VPC cidr will be passed to the browfield spoke vpc module.
 #   subnet_tags:
 #     - key: 'Networking-Created-Resource'
 #       value: 'Do-Not-Delete-Networking-Created-Resource'
 #   route_table_tags:
 #     - key: 'Aviatrix-Created-Resource'
 #       value: 'Do-Not-Delete-Aviatrix-Created-Resource'
 #   configure_gw_name: True              ## Determine whether we should generate terraform code for accepting your spoke_gw_name and transit_gw_name inputs.
 #   configure_spoke_advertisement: False  ## Determine whether we should generate terraform code for accepting your spoke_advertisement input
 #   enable_spoke_egress: False
 #   snat_policies:
 #      - src_ip: "100.64.0.0/10"
 #        connection: "$transit_name"
 #        new_src_ip: "$private_ip"
 #        protocol: "all"
 tgw:
   tgw_account: "955315316192"             ## TGW account, set it to controller account if not using TGW.
   tgw_role: "aviatrix-role-app"           ## permission for accessing TGW account. set it to aviatrix-role-app if not using TGW.
   # tgw_by_region:
   #   us-east-1: "tgw-04067720ba42d1f13"
   #   us-west-2: "tgw-0011223374c528899"
 account_info:
   - account_id: "347477146224"
     account_name: "AwsDevOps"              ## empty string ("") implies default name "aws-<account_id>"
     # onboard_account: False               ## Determine whether we should generate terraform resource for onboarding the account_name
     # role_name: "aviatrix-role-app"       ## permission for accessing this account
     # spoke_gw_size: "t3.micro"
     # filter_cidrs: []                     ## Default is no filter
     # hpe: True
     # managed_tgw: False
     # enable_spoke_egress: False
     regions:
       - region: "us-east-1"
         vpcs:
           - vpc_id: "vpc-0f12325c153807493"
             avtx_cidr: "10.1.0.0/25"
             spoke_gw_name: "spoke1"
             transit_gw_name: "transit-gw-us-east"
             # gw_zones: []
             # spoke_gw_tags:
             #   - key: "SnowAppName"
             #     value: "Aviatrix"
             #   - key: "SnowAppCode"
             #     value: "Aviatrix"
             # domain: "prod"                 ## Default is None
             # encrypt: False
             # inspection: False
             # spoke_advertisement: ["1.1.1.1/32"]        ## Default is None
             # spoke_routes: ["0.0.0.0/1","128.0.0.0/1]   ## Default is None
           - vpc_id: "vpc-0b622852b31c1662e"
             avtx_cidr: "10.2.0.0/25"
             spoke_gw_name: "spoke2"
             transit_gw_name: "transit-gw-us-east"
           - vpc_id: "vpc-01f793f283fd3951c"
             avtx_cidr: "10.3.0.0/25"
             spoke_gw_name: "spoke3"
             transit_gw_name: "transit-gw-us-east"
 # switch_traffic:
 #   delete_tgw_route_delay: 5
 # cleanup:
 #   vpc_cidrs: ["100.64.0.0/25","100.127.0.0/25"]   ## VPC cidrs to be deleted
 #   resources: ["VGW", "NAT", "TGW_ATTACHMENT"]
```

- Above is an example that show the minimal required YAML configuration for AWS.
- Optional attributes are commented out with a single pound (#) character.
- All the optional boolean attributes are shown with its default values if omitted.
- Double pound (##) characters marks the beginning of a comment.

### Migration input parameters

| Field              |  Type         | Required | Description                                      |
| :-----------       |:------------- | :------- | :--------------------------------------------    |
| aws:               |               |   No    | Use AWS S3 to backup the generated account folder. Omit this section if S3 backup is not required. |
| s3:                |               |         | Setup s3 for storing the terraform output files  |
| account:           | string        |         | s3 bucket account number                         |
| role_name:         | string        |         | s3 bucket access permission                      |
| name:              | string        |         | s3 bucket name                                   |
| region:            | string        |         | s3 bucket region                                 |
|                    |               |          |                                                  |
| terraform:         |               |   Yes    | Mark the beginning of terraform info             |
| terraform_output   | string        |   Yes    | Absolute path to the TF files created            |
| terraform_version  | string        |   Yes    | Version string in terraform version syntax       |
| aviatrix_provider  | string        |   Yes    | Version string in terraform version syntax       |
| aws_provider       | string        |   Yes    | Version string in terraform version syntax       |
| enable_s3_backend  | bool          |   No     | Generate terraform S3 backend config. False by default if this attribute is omitted |
| module_source      | string        |   No     | Override the module source in vpc.tf. Default is "../module_aws_brownfield_spoke_vpc" if omitted  |
| module_name        | string        |   No     | Define the name of the module generated under terraform_output folder. Default is "module_aws_brownfield_spoke_vpc". |
| tf_cloud:          |               |   No     | terraform cloud integration                      |
| organization       | string        |   Yes    | This is a mandatory attribute for terraform cloud integration |
| workspace_name     | string        |   Yes    | Either workspace_name or tags should be defined, not both.  |
| tags               | List of string|   Yes    | This information will be used to define the cloud block in terraform |
|                    |               |          |                                                  |
| aviatrix:          |               |   Yes    | Generate terraform resource for onboarding an Aviatrix account.              |
| controller_ip      | string        |   Yes    | Aviatrix controller IP address                   |
| controller_account | string        |   Yes    | The AWS Account # where the controller resides   |
|                    |               |          |                                                  |
|tf_controller_access|               |   No     | Mark the beginning of tf_controller_access inside terraform section. See [tf_controller_access](#tf-controller-access) for details. |
| mode               | string        |   No     | Default to "ENV". Available options are "SSM" and "ENV" |
| alias              | string        |   No     | Default to "us_west_2". Used only in SSM mode.   |
| region             | string        |   No     | Default to "us-west-2". Used only in SSM mode.   |
| password_store     | string        |   No     | Default to "avx-admin-password". Used only in SSM mode. |
| username           | string        |   No     | Default to "admin".  Used only in SSM mode.      |
| ssm_role           | string        |   No     | Default to "". Used only in SSM mode. See [tf_controller_access](#tf-controller-access) for details.|
|                    |               |          |                                                  |
| ctrl_role_app      | string        |   No     | Controller app role name to be used in the SPOKE account. Default is aviatrix-role-app. |
| ctrl_role_ec2      | string        |   No    | Name of the role associated with the controller EC2. Default is aviatrix-role-ec2. |
| gateway_role_app   | string        |   No     | Gateway role name. Default is aviatrix-role-app. |
| gateway_role_ec2   | string        |   No     | Gateway instance profile name. Default is aviatrix-role-ec2. |
|                    |               |          |                                                  |
| alert:             |               |   No     | Mark beginning of alert                          |
| expect_dest_cidrs  | list          |   No     | Alert IP not fall within the given CIDR list.  Default is ["0.0.0.0/0"], turning off this alert. |
| expect_vpc_prefixes| list          |   No     | Alert VPC prefix not fall within given CIDR ranges. Default is ["0.0.0.0/0"], turning off this alert.|
| check_gateway_existence | bool     |   No     | True by defaut if omitted. Check if a spoke gateway has been deployed in the VPC. |
|                    |               |          |                                                  |
| config:            |               |   No     | Mark beginning of script feature config          |
| import_resources   | bool          |   No     | Default is True. Determine whether we should import existing IGW and subnets into terraform. See [import_resources](#import-resources) for additional details.|
| add_vpc_cidr       | bool          |   No     | Default is True. Determine whether we should generate terraform code for adding VPC cidr.|
| filter_tgw_attachment_subnet| bool |   No     | Enable tgw attachment subnet filtering.  Set to True if we should not migrate subnets used by TGW vpc attachement. Set to False if omitted. See [filter_tgw_attachment](#filter-tgw-attachment) for additional details. |
| allow_vpc_cidrs    | list          |   No     | List of allowed VPC CIDRs. Only the allowed CIDRs will be copied to vpc_cidr and passed to the brownfield spoke VPC terraform module. Default is ["0.0.0.0/0"], allowing any CIDR. |
| subnet_tags        | list          |   No     | List of tags to be added to the subnet(s). Omit this section if no tags required.   |
| key                | String        |   No     |  name of the tag                                 |
| value              | String        |   No     |  value of the tag                                |
| route_table_tags   | list          |   No     | List of tags to be added to the route table(s). Omit this section if no tags required.   |
| key                | String        |   No     |  name of the tag                                 |
| value              | String        |   No     |  value of the tag                                |
| configure_gw_name  | bool          |   No     | Default is True.  Determine whether we should generate terraform code for accepting your spoke_gw_name and transit_gw_name inputs. |
| configure_spoke_advertisement| bool|   No     | Default is False. Determine whether we should generate terraform code for accepting your spoke_advertisement input. |
| enable_spoke_egress| bool          |   No     | Default is False. Must be set to True to enable spoke egress and configure spoke gateway with snat_policies. See [enable_spoke_egress](#enable-spoke-egress) for additional details. |
| snat_policies:     |               |   No     | Mark the beginning of snat policies section, followed by a list of policies.  Each policy is made up of the attributes from the Customized SNAT API policy_list. See [snat_policies](#snat-policies) for additional details. |
|   src_ip           | String        |   Yes    |  Source ip address range  |
|   connection       | String        |   Yes    |  "None" or "transit_name" to choose the transit gw as the connection |
|   new_src_ip       | String        |   Yes    |  Snat ip address. It can be an IP address or "private_ip" to use the spoke gateway private IP address     |
|   protocol         | String        |   Yes    |  all                                             |
|                    |               |          |                                                  |
| tgw:               |               |   Yes    | List out all the TGWs used, assuming all TGWs are defining within one account. If no tgw is used, set config/filter_tgw_attachment_subnet to False, and set tgw_acount and tgw_role to the account and role used by the controller; tgw_by_region can be omitted. |
| tgw_account        | string        |   Yes    | TGW account number                               |
| tgw_role           | string        |   Yes    | TGW account access role                          |
| tgw_by_region:     |               |   No    | Mark beginning of tgw_by_region object, defining region and tgw_id pair |
| us-east-1          | string        |   No     | tgw_id in region us-east-1                       |
| us-east-2          | string        |   No     | tgw_id in region us-east-2                       |
|    ...             |  ...          |   ...    |         ...                                      |
| eu-north-1         | string        |   No     | tgw_id in region eu-north-1                      |
|                    |               |          |                                                  |
| account_info:      |               |   Yes    | Spoke VPC info                                   |
| account_id         | string        |   Yes    | AWS Account #                                    |
| account_name       | string        |   Yes    | account name in the controller                   |
| onboard_account    | bool          |   No     | Default is False, i.e., account already existed in controller. |
| role_name          | string        |   No     | IAM role assumed to execute API calls. Default is aviatrix-role-app.|
| spoke_gw_size      | string        |   No     | Spoke gateway instance size. Default is t3.micro. |
| filter_cidrs       | list(str)     |   No     | Filters out any route within specified CIDR when copying the route table.  No need to add RFC1918 routes in the list; they are filtered by default.  Default is empty list [], i.e., no filtering required. |
| hpe                | bool          |   No     | Spoke gateway setting (True/False). Default is True.|
| managed_tgw        | bool          |   No     | Default is False. See [managed_tgw](#managed-tgw) for additional details. |
| enable_spoke_egress| bool          |   No     | This attribute is availabe at both account level and vpc level. Default is False if omitted. |
|                    |               |          |                                                  |
| regions:           |               |   Yes    | Mark the beginning of region list                |
| region             | string        |   Yes    | AWS region                                       |
|                    |               |          |                                                  |
| vpcs:              |               |   Yes    | Mark the beginning of vpc list                   |
| vpc_id             | string        |   Yes    | VPC IDs to be migrated                           |
| avtx_cidr          | string        |   Yes    | set avtx_cidr in vpc-id.tf                       |
| gw_zones           | list ["a",...]|    No    | Zone letters to deploy spoke gateways in.  Discovery will deduce the zones if an empty list [] is used. See [gw_zones](#gw-zones) for additional details. |
| spoke_gw_tags      |               |    No    | mark beginning of a list, following by a list of key and value object|
|  key               | String        |   No     |  name of the tag                                 |
|  value             | String        |   No     |  value of the tag                                |
| domain             |  string       |   No     | Default is None.                                 |
| encrypt            |  bool         |   No     | Default is False.  No volume encryption          |
| inspection         |  bool         |   No     | Default is False.  No inpsection required        |
| spoke_advertisement| list(cidr str)|   No     | Default is using the defaut spoke advertisement  |
| spoke_routes      | list(cidr str)|   No     | Default is using default RFC1918 routes programmed by the controller.|
| spoke_gw_name      | string        |  Yes      | Not required if configure_gw_name is set to False     |
| transit_gw_name    | string        |  Yes      | Not required if configure_gw_name is set to False     |
|                    |               |                                                             |
| switch_traffic:    |               |   No    | Mark the beginning of switch_traffic config      |
| delete_tgw_route_delay| integer    |   No     | specifiy the delay between spoke-gw-advertize-cidr and tgw-route-removal. Default is 5 second.|
|                    |               |          |                                                  |
| cleanup:           |               |   No    | Mark the beginning of cleanup list               |
| vpc_cidrs          | list          |   No    | CIDR's to be deleted from VPC                    |
| resources          | list ["VGW"]  |   No    | Delete resources like VGW in a VPC. Available options are [ "TGW_ATTACHMENT", "VGW", "NAT", "INTERNAL_LB"]. VIF is implicitly deleted when deleting VGW.|
| report_only        | bool          |   No    | Set to False by default. It is used for debugging purpose only. When set to True, report the resources to be deleted into tmp/vpc-id.resource without deleting the resources. |

  <a id="tf-controller-access"></a>
  **tf_controller_access**: This is a subsection inside terraform config. Two modes are supported:

  - SSM. Terraform will retrieve controller password from AWS SSM. The attributes under tf_controller_access are used to configure the provider and data access for SSM. To grant SSM permission, one can set ssm_role to a role with the ssm:GetParameter permission, e.g., aviatrix-role-app.  If defined, an assume_role statement is generated in the terraform AWS provider; otherwise, no assume_role statement is generated.
  - ENV. Use environment variables AVIATRIX_USERNAME and AVIATRIX_PASSWORD to pass controller credential into terraform.

  <a id="gw-zones"></a>
  **gw_zones**: If **gw_zones** is not specified, gw_zones with the most private subnets will be used.

  <a id="managed-tgw"></a>
  **managed_tgw**: The **managed_tgw** flag has an impact to switch_traffic and cleanup as follows:

  1. When managed_tgw is True, the options --rm_static_route and --rm_propogated_route are ignored in switch_traffic.  Instead, switch_traffic will update the spoke gw adverstised cidr to use more specific cidrs.  Basically, each VPC cidr will be splited into multiple specific cidrs and joined together as the spoke gw advertized cidr.  As as result, routes pointing to the spoke gw will be preferred over the ones pointing to the managed tgw.  That is, no need to remove the propagated route or static route in the tgw route table.

  2. Cleanup script will call controller API to first detach the vpc from tgw  and  then update  the spoke gw advertized cidr to use original VPC cidrs, removing the more specific cidrs.

  <a id="filter-tgw-attachment"></a>
  **filter_tgw_attachment**:  You might have used empty subnets for AWS TGW attachment purpose; in which case, those subnets should not be migrated and can be deleted. Setting filter_tgw_attachment to True will prevent tgw-attachment subnets from being migrated at dm.discovery and will delete them at dm.cleanup.

  <a id="enable-spoke-egress"></a>
  **enable_spoke_egress**:  When enable_spoke_egress is True, route to NAT gateway and quad-zero route to non IGW are not copied.

  <a id="snat-policies"></a>
  **snat_policies**: Each policy is made up of the attributes from the Customized SNAT API policy_list.  The following attributes are supported:

      src_ip: "100.64.0.0/10"
      connection: "$transit_name"
      new_src_ip: "$private_ip"
      protocol: "all"
  
  1. In this example, "100.64.0.0./10" and "all" are string constant while a string starting with a dollar sign represents a gateway parameter that needs to be lookup.  Two parameters are currently supported: $transit_name is the name of transit gateway to which the spoke gateway is attached. $private_ip is the private IP address of the spoke gateway. They are retrieved from the spoke gateway using the list_gateway_info API.

  2. dm.discovery generates the SNAT policies in the terraform brownfield main module, but SNAT policy is not staged until dm.switch_traffic.

  <a id="import-resources"></a>
  **import_resources**: When import_resources is False, dm.discovery will NOT generate the associated terraform resources for import existing IGW and subnets.  The step for importing IGW and subnets (**source terraform-import-subnets.sh**) can be skipped because NO terraform-import-subnets.sh is generated.  **tmp/terraform-undo-import-subnets.sh** does not exist either.
  
  When import_resources is True, dm.discovery generates the terraform resources for importing existing IGW and subnets of the VPC.  They are imported using the step **source terraform-import-subnets.sh**.  In this case, the imported IGW reference is passed to the brownfield modules. This imposes a dependence between the brownfield module and the imported IGW resource.  This also means that we cannot remove Aviatrix resources using **terraform destroy** without **deleting the imported IGW** together.
  Here is an example how the igw_id is setup and passed into the brownfield module:

  ```
  module "vpc-04de3d319307cc5e9" {
    source         = "../abc"
    vpc_name       = "test-vpc"
    vpc_id         = "vpc-04de3d319307cc5e9"
    vpc_cidr       = ["10.90.0.0/16", "10.92.0.0/16", "11.90.0.0/16"]
    igw_id         = aws_internet_gateway.igw-vpc-04de3d319307cc5e9.id
    spoke_gw_name  = "spk1-gw"
    transit_gw     = "aws-use1-transit-gw"
    avtx_cidr      = "11.90.2.0/24"
    hpe            = false
    avtx_gw_size   = "t3.micro"
    region         = "us-east-1"
    gw_zones       = ["us-east-1a", "us-east-1b"]
    enable_spoke_egress = true
    account_name   = "AWSOpsTeam"
    route_tables   = var.route_tables_vpc-04de3d319307cc5e9
    tags           = {"SnowAppName"= "Aviatrix", "SnowAppCode"= "Aviatrix", "cis.asm.vm.riskException"= "RK0026082"}
    role_arn       = local.role_arn
    switch_traffic = false
  
    providers = { aws = aws.spoke_us_east_1 }
  }

  resource "aws_internet_gateway" "igw-vpc-04de3d319307cc5e9" {
    vpc_id = "vpc-04de3d319307cc5e9"
    tags = {"name2"= "value2", "name1"= "value1"}
    provider = aws.spoke_us_east_1
    lifecycle {
      prevent_destroy = true
    }
  }
  ```

- The reference aws_internet_gateway.igw-vpc-04de3d319307cc5e9.id imposes a dependence between the brownfield module and the aws_internet_gateway igw-vpc-04de3d319307cc5e9 resource. 
- The aws_internet_gateway igw-vpc-04de3d319307cc5e9 resource cannot be unimported first if we ever want to remove the staged Aviatrix resources using **terraform destroy**, or else the destroy will fail and complain about the missing reference.  To workaround the IGW reference dependence, we can comment out the igw_id attribute in the associated vpc-id.tf before running terraform destroy.  The igw_id attribute will then default to "" (empty string) in the brownfield module and will pass the terraform check. 
- The shell script **tmp/dmhelp.sh** defines the two functions **cigwid** and **rigwid** that can be used to comment out the igw_id attributes (in the associated vpc-id.tf) and restore them after.  That is, for **terraform destroy**, do the followings in the account folder:
  ```
  source tmp/dmhelp.sh
  cigwid
  terraform destroy
  rigwid
  ```
