# TERRAFORM
* Infrastructure as code is about managing and provisioning computer data center resources through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

## PROBLEMS WITH TRADITIONAL IT INFRASTRUCTURE DEPLOYMENTS
* Slow deployments
* Expensive
* Limited automation
* Human error
* Inconsistencies

## TYPES OF IAC TOOLS
* Configuration managenent: Ansible, Puppet, Salt Stack
    - Install and manage software
    - Maintains standard structure
    - Version control
    - Idempotent
* Server templating: Docker, Vagrant, Packer
    - Creation of a custom image of a virtual machine or a container
    - Pre installed dependencies and softwares
    - Virtual machine or docker images
    - Immutable infrastructure
* Provisioning tools: Terraform, Cloudformation
    - Deploy immutable infrastructure resources
    - Servers, databases, Networks, Components etc
    - Multiple providers

## WHY TERRAFORM ?
* Can handle multiple cloud platforms, using providers
* HCL - declarative language (.tf)
* Resource lifecycle managenent (provisioning, configuring, decommissioning)
* Terraform state is the blueprint of what is deployed by terraform, it can read attributes of existing infrastructure components by configuring data sources
* Terraform can import other resources inside it if any were created manually
* A block contains information about the infrastructure platform and resources within the platform that we want to create

## HCL BASICS
```tcl
# SYNTAX
BLOCK_TYPE "RESOURCE_TYPE" "RESOURCE_NAME" {
    PARAMETERS
}
```

> [!NOTE]
> ### CREATING A LOCAL FILE
> ```tcl
> resource "local_file" "pet" {
>     filename = "/root/pets.txt"
>     content = "HUH"
> }
>
> resource "local_sensitive_file" "secret" { # store sensitive data in a file and do not print sensitive info when we run terraform plan
>     filename = "/root/data.txt"
>     content = "HUH"
> }
> ```

### TERRAFORM WORKFLOW

| COMMAND | EFFECT |
| ------- | ------ |
| `terraform init` | Installs plugins and initializes different backends |
| `terraform plan` | See execution plan made by terraform |
| `terraform apply` | Apply the execution plan |
| `terraform destroy` | Destroy all the infrastructure |

> [!TIP]
> #### UPDATE RESOURCE
> * Change the config and run `terraform plan` and then run `terraform apply`

## TERRAFORM BASICS
### PROVIDERS
* AWS, GCP, Azure, etc
* Official providers, partners, community
* Providers are downloaded as hidden directory in the configuration directory
* If we change config and use a new provider that is not installed, the `terraform apply` command won't work as the provider is not installed, we first need to install it using `terraform init`

> [!NOTE]
> #### PROVIDER VERSIONS
> * Sometimes there is need of a specific version of a provider
> * See more for specific versions of providers in the use provider section
> * Use provider versions like `"> 1.2.0, 2.0.0, !=1.4.0, ~> 1.3.0"` to get a specific range of the version of the provider
>
> * Provider full names follow the convention `HOSTNAME/ORGANIZATIONAL_NAMESPACE/PLUGIN_NAME: version = "~> VERSION"`, where the `HOST_NAME` is the hostname of the plugin 
>
> #### USE SPECIFIC VERSION OF THE PLUGINS
> ```tcl
> terraform {
>     required_providers {
>         PROVIDER_NAME = {
>             source = "SOURCE"
>             version = "VERSION"
>         }
>     }
> }
> ```
> * `~> x.x.x` means any version less than `x.x+1.0` and greater than or equal to `x.x.x`

> [!TIP]
> #### MULTI REGION PROVIDERS
> * Some times in the script we need to work with multiple regions in aws
> * Multiple AWS providers of `aws` type with different regions can be created and called them according to usage
> ```tcl
> provider "aws" {
>     region = "us-east-1"
>     alias = "aws_us"
> }
>
> provider "aws" {
>     region = "ap-south-1"
>     alias = "aws_mumbai"
> }
>
> # CREATE EC2 IN US
> resource "aws_instance" "testing_nfs" {
>     ami = "AMI"
>     instance_type = "INSTANCE_TYPE"
>     provider = aws.aws_us #PROVIDER FORMAT IS "PROVIDER_NAME.ALIAS_NAME"
> }
>
> # CREATE EC2 IN MUMBAI
> resource "aws_instance" "testing_nfs" {
>     ami = "AMI"
>     instance_type = "INSTANCE_TYPE"
>     provider = aws.aws_mumbai #PROVIDER FORMAT IS "PROVIDER_NAME.ALIAS_NAME"
> }
> ```

