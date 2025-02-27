# Check Point CloudGuard Network Security Management Server Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Management Server into an existing VPC.

These types of Terraform resources are supported:
* [AWS Instance](https://www.terraform.io/docs/providers/aws/r/instance.html)
* [Security Group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [Network interface](https://www.terraform.io/docs/providers/aws/r/network_interface.html)
* [EIP](https://www.terraform.io/docs/providers/aws/r/eip.html) - conditional creation
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation

See the [Security Management Server with CloudGuard for AWS](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk130372) for additional information

This solution uses the following modules:
- /terraform/aws/modules/amis
- /terraform/aws/cme-iam-role

## Configurations

The **main.tf** file includes the following provider configuration block used to configure the credentials for the authentication with AWS, as well as a default region for your resources:
```
provider "aws" {
  region = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```
The provider credentials can be provided either as static credentials or as [Environment Variables](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#environment-variables).
- Static credentials can be provided by adding an access_key and secret_key in /terraform/aws/management/**terraform.tfvars** file as follows:
```
region     = "us-east-1"
access_key = "my-access-key"
secret_key = "my-secret-key"
```
- In case the Static credentials are used, perform modifications described below:<br/>
  a. The next lines in main.tf file, in the provider aws resource, need to be commented for sub-module /terraform/aws/cme-iam-role:
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
  //  region = var.region
  //  access_key = var.access_key
  //  secret_key = var.secret_key
  }
  ```
  b. The next lines in main.tf file, in the provider aws resource, need to be commented for sub-module /terraform/aws/cme-iam-role:
  ```
  provider "aws" {
  //  region = var.region
  //  access_key = var.access_key
  //  secret_key = var.secret_key
  }
  ```

## Usage
- Fill all variables in the /terraform/aws/management/**terraform.tfvars** file with proper values (see below for variables descriptions).
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

- Variables are configured in /terraform/aws/management/**terraform.tfvars** file as follows:

  ```
    //PLEASE refer to README.md for accepted values FOR THE VARIABLES BELOW
    
    // --- VPC Network Configuration ---
    vpc_id = "vpc-12345678"
    subnet_id = "subnet-abc123"
    
    // --- EC2 Instances Configuration ---
    management_name = "CP-Management-tf"
    management_instance_type = "m5.xlarge"
    key_name = "privatekey"
    allocate_and_associate_eip = true
    volume_size = 100
    volume_encryption = "alias/aws/ebs"
    enable_instance_connect = false
    instance_tags = {
    key1 = "value1"
    key2 = "value2"
    }
    
    // --- IAM Permissions ---
    iam_permissions = "Create with read permissions"
    predefined_role = ""
    sts_roles = []
    
    // --- Check Point Settings ---
    management_version = "R81-BYOL"
    admin_shell = "/bin/bash"
    management_password_hash = "12345678"
    
    // --- Security Management Server Settings ---
    management_hostname = "mgmt-tf"
    is_primary_management = "true"
    SICKey = ""
    allow_upload_download = "true"
    gateway_management = "Locally managed"
    admin_cidr = "0.0.0.0/0"
    gateway_addresses = "0.0.0.0/0"
    primary_ntp = ""
    secondary_ntp = ""
    management_bootstrap_script = "echo 'this is bootstrap script' > /home/admin/testfile.txt"
  ```

- Conditional creation
    - To create an Elastic IP and associate it to the Management instance:
  ```
  allocate_and_associate_eip = true
  ```
  - To create IAM Role:
  ```
  iam_permissions = "Create with read permissions" | "Create with read-write permissions" | "Create with assume role permissions (specify an STS role ARN)"
  ```
- To tear down your resources:
    ```
    terraform destroy
    ```

## Inputs
| Name          | Description   | Type          | Allowed values | Default       | Required      |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| vpc_id        | The VPC id in which to deploy | string    | n/a   | n/a   | yes |
| subnet_id     | To access the instance from the internet, make sure the subnet has a route to the internet | string    | n/a   | n/a   | yes  |
| management_name | (Optional) The name tag of the Security Management instance  | string    | n/a   | Check-Point-Management-tf  | no  |
| management_instance_type | The instance type of the Security Management Server  | string  | - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.12xlarge <br/> - m5.24xlarge | m5.xlarge  | no  |
| key_name | The EC2 Key Pair name to allow SSH access to the instance | string  | n/a | n/a | yes |
| allocate_and_associate_eip | If set to true, an elastic IP will be allocated and associated with the launched instance | bool  | true/false | true | yes |
| volume_size  | Root volume size (GB) - minimum 100  | number  | n/a  | 100  | no  |
| volume_encryption  | KMS or CMK key Identifier: Use key ID, alias or ARN. Key alias should be prefixed with 'alias/' (e.g. for KMS default alias 'aws/ebs' - insert 'alias/aws/ebs')  | string  | n/a  | alias/aws/ebs  | no  |
| enable_instance_connect  | Enable SSH connection over AWS web console. Supporting regions can be found [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/)  | bool  | true/false  | false  | no  |
| instance_tags  | (Optional) A map of tags as key=value pairs. All tags will be added to the Management EC2 Instance  | map(string)  | n/a  | {}  | no  |
| iam_permissions  | IAM role to attach to the instance profile  | string  | - None (configure later) <br/> - Use existing (specify an existing IAM role name) <br/> - Create with assume role permissions (specify an STS role ARN) <br/> - Create with read permissions <br/> - Create with read-write permissions  | Create with read permissions  | no  |
| predefined_role  | (Optional) A predefined IAM role to attach to the instance profile. Ignored if var.iam_permissions is not set to 'Use existing'  | string  | n/a  | ""  | no  |
| sts_roles  | (Optional) The IAM role will be able to assume these STS Roles (list of ARNs). Ignored if var.iam_permissions is set to 'None' or 'Use existing'  | list(string)  | n/a  | []  | no  |
| management_version  | Management version and license  | string  | - R80.40-BYOL <br/> - R80.40-PAYG <br/> - R81-BYOL <br/> - R81-PAYG <br/> - R81.10-BYOL <br/> - R81.10-PAYG | R81-BYOL | no  |
| admin_shell  | Set the admin shell to enable advanced command line configuration  | string  | - /etc/cli.sh <br/> - /bin/bash <br/> - /bin/csh <br/> - /bin/tcsh | /etc/cli.sh | no |
| management_password_hash | (Optional) Admin user's password hash (use command "openssl passwd -6 PASSWORD" to get the PASSWORD's hash) | string | n/a | "" | no |
| management_hostname  | (Optional) Security Management Server prompt hostname  | string  | n/a  | ""  | no  |
| is_primary_management  | Determines if this is the primary management server or not  | bool  | true/false  | true  | no  |
| SICKey  | Mandatory only when deploying a secondary Management Server, the Secure Internal Communication key creates trusted connections between Check Point components. Choose a random string consisting of at least 8 alphanumeric characters  | string  | n/a  | ""  | no  |
| allow_upload_download | Automatically download Blade Contracts and other important data. Improve product experience by sending data to Check Point | bool | true/false | true | no |
| gateway_management  | Select 'Over the internet' if any of the gateways you wish to manage are not directly accessed via their private IP address  | string  | - Locally managed <br/> - Over the internet  | Locally managed  | no  |
| admin_cidr  | (CIDR) Allow web, ssh, and graphical clients only from this network to communicate with the Security Management Server  | string  | valid CIDR  | 0.0.0.0/0  | no  |
| gateway_addresses  | (CIDR) Allow gateways only from this network to communicate with the Security Management Server  | string  | valid CIDR  | 0.0.0.0/0  | no  |
| primary_ntp  | (Optional) The IPv4 addresses of Network Time Protocol primary server | string  | n/a  | 169.254.169.123  | no  |
| secondary_ntp  | (Optional) The IPv4 addresses of Network Time Protocol secondary server  | string  | n/a  | 0.pool.ntp.org  | no  |
| management_bootstrap_script | (Optional) Semicolon (;) separated commands to run on the initial boot | string | n/a | "" | no |


## Outputs
| Name  | Description |
| ------------- | ------------- |
| management_instance_id  | The deployed Security Management Server AWS instance id  |
| management_instance_name  | The deployed Security Management AWS instance name  |
| management_instance_tags  | The deployed Security Management Server AWS tags  |
| management_public_ip  | The deployed Security Management Server AWS public ip  |
| management_url  | URL to the portal of the deployed Security Management Server  |

## Revision History
In order to check the template version, please refer to [sk116585](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk116585)

| Template Version | Description   |
| ---------------- | ------------- |
| 20210309 | First release of Check Point Security Management Server Terraform module for AWS |
| 20210329 | Stability fixes |



## License

This project is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details
