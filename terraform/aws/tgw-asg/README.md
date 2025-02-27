# Check Point CloudGuard Network Transit Gateway Auto Scaling Group Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Gateway Auto Scaling Group for Transit Gateway with an optional Management Server into an existing VPC.

These types of Terraform resources are supported:
* [AWS Instance](https://www.terraform.io/docs/providers/aws/r/instance.html)
* [Security Group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [Network interface](https://www.terraform.io/docs/providers/aws/r/network_interface.html)
* [CloudWatch Metric Alarm](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_metric_alarm)
* [EIP](https://www.terraform.io/docs/providers/aws/r/eip.html)
* [Launch configuration](https://www.terraform.io/docs/providers/aws/r/launch_configuration.html)
* [Auto Scaling Group](https://www.terraform.io/docs/providers/aws/r/autoscaling_group.html)
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation

See the [CloudGuard Network for AWS Transit Gateway R80.10 and Higher Deployment Guide](https://sc1.checkpoint.com/documents/IaaS/WebAdminGuides/EN/CP_CloudGuard_AWS_Transit_Gateway/Content/Topics-AWS-TGW-R80-10-AG/Introduction.htm) for additional information

This solution uses the following modules:
- /terraform/aws/modules/autoscale
- /terraform/aws/modules/management
- /terraform/aws/modules/cme-iam-role

## Configurations

The **main.tf** file includes the following provider configuration block used to configure the credentials for the authentication with AWS, as well as a default region for your resources:
```
provider "aws" {
    region = var.region
    access_key = var.aws_access_key_ID
    secret_key = var.aws_secret_access_key
}
```
The provider credentials can be provided either as static credentials or as [Environment Variables](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#environment-variables).
- Static credentials can be provided by adding an access_key and secret_key in /terraform/aws/tgw-asg/**terraform.tfvars** file as follows:
```
region     = "us-east-1"
access_key = "my-access-key"
secret_key = "my-secret-key"
```
- In case the Static credentials are used, perform modifications described below:<br/>
  a. The next lines in main.tf file, in the provider aws resource, need to be commented for sub-modules /terraform/aws/autoscale, /terraform/aws/modules/management and /terraform/aws/modules/cme-iam-role:
  ```
  provider "aws" {
  //  region = var.region
  //  access_key = var.access_key
  //  secret_key = var.secret_key
  }
  ```
- In case the Environment Variables are used, perform modifications described below:<br/>
  a. The next lines in main.tf file, in the provider aws resource, need to be commented:
  ```
  provider "aws" {
  //    region = var.region
  //    access_key = var.aws_access_key_ID
  //    secret_key = var.aws_secret_access_key
  }
  ```
  b. The next lines in main.tf file, in the provider aws resource, need to be commented for sub-modules /terraform/aws/autoscale, /terraform/aws/modules/management and /terraform/aws/modules/cme-iam-role:
  ```
  provider "aws" {
  //    region = var.region
  //    access_key = var.aws_access_key_ID
  //    secret_key = var.aws_secret_access_key
  }
  ```
 
## Usage
- Fill all variables in the /terraform/aws/tgw-asg/**terraform.tfvars** file with proper values (see below for variables descriptions).
- From a command line initialize the Terraform configuration directory:
    ```
    terraform init
    ```
- Create an execution plan:
    ```
    terraform plan
    ```
- Create or modify the deployment:
    ```
    terraform apply
    ```
  
- Variables are configured in /terraform/aws/tgw-asg/**terraform.tfvars** file as follows:

  ```
  //PLEASE refer to README.md for accepted values FOR THE VARIABLES BELOW
  
  // --- Network Configuration ---
  vpc_id = "vpc-12345678"
  gateways_subnets = ["subnet-123b5678", "subnet-123a4567"]
  
  // --- General Settings ---
  key_name = "privatekey"
  enable_volume_encryption = true
  enable_instance_connect = false
  allow_upload_download = true
  
  // --- Check Point CloudGuard Network Security Gateways Auto Scaling Group Configuration ---
  gateway_name = "Check-Point-gateway"
  gateway_instance_type = "c5.xlarge"
  gateways_min_group_size = 2
  gateways_max_group_size = 8
  gateway_version = "R81-BYOL"
  gateway_password_hash = "12345678"
  gateway_SICKey = ""
  enable_cloudwatch = true
  asn = "6500"
  
  // --- Check Point CloudGuard Network Security Management Server Configuration ---
  management_deploy = true
  management_instance_type = "m5.xlarge"
  management_version = "R81-BYOL"
  management_password_hash = "12345678"
  management_permissions = "Create with read-write permissions"
  management_predefined_role = ""
  gateways_blades = true
  admin_cidr = "0.0.0.0/0"
  gateways_addresses = "0.0.0.0/0"
  gateway_management = "Locally managed"
  
  // --- Automatic Provisioning with Security Management Server Settings ---
  control_gateway_over_public_or_private_address = "private"
  management_server = "management-server"
  configuration_template = "template-name"
  ```

- Conditional creation
  - To create a Security Management server with IAM Role:
  ```
  management_permissions = "Create with read permissions" | "Create with read-write permissions" | "Create with assume role permissions (specify an STS role ARN)"
  ```
  - To enable cloudwatch for ASG:
  ```
  enable_cloudwatch = true
  ```
  Note: enabling cloudwatch will automatically create IAM role with cloudwatch:PutMetricData permission
  - To deploy Security Management Server:
  ```
  management_deploy = true
  ```
- To tear down your resources:
    ```
    terraform destroy
    ```

## Inputs
| Name          | Description   | Type          | Allowed values | Default       | Required      |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| vpc_id    | Select an existing VPC | string | n/a   | n/a   | yes |
| gateways_subnets  | Select at least 2 public subnets in the VPC. If you choose to deploy a Security Management Server it will be deployed in the first subnet | list(string) | n/a | n/a  | yes |
| key_name | The EC2 Key Pair name to allow SSH access to the instances | string  | n/a | n/a | yes |
| enable_volume_encryption | Encrypt Environment instances volume with default AWS KMS key | bool | true/false | true | no |
| enable_instance_connect  | Enable SSH connection over AWS web console. Supporting regions can be found [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/) | bool  | true/false  | false  | no  |
| allow_upload_download | Automatically download Blade Contracts and other important data. Improve product experience by sending data to Check Point | bool | true/false | true | no |
| gateway_name | (Optional) The name tag of the Security Gateway instances | string | n/a | Check-Point-Gateway | no |
| gateway_instance_type | The instance type of the Security Gateways | string  | - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.18xlarge <br/> - c5n.large <br/> - c5n.xlarge <br/> - c5n.2xlarge <br/> - c5n.4xlarge <br/> - c5n.9xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.8xlarge  | c5.xlarge  | no  |
| gateways_min_group_size | The minimal number of Security Gateways | number | n/a | 2 | no |
| gateways_max_group_size | The maximal number of Security Gateways | number | n/a | 10 | no |
| gateway_version | Gateway version and license | string | - R80.40-BYOL <br/> - R80.40-PAYG-NGTP <br/> - R80.40-PAYG-NGTX <br/> - R81-BYOL <br/> - R81-PAYG-NGTP <br/> - R81-PAYG-NGTX <br/> - R81.10-BYOL <br/> - R81.10-PAYG-NGTP <br/> - R81.10-PAYG-NGTX | R81-BYOL | no |
| gateway_password_hash | (Optional) Admin user's password hash (use command 'openssl passwd -6 PASSWORD' to get the PASSWORD's hash) | string | n/a | "" | no |
| gateway_SIC_Key | The Secure Internal Communication key for trusted connection between Check Point components. Choose a random string consisting of at least 8 alphanumeric characters | string | n/a | n/a | yes |
| enable_cloudwatch  | Report Check Point specific CloudWatch metrics | bool  | true/false  | false  | no  |
| asn | The organization Autonomous System Number (ASN) that identifies the routing domain for the Security Gateways | string | n/a | 6500 | no |
| management_deploy  | Select 'false' to use an existing Security Management Server or to deploy one later and to ignore the other parameters of this section | bool  | true/false  | true  | no  |
| management_instance_type | The EC2 instance type of the Security Management Server  | string  | - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.12xlarge <br/> - m5.24xlarge  | m5.xlarge  | no  |
| management_version  | The license to install on the Security Management Server  | string  | - R80.40-BYOL <br/> - R80.40-PAYG <br/> - R81-BYOL <br/> - R81-PAYG <br/> - R81.10-BYOL <br/> - R81.10-PAYG | R81-BYOL  | no  |
| management_password_hash | (Optional) Admin user's password hash (use command 'openssl passwd -6 PASSWORD' to get the PASSWORD's hash) | string | n/a | "" | no |
| management_permissions  | IAM role to attach to the instance profile  | string  | - None (configure later) <br/> - Use existing (specify an existing IAM role name) <br/> - Create with assume role permissions (specify an STS role ARN) <br/> - Create with read permissions <br/> - Create with read-write permissions  | Create with read-write permissions  | no  |
| management_predefined_role  | ((Optional) A predefined IAM role to attach to the instance profile. Ignored if IAM role is not set to 'Use existing'  | string  | n/a  | ""  | no  |
| gateways_blades | Turn on the Intrusion Prevention System, Application Control, Anti-Virus and Anti-Bot Blades (additional Blades can be manually turned on later) | bool | true/false | true | no |
| admin_cidr  | (CIDR) Allow web, ssh, and graphical clients only from this network to communicate with the Management Server  | string  | valid CIDR  | n/a  | no  |
| gateway_addresses  | Allow gateways only from this network to communicate with the Security Management Server  | string  | valid CIDR | n/a | no  |
| gateway_management  | Select 'Over the internet' if any of the gateways you wish to manage are not directly accessed via their private IP address  | string  | - Locally managed <br/> - Over the internet | Locally managed  | no  |
| control_gateway_over_public_or_private_address  | Determines if the gateways are provisioned using their private or public address  | string  | - private <br/> - public  | private  | no  |
| management_server | (Optional) The name that represents the Security Management Server in the automatic provisioning configuration | string | n/a | management-server | no |
| configuration_template |(Optional) A name of a Security Gateway configuration template in the automatic provisioning configuration | string | n/a | TGW-ASG-configuration | no |


## Outputs
| Name  | Description |
| ------------- | ------------- |
| management_instance_name  | The deployed Security Management AWS instance name |
| management_public_ip  | The deployed Security Management Server AWS public ip  |
| management_url  | URL to the portal of the deployed Security Management Server  |
| autoscaling_group_name  | The name of the deployed AutoScaling Group  |
| configuration_template  | The name that represents the configuration template. Configurations required to automatically provision the Gateways in the Auto Scaling Group, such as what Security Policy to install and which Blades to enable, will be placed under this template name  |
| controller_name  | The name that represents the controller. Configurations required to connect to your AWS environment, such as credentials and regions, will be placed under this controller name  |

## Revision History
In order to check the template version, please refer to [sk116585](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk116585)

| Template Version | Description   |
| ---------------- | ------------- |
| 20210329 | First release of Check Point Transit Gateway Auto Scaling Group Terraform module for AWS |



## License

This project is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details