> [!TIP]
> #### DEPENDENCY LOCK FILE (`terraform.lock.hcl`)
> * It helps managing externally provided dependencies
> * Ensures all providers are being used in the same version accross all the environments and operations 
> * It provides:
>     - Consistency
>     - Reproducibility
>     - Compatibility
> * It contains:
>     - Exact provider versions
>     - Provider checksums
>     - Information
> * To update the state lock file when upgrading to new versions of providers, run `terraform init -upgrade`
> * Always commit the state lock file


### CONFIGURATION DIRECTORY
* Terraform considers any file with `.tf` config as terraform in the project directory
* Resources can be defined by multiple files but all resources should be in the `main.tf` file
* There are other files too for their own purposes, named: `variables.tf`, `outputs.tf`, `providers.tf` etc

### VARIABLES FILE (`variable.tf`)
* Store dynamic variables for configurations
* If we do not provide any default value to a variable or override it programatically, it prompts while creating resource

> [!IMPORTANT]
> ```tcl
> variable "VARIABLE_NAME" {
>     default = VALUE
>     description = "DESCRIPTION"
>     type = TYPE
>     sensitive = SENSITIVITY
>     validation {
>         condition = CONDITION
>         error_message = "ERROR_MESSAGE"
>     }
> }
> ```
> * `default` parameter contains the default/value of the variable
> * `description` is just a description
> * `type` is the type of data we are storing
> * `sensitive` to define the sensitivity of the data, this decides wheather this variable is visible or not in the terraform state file
>     - If `true`: not visible in the state file
>     - If `false`: visible in the state file
> * `validation is to make sure that the variable is following a convention or have a right value and many more`
> * Even the output resource is not able to print the sensitive info/variable, but at the same time, we have a sensitive clause for output resource too which is just to avoid any mishaps

* Variable block has 3 properties, available types are : `bool`, `string`, `number`, `list`, `map`, `object`, `tuple` and `any` 
```tcl
# variables.tf
variable "VARIABLE_NAME" {
    default = 2
    type = number
    description = "DESCRIPTION"
}

variable "VARIABLE_NAME" {
    default = true
    type = bool
}

variable "VARIABLE_NAME" {
    default = "DESCRIPTION"
    type = string
}

variable "VARIABLE_NAME" {
    default = 9
    type = number
}

variable "VARIABLE_NAME" {
    default = ["STR1", "STR2", "STR3", "STR4", "STR5"]
    type = list(string)
}

variable "VARIABLE_NAME" {
    default = {
        "KEY1" = "VAL1"
        "KEY2" = 12
    }
    type = map
}

variable "VARIABLE_NAME" {
    default = {
        "KEY1" = 23
        "KEY2" = 0
    }
    type = map(number)
}

variable "VARIABLE_NAME" {
    type = object({
        name = string
        color = string
        age = number
        food = list
        healthy = bool
    })
    default = {
        name = "NAME"
        color = string
        age = 123
        food = ["FOOD1", "FOOD2"]
        healthy = true
    }
}

variable "VARIABLE_NAME" {
    default = [1, true, "STRING"]
    type = tuple([number, bool, string])
}

# main.tf
resource local_file "qwe" {
    filename = "/qwe/asd.txt"
    content = var.VARIABLE_NAME["KEY1"]
}
```

> [!NOTE]
> #### VARIABLES AS ENVIRONMENT VARIABLES
> * In this case we can export variables in shell `export TF_VAR_VARNAME="VALUE"` and then run the apply or plan command
> * Or we can give variables in run time `terraform apply -var "var1=val1" -var "var2=val2"`

> [!NOTE]
> #### .tfvars and .tfvars.json FILE
> * We can define variables in a `.tfvars` file in following format
> ```tcl
> var1 = "val1"
> var2 = "2"
> seperator = "."
> ```
>
> * Then use command like `terraform apply -var-file variables.tfvars`
> * To make this file automatically loadable by terraform save it as `*.auto.tfvars` or `*.auto.tfvars.json`

> [!IMPORTANT]
> **Variable Precedence:** `main.tf > -var or -var-file > *.auto.tfvars (alphabetical order) > terraform.tfvars > environment variables`

> [!NOTE]
> Command `terraform show` to get variable states and properties

### RESOURCE DEPENDENCIES
* If output of one variable `A` is required by another variable `B` then `B` is dependent on `A`
* While creating a resource, terraform creates `A` forst then `B`, but deletes in reverse order. This is called inplicit dependency
* But dependency can also specify external dependency
```tcl
resource "RESOURCE_TYPE" "DEPENDENT_RESOURCE_NAME" {
    ...
    depends_on = [random_pet.my-pet]
}

resource "RESOURCE_NAME" "RESOURCE_NAME" {
    ...
}
```

### OUTPUT VARIABLES
* We can print variables on screen
* Even the output resource is not able to print the sensitive info/variable, but at the same time, we have a sensitive clause for output resource too which is just to avoid any mishaps
```tcl
output "OUTPUT_NAME" {
    value = var.SECRET_VARIABLE_NAME
    sensitive = true
}
```

```tcl
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
}

output "OUTPUT_NAME" {
    value = RESOURCE_TYPE.RESOURCE_NAME.ATTRIBUTE
    description = "DESCRIPTION"
}
```
> [!TIP]
> * Commands like these can also be used to print output on screen
>     - `terraform output`
>     - `terraform output VARIABLE_NAME`
> * Sensitive output variables can be seen using these commands

> [!NOTE]
> #### EXAMPLES
>
> * For checking that a variable that we are defining is starting with `ami-` or not
> ```tcl
> variable "ami_name" {
>     default = "SOME_NAME"
>     type = string
>     validation {
>         condition = substr(var.ami_name, 0, 4) == "ami-"
>         error_message = "This variable must start with 'ami-'"
>           }
> }
> ```
> * For checking if a number is not exceeding a range
> ```tcl
> variable "VARIABLE_NAME" {
>     default = VALUE
>     type = number
>     validation {
>         condition = var.VARIABLE_NAME >= var.SMALL_NUMBER && var.VARIABLE_NAME < var.LARGE_NUMBER
>         error_message = "This number must be in a range \[ ${var.lesser_number} , ${var.greater_number} )"
>     }
>           }
> ```

## TERRAFORM STATE
* State file is a json data structure that contains the that maps real world infrastructure resources to resource definitions in the config file
* The state file can be visualized by `cat terraform.tfstate`
* Each resource in terraform has a id to identify the resources in the real world and it also helps in resolving resource dependencies
* Terraform also helps in tracking metadata (helps in deleting resources)

### REMOTE STATE
* Statelocking to prevent the corruption of state file

> [!IMPORTANT]
> #### REMOTE STATE and STATE LOCKING (REMOTE BACKEND)
> * **State Locking**: Terraform feature to avoid corruption of state file when two terraform workflows are running simultaneously
> * **Remote State**: Project are worked upon as a team, everybody needs access to the state file while working on the infrastructure
> * Remote state files provide :
>     - Team collaboration
>     - Security
>     - Reliability
>
> * Types of backend:
>     - s3
>     - AzureRM
>     - GCS
>     - Consul
>     - Artifactory
>     - HCP

> [!TIP]
> #### REMOTE BACKEND with S3 and DYNAMODB
> * Along with the `main.tf` also create one more file `terraform.tf` that contains remote backend configurations
> * Configure `terraform.tf` as below
> ```tcl
> terraform {
>     backend "s3" {
>         bucket = "BUCKET_NAME"
>         key = "PATH_OF_TERRAFORM_FILE_ON_S3"
>         region = "BUCKET_REGION"
>         dynamodb_table = "TABLE_NAME"
>     }
> }
> ```
> * Before using the remote backend, run `terraform init` to initialize it

> [!IMPORTANT]
> #### TAINT
> * Sometimes terraform apply fails due to wrong path or command in provisioner sub block or attribute
> * Tainted resources are entirely recreated
> * Terraform marks those resources as tainted and then they are re created when next apply command is run
> * There are situations when we want to recreate a reource forcefuly, we taint it using
> ```bash
> terraform taint RESOURCE_TYPE.RESOURCE_NAME
> ```
> * To untaint the resource
> ```bash
> terraform untaint RESOURCE_TYPE.RESOURCE_NAME
> ```

> [!TIP]
> #### PERFORMANCE
> * Performance goes down as number of resources increase because fetching from cloud provider through network (slow process)
> * When we need a resource replacement we need it to be fast so we want terraform to refer to state file but not refresh it `terraform plan --refresh=False`

* It also helps in collaboration in a team
* It contains sensitive info like ssh key pairs

> [!CAUTION]
> * Config file should be stored in a version control system but the state file should be stored in a backend system like s3 or a remote server

> [!WARNING]
> * Never change terraform state file manually, instead use terraform commands

## WORKING WITH TERRAFORM
| COMMAND | EFFECT |
| ------- | ------ |
| `terraform validate` | Check syntax errors |
| `terraform fmt` | Format the file in required standard format |
| `terraform show` | See current infrastructure as seen by terraform |
| `terraform show -json` | See current infrastructure in json format |
| `terraform providers` | See all installed providers |
| `terraform output` | See all output resources |
| `terraform output RESOURCE_TYPE.RESOURCE_NAME ...` | Print specific variables |
| `terraform providers mirror /path/to/mirror/the/providers` | Mirror providers |
| `terraform refresh` | Refresh/sync terraform to real world infrastructure if any manual change made |
| `terraform graph` | Get dependency graph, we can put the o/p in graph visualiser |

## STATE COMMANDS
| COMMAND | EFFECT |
| ------- | ------ |
| `terraform state list` | List all the resources |
| `terraform state show RESOURCE` | Print info of resource, `RESOURCE` taken by `list` command |
| `terraform state mv RESOURCE_TYPE.RESOURCE_NAME RESOURCE_TYPE.NEW_RESOURCE_NAME` | Renames the resource |
| `terraform state pull` | Pull remote state file from backend |
| `terraform state rm RESOURCE_TYPE.RESOURCE_NAME` | Removing the resources from state file (does not delete resource), once removed from state file, one must remove the associated resource blocks from config file too |
| `terraform state push PATH_OF_TERRAFORM_FILE` | Override remote state file with local state file, use with caution (very dangerous) |


> [!IMPORTANT]
> ### MUTABLE VS IMMUTABLE INFRASTRUCTURE
> * Terraform uses immutable way of managing resources

## RESOURCE TARGETING
* There are times when there are changes in config file but only few are needed to implemented at the momment, to solve that issue, we have this feature
* This feature should be rarely used in case of urgency
```bash
terraform apply -target RESOURCE_TYPE.RESOURCE_NAME
```

## LIFECYCLE
* Normally resourecs are destroyed before creating but this can be altered by using `create_before_destroy` parameter for resource in the lifecycle parameter
* If we do not want any resource to never get destroyed it can be prevented using `prevent_destroy`, usable for databases, note that `terraform destroy` command will destroy the resource
* When there is no need to consider the changes in the tags attribute we must `ignore_changes` and similarly in any other attributes like ami or subnet or file content and many more or use all to ignore all attributes
```tcl
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
    tags = {
        TAG1 = "VALUE1"
        TAG2 = "VALUE2"
    }
    # ATTRIBUTES CAN BE `tags` `instance_type` etc
    # OR WE CAN SET ignore_changes = all to avoid change due to any parameter
    lifecycle {
        LIFECYCLE_RULE = VALUE
        ignore_changes = [
            PARAMETER1
            PARAMETER2
            ...
        ]
    }
}
```

## DATASOURCES
* To use data from outside world in terraform
* See terraform registry for more details on the attributes for the data block
```hcl
data "aws_key_pair" "KEY_PAIR_RESOURCE_NAME" {
    key_name = "KEY_NAME_IN_AWS"
}
```
* Data sources only read from and to the infrastructure and its resources, it does not modify

* Generally all data and config for resources are defined in the terraform file itself, but what if one already exist and we want to reuse it.
* A resource is used to create infrastructure but a data source is created to read infrastructure

> [!TIP]
> ### EXAMPLE
> * For example an aws keypair exist in AWS and it is needed to create a new resource
> ```tcl
> data "aws_key_pair" "KEY_NAME" {
>     key_name = "KEY_NAME_IN_AWS"
> }
>
> resource "aws_instance" "INSTANCE_NAME" {
>     ami = "AMI"
>     instance_type = "INSTANCE_TYPE"
>     key_name = data.aws_key_pair.KEY_NAME.key_name
> }
> ```


## META ARGUMENTS
* We have altready seen two meta arguments `depends_on` and `lifecycle`
* Now for the loop and iteration types `count` and `for each`

### COUNT
* Count issues the number of instances to be created for that resource block
* Creates multiple resources and now the resource block is not considered as a single resource but a list of resources that can be accessed using the `[]` method to assess the elements
```tcl
# main.tf
resource "RESOURCE_NAME" "RESOURCE_NAME" {
    filename = var.VARIABLE_NAME[count.index]
    count = length(var.VARIABLE_NAME)
}

variables.tf
variable "VARIABLE_NAME" {
    default = [
        "VAL1"
        "VAL2"
        "VAL3"
    ]
}
```

> [!CAUTION]
> * When there is a need to change the length of the variable or the number of files in the list, all the resources gets removed and changed, to solve this issue, we use the `for_each` argument

### FOR EACH
* It only works with `map` and `set`
* The resources are created and seen as a map and not a list
```tcl
# main.tf
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    PARAMETER = each.value
    for_each = var.VARIABLE_NAME
}

# variables.tf
variable "VARIABLE_NAME" {
    type = set(string)
    default = [
        "VAL1"
        "VAL2"
        "VAL3"
    ]
}
```

> [!NOTE]
> * When we remove a resource from the set or map, say R1, only R1 is destroyed, none is recreated


## PROVISIONERS
* Provides features to run tasks and commands on remote resources and locally on the machine

> [!TIP]
> * Try avoiding provisioners and use native solutions provided by cloud platforms like `aws user data`

### REMOTE EXEC
* Run tasks on remote resource after the resource is deployed
* All these commands need connection to be established, in our case it is ssh, so we use the connection block
```tcl
resource "aws_instance" "RESOURCE_NAME" {
    ami = "AMI"
    instance_type = "INSTANCE_TYPE"
    provisioner "remote-exec" {
        inline = [
            "sudo apt update",
            "sudo apt install nginx -y",
            "sudo systemctl enable nginx",
            "sudo systemctl start nginx"
        ]
    }
    connection {
        type = "ssh"
        host = self.public_ip
        user = "ubuntu"
        private_key = file("PATH_TO_PRIV_KEY")
    }
    key_name = aws_key_pair.KEY_NAME.key_name
    vpc_security_group_ids = [ aws_security_group.ssh-access.id ]
}
```

### LOCAL EXEC
* Run tasks on local machine
* For example storing the public ip of instance
* If the command fails, the terraform resource provisioning also fails
```tcl
resource "aws_instance" "RESOURCE_NAME" {
    ...
    provisioner "local-exec" {
        command = "echo ${aws_instance.RESOURCE_NAME.public_ip} >> /tmp/ips.txt"
    }
    provisioner "local-exec" {
        when = destroy
        command = "echo ${aws_instance.RESOURCE_NAME.public_ip} destroyed >> /tmp/nips.txt"
    }
    provisioner "local-exec" { # CONTINUE EVEN IF PROVISIONER COMMAND FAILS
        on_failure = continue
        command = "echo 'asd'"
    }
}
```

## LOGGING and DEBUG
* To enable logging, run `export TF_LOG=LOG_LEVEL` before terraform commands
* Store all the logs in files `export TF_LOG_PATH=PATH`
* There are few `LOG_LEVEL` in terraform:
    - `INFO`
    - `WARNING`
    - `ERROR`
    - `DEBUG`
    - `TRACE`

## MODULE
* A configuration directory containing `.tf` files is a module in itself
> [!IMPORTANT]
> ### EXAMPLE
> ```tcl
> root_proj_dir
>     modules
>         payroll_app
>             app_server.tf
>             s3_bucket.tf
>             dynamodb_table.tf
>             variables.tf
>     us_server
>         main.tf 
>     uk_server
>         main.tf
>
> # root_proj_dir/modules/payroll_app/app_server.tf
> resource "aws_instance" "app_server" {
>     ami = var.ami
>     instance_type = "t2.medium"
>     tags = {
>         Name = "${var.app_region}-app-server"
>     }
>     depends_on = [
>                     aws_dynamodb_table.payroll_db
>                     aws_s3_bucket.payroll_data
>                 ]
>
> # root_proj_dir/modules/payroll_app/s3_bucket.tf
> resource "aws_s3_bucket" "payroll_data" {
>     bucket = "${var.app_region}-${var.bucket}"
> }
>
> # root_proj_dir/modules/payroll_app/dynamodb_table.tf
> resource "aws_dynamodb_table" "payroll_db" {
>     name = "user_data"
>     billing_mode = "PAY_PER_REQUEST"
>     hash_key = "EmployeeID"
>
>     attributes = {
>         name = "EmployeeID"
>         type = "N"
>     }
> }
>
> # root_proj_dir/modules/payroll_app/variables.tf
> variable "app_region" {
>     type = string
> }
> variable "bucket" {
>     default = "flexit-payroll-alpha-22001c"
> }
> variable "ami" {
>     type = string
> }
>
> # root_proj_dir/us_server/main.tf
> module "us_payroll" {
>     source = "../modules/payroll_app"
>     app_region = "us-east-1" # PASSING VARIABLES TO MODULE VARIABLES
>     ami = "ami-12314234243"
> }
>
> resource "aws_instance" "showpiece_ec2" {
>     ami = "ami-123131131231"
>     instance_type = "t2.micro"
>     depends_on = [
>                     module.us_payroll.aws_instance.app_server
>                 ]
> }
>
> # root_proj_dir/us_server/provider.tf
> provider "aws" {
>     region = "us-east-1"
> }
> ```

> [!TIP]
> ### MODULES FROM TERRAFORM REGISTRY
> * Search terraform registry for usage and availability
> * Install terraform module using `terraform get`, but make sure that it has been used by the configuration files
> * Using the `terraform-aws-modules/security-group/aws/modules/ssh`
> ```tcl
> # main.tf
> module "sec_grp_ssh" {
>     source = "terraform-aws-modules/security-group/aws/modules/ssh"
>     version = "3.16.0"
>     vpc_id = "vpc-1231243"
>     ingress_cidr_blocks = [
>                             "10.10.0.0/16"
>                         ]
>     name = "ssh-access"
> }
> ```

## TERRAFORM FUNCTIONS
* We have already seen some functions like `file('PATH')` and `toset(var.VARIABLE_NAME)` and `len(var.VARIABLE_NAME)`
### NUMERIC FUNCTIONS
* `max(1,2,3,4)` and `min(1,2,3,4)`
* To use sets or lists in min and max functions we must do it like this
```tcl
variable "huh" {
    type = set(number)
    default = [1,2,3,4]
}

# TERRAFORM CONSOLE
> max(var.huh...) # THESE THREE DODS ARE EXPANSION SYNTAX
```
* `ceil(10.12)` and `floor(1.33)`

### STRING FUNCTIONS
* `split(',', 'uhhu,uhu,df,vdh,df,df')`
* `upper('skjdhvljhd')`
* `lower('JHBHBvihvibDFF')`
* `substr('skjdbbvjbsjlv', START_OFFSET, TOTAL_CHARACTERS)`
* `join(',', ['sdfsd','hfghf','wrew','bvc','yi'])`

### COLLECTION FUNCTIONS
* `len(var.VARIABLE_NAME)`
* `index(var.VARIABLE_NAME, TO_FIND)`
* `element(var.VARIABLE_NAME, INDEX)`
* `contains(var.VARIABLE_NAME, TO_FIND)`

### MAP FUNCTIONS
* `keys(var.VARIABLE_NAME)`
* `values(var.VARIABLE_NAME)`
* `lookup(var.VARIABLE_NAME, KEY)`
* `lookup(var.VARIABLE_NAME, KEY, DEFAULT_VALUE)`

### CONDITIONAL STATEMENTS
* Almost same as python but some anomalies are there
* **Comparison operators**: `==`, `!=`, `<`, `>`, `<=`, `>=`
* **Logical operators**: `&&`, `||`, `!`

#### IF CONDITIONS
* `condition ? true_val : false_val`
```tcl
variable "length" {
    type = string
}
resource "random_password" "password" {
    length = var.length < 8 ? 8 : var.length
}
```

## LOCAL VALUES
* There are repetitive tasks like adding tage to resources, to eleminate such function, we use loacl values
```tcl
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    count = locals.resource_count
    ...
    tags = local.COMMON_TAGS
}

locals {
    COMMON_TAGS = {
        Name = "NAME"
        Department = "DEPARTMENT"
    }
    resource_count = VALUE
}
```

## DYNAMIC BLOCK and SPLAT EXPRESSIONS
* When there are too many ingress rules to be applied to a VPC, this way helps
```tcl
resource "aws_security_group" "RESOURCE_NAME" {
    name = "NAME"
    vpc_id = aws_vpc.VPC_RESOURCE_NAME.id
    dynamic "ingress" {
        for_each = var.INGRESS_VARIABLE
        content {
            from_port = INGRESS_VARIABLE.value
            to_port = INGRESS_VARIABLE.value
            protocol = "tcp"
            cidr_blocks = [ "0.0.0.0/0" ]
        }
    }
}

output "to_ports" { # THIS IS A SPECIAL EXPRESSION CALLED SPLAT THAT ITERATES THROUGH ALL THE ELEMENTS
    value = aws_security_group.RESOURCE_NAME.ingress[*].to_port
}
```

## WORKSPACES
* Logical sections in terraform project
* Allow to use configurration files within a directory to be reused multiple times for different purposes
```
root_proj_dir
    main.tf
    variables.tf
```
| COMMAND | EFFECT |
| ------- | ------ |
| `terraform workspce new WORKSPACE_NAME` | Create workspace |
| `terraform workspace list` | List workspaces |
| `terraform workspace select WORKSPACE_NAME` | Select workspaces |

> [!TIP]
> ### EXAMPLE
> * Manage resource in multiple environments
> * `main.tf`
> ```tcl
> resource "aws_instance" "RESOURCE_NAME" {
>     ami = "AMI"
>     instance_type = lookup(var.instance_type, terraform.workspace)
> }
> ```
> * `variable.tf`
> ```tcl
> variable "instance_type" {
>     type = map
>     default = {
>         "development" = "t2.micro"
>         "production" = "m5.large"
>     }
> }
> ```


> [!TIP]
> * Now here is a catch, we do not have any terraform.tfstate file in the main directory, now we have a new subdirectory structure where all the state files are stored named `terraform.tfstate.d`
>
> ```
> terraform.tfstate.d
>     WORKSPACE1
>         terraform.tfstate
>     WORKSPACE2
>         terraform.tfstate
> ```


## AWS
### IAM
* Setup IAM to grant access to one aws service to another
### IAM USER
```
provider "aws" {
    region = "us-west-2"
    access_key = "asdasd"
    secret_key = "qweqwe"
}

resource "aws_iam_user" "admin-user" {
    name = "Kushal"
    tags = {
        Description = "Team lead"
    }
}
```
### USING SECRET KEYS
* Storing a secret value is not safe in the main terraform files
#### USING CONFIG FILE
```
# /home/user/.aws/credentials
[default]
aws_access_key_id = "asdasd"
aws_secret_access_key = "qweqwe"

# main.tf
provider "aws" {
    region = "us-west-2"
}

resource "aws_iam_user" "admin-user" {
    name = "Kushal"
    tags = {
        Description = "Team lead"
    }
}
```
#### USING ENVIRONMENT VARIABLES
* We export variables and then they are picked up by terraform
```
export AWS_ACCESS_KEY_ID=asdasd
export AWS_SECRET_ACCESS_KEY_ID=qweqwe
# We can also place the AWS_REGION, completely replacing the aws provider block
export AWS_REGION=us-west-2
```
```
# main.tf
resource "aws_iam_user" "admin-user" {
    name = "Kushal"
    tags = {
        Description = "Team lead"
    }
}
```
### IAM POLICY
* We can use here doc in Terraform
```
[COMMAND] <<DELIMITER
...
...
...
DELIMITER
```
```
resource "aws_iam_user" "admin-user" {
    name = "Kushal"
    tags = {
        Description = "Team lead"
    }
}

resource "aws_iam_policy" "adminUser" {
    name = "All admin users"
    policy = <<EOF
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "*",
                "Resource": "*"
            }
        ]
    }
}

resource "aws_iam_user_policy_attachment" "lucy-admin-user" {
    user = aws_iam_user.admin-user.name
    policy_arn = aws_iam_policy.adminUser.arn
}
```
* We can also create a policy json file and then read it from the terraform function, note that the file should be in the same terraform directory
```
# IAM policy file admin-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}

resource "aws_iam_user" "admin-user" {
    name = "Kushal"
    tags = {
        Description = "Team lead"
    }
}

resource "aws_iam_policy" "adminUser" {
    name = "All admin users"
    policy = file("admin-policy.json")
}

resource "aws_iam_user_policy_attachment" "lucy-admin-user" {
    user = aws_iam_user.admin-user.name
    policy_arn = aws_iam_policy.adminUser.arn
}
```

### AWS S3
* To access the bucket, we can use the url `https://BUCKET_NAME.REGION.amazonaws.com`
* Fi want to access a file located in `/pictures/huhu.jpeg`, we can do `https://BUCKET_NAME.REGION.amazonaws.com/pictures/huhu.jpeg`
* We use bucket policies to set bucket access policies
```
$ main.tf
resource "aws_s3_bucket" "huhu" { # TO CREATE A S3 BUCKET
    bucket = "huhu"
    tags = {
        Description = "Huhu"
    }
}

resource "aws_s3_bucket_object" "hu" { # TO UPLOAD FILES TO BUCKET
    content = "/root/huhu/hu.txt"
    key = "hu.txt"
    bucket = aws_s3_bucket.huhu.id
}

resource "aws_iam_group" "huh" { # CREATE A USER GROUP POLICY
    group_name = "huh"
    policy = <<EOF
    {
        "Version": 2012-10-17,
        "Statement": [
            {
                "Action": "*",
                "Effect": "Allow",
                "Resource": "arn:aws:s3:::${aws_s3_bucket.huhu.id}",
                "Principal": {
                    "AWS": [
                        "${data.aws_iam_group.hu.arn}"
                    ]
                }
            }
        ]
    }
}
```

### DYNAMO DB
* NoSQL database from aws
```
resource "aws_dynamodb_table" "hu" {
    name = "huhu"
    hash_key = "qwe"
    billing_mode = "PEY_PER_REQUEST"
    attribute = {
        name = "qwe"
        type = "S"
    }
}
```

### EC2
```tcl
# main.tf
resource "aws_instance" "my_ec2" {
    ami = "AMI"
    instance_type = "t2.micro"

    tags = {
        Name = "huhu"
    }

    user_data = <<-EOF
                #!/bin/bash
                sudo apt update
                sudo apt install nginx -y
                sudo apt install ansible
                EOF

    }
    connection { # PROVIDE CONNECTION PARAMETERS FOR PROVISIONER REMOTE-EXEC
        type = "ssh"
        host = "self.public_ip"
        user = "ubuntu"
        private_key = file("path/to/private/key/on/local/machine")
    }

    provisioner "local-exec" { # EXECUTE COMMAND ON LOCAL MACHINE THAT RUNS TERRAFORM BINARY
        # HERE WE WILL STORE THE PUBLIC IP IN A TEXT FILE
        command = "echo ${aws_instance.myh_ec2.public_ip} >> /path/to/store.txt"
    }

    provisioner "local-exec" {
        # HERE WE WILL EXECUTE THIS PROVISIONER ON A CONDITION
        when = destroy
        on_failure = continue
        command = "echo ${aws_instance.myh_ec2.public_ip} destroyed > /path/to/store.txt"
    }

    key_name = aws_key_pair.web.id
    vpc_security_group_ids = [aws_security_group.ssh-access.id]

}

resource "aws_key_pair" "web" {
    public_key = file("path/to/pub/key/on/local/machine.pub")
}

resource "aws_security_group" "ssh-access" {
    name = "ssh-access"
    description = "huhu"
    ingress = {
        from_port = 22
        to_port = 22
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
}

output publicip {
    value = aws_instance.webserver.public_ip
}

# provider.tf
provider "aws" {
    region = "us-west-1"
}
```


#### CREATING KEY PAIRS and USING IN EC2
```tcl
resource "aws_key_pair" "KEY_NAME" {
    key_name = "key_production"
    public_key = "..."
}

resource "aws_instance" "my_ec2" {
    ami = "AMI"
    instance_type = "INSTANCE_TYPE"
    key_name = aws_key_pair.KEY_NAME.key_name
}
```
* For example an aws keypair exist in AWS and it is needed to create a new resource
```tcl
data "aws_key_pair" "KEY_NAME" {
    key_name = "KEY_NAME_IN_AWS"
}

resource "aws_instance" "INSTANCE_NAME" {
    ami = "AMI"
    instance_type = "INSTANCE_TYPE"
    key_name = data.aws_key_pair.KEY_NAME.key_name
}
```
