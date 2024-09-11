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
> * Provider full names follow the convention `HOSTNAME/ORGANIZATIONAL_NAMESPACE/PLUGIN_NAME: version = "~> VERSION"`, where the `HOST_NAME` is the hostname of the plugin 

> [!NOTE]
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

### CONFIGURATION DIRECTORY
* Resources can be defined by multiple files but all resources should be in the `main.tf` file
* There are other files too for their own purposes, named: `variables.tf`, `outputs.tf`, `providers.tf` etc

### VARIABLES FILE
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

* We can also use variable like below
    - In this case we can export variables in shell `export TF_VAR_VARNAME="VALUE"` and then run the apply or plan command
    - Or we can give variables in run time `terraform apply -var "var1=val1" -var "var2=val2"`
```tcl
variable "VARIABLE_NAME" {

}
```

#### .tfvars and .tfvars.json FILE
* We can define variables in a `.tfvars` file in following format
```tcl
var1 = "val1"
var2 = "2"
seperator = "."
```
* Then use command like `terraform apply -var-file variables.tfvars`
* To make this file automatically loadable by terraform save it as `*.auto.tfvars` or `*.auto.tfvars.json`

* **Variable Precedence:** `main.tf > -var or -var-file > *.auto.tfvars (alphabetical order) > terraform.tfvars > environment variables`

> [!NOTE]
> We can use `terraform show` to get variable states and properties

### RESOURCE DEPENDENCIES
* If output of one variable `A` is required by another variable `B` then `B` is dependent on `A`
* While creating a resource, terraform creates `A` forst then `B`, but deletes in reverse order. This is called inplicit dependency
* But we can also specify external dependency
```tcl
resource "local_file" "pet" {
    filename = "/qwe/asd"
    content = "huhu"
    depends_on = [random_pet.my-pet]
}

resource "random_pet" "my-pet" {
    prefix = "Mr"
    seperator = "."
    length = var.length
}
```
```
# CREATE A TLS_PRIVATE_KEY AND STORE IN `terraform state` AND THEN SAVE IN A FILE
resource "tls_private_key" "pvtkey" {
    algorithm = "RSA"
    rsa_bits  = 4096
}

resource "local_file" "key_details" {
    content  = tls_private_key.pvtkey.private_key_pem
    filename = "/root/key.txt"
}
```

### OUTPUT VARIABLES
* We can print variables on screen
```
resource "random_pet" "my-pet" {
    prefix = "Mr"
    seperator = "."
    length = "4"
}

output pet-name {
    value = random_pet.my-pet.id
    description = "qwe"
}
```
* We can also use commands
    - `terraform output`
    - `terraform output VARIABLE_NAME`

## TERRAFORM STATE
* State file is a json data structure that contains the that maps real world infrastructure resources to resource definitions in the config file
* We can see the state file by `cat terraform.tfstate`
* Each resource in terraform has a id to identify the resources in the real world and it also helps in resolving resource dependencies
* Terraform also helps in tracking metadata (helps in deleting resources)
* Performance because of getting state from local system instead of fetching from cloud provider through network (elemination of slow process)
* When we need a resource replacement we need it to be fast so we want terraform to refer to state file but not refresh it `terraform plan --refresh=False`
* It also helps in collaboration in a team
* It is non optional
* It contains sensitive info like ssh key pairs
* Config file should be stored in a version control system but the state file should be stored in a backend system like s3 or a remote server
* Never change terraform state file manually, instead use terraform commands

## WORKING WITH TERRAFORM
* `terraform validate` to check syntax errors
* `terraform fmt` to format the file in required standard format
* `terraform show` to see current infrastructure as seen by terraform
* `terraform providers` to see all installed providers
* `terraform providers mirror /path/to/mirror/the/providers`
* `terraform refresh` to refresh terraform to real world infrastructure if any manual change made
* `terraform graph` to get dependency graph, we can put the o/p in graph visualiser

### MUTABLE VS IMMUTABLE INFRASTRUCTURE
* Terraform uses immutable way of managing resources

### LIFECYCLE
* Normally resourecs are destroyed before creating but this can be altered by using `create_before_destroy` parameter for resource in the lifecycle parameter
* If we do not want any resource to never get destroyed we can use `prevent_destroy`, usable for databases, note that `terraform destroy` command will destroy the resource
* When there is no need to consider the changes in the tags attribute we must `ignore_changes` and similarly in any other attributes like ami or subnet or file content and many more or use all to ignore all attributes
```
resource "local_file" "key_details" {
    content  = tls_private_key.pvtkey.private_key_pem
    filename = "/root/key.txt"

    tags = {
        name = "huhu"
    }

    lifecycle {
        create_before_destroy = true
        ignore_changes = [
            tags
        ]

    }
}
```
### DATASOURCES
* To use data from outside world in terraform
* See terraform registry for more details on the attributes for the data block
```
resource "local_file" "dog" {
    filename = "/root/pet.txt"
    content = data.local_file.dog.content
}
data "local_file" "dog" {
    filename = "/root/dog.txt"
}
```
* Data sources only read from and to the infrastructure and its resources, it does not modify

### META ARGUMENTS
* We have altready seen two meta arguments `depends_on` and `lifecycle`
* Now for the loop and iteration types `count` and `for each`
#### COUNT
* Creates multiple resources and now the resource block is not considered as a single resource but a list of resources that can be accessed using the `[]` method to assess the elements
```
# main.tf
resource "local_file" "qwe" {
    filename = var.filename[count.index]
    count = 3
}

variables.tf
variable "filename" {
    default = "/root/qwe.txt"
}
```
* But since we have only defined the parameter `count` it creates the same file 3 tines, but not 3 different files so we change the filename varibale block as
```
# main.tf
resource "local_file" "qwe" {
    filename = var.filename[count.index]
    count = 3
}

variables.tf
variable "filename" {
    default = [
        "/root/qwe.txt"
        "/root/qwe2.txt"
        "/root/qwe3.txt"
    ]
}
```
* Still we have a problem, if we change the number of elements in the varible block `filename` it gives error, to solve we do this
```
# main.tf
resource "local_file" "qwe" {
    filename = var.filename[count.index]
    count = length(var.filename)
}

variables.tf
variable "filename" {
    default = [
        "/root/qwe.txt"
        "/root/qwe2.txt"
        "/root/qwe3.txt"
    ]
}
```
* NOTE: When we change the length of the variable or the number of files in the list, all the resources gets removed and changed, to solve this issue, we use the for_each parameter/argument

#### FOR EACH
* It only works with map and set
* The resources are created and seen as a map and not a list
```
# main.tf
resource "local_file" "pet" {
    filename = each.value
    for_each = var.filename
}

# variables.tf
variable "filename" {
    type = set(string)
    default = [
        "/root/qwe.txt"
        "/root/qwe2.txt"
        "/root/qwe3.txt"
    ]
}
```

#### VERSION CONSTRAINTS
* Sometimes we need a specific version of a provider
* See more for specific versions of providers in the use provider section
* We can use `"> 1.2.0, 2.0.0, !=1.4.0, ~> 1.3.0"` to get a specific range of the version of the provider
```
# main.tf
terraform {
    required_providers {
        local = {
            source = "hashocorp/local"
            version = "1.4.0"
        }
    }
}
```

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


## REMOTE STATE
* Statelocking to prevent the corruption of state file
### REMOTE BACKEND WITH S3 AND DYNAMODB
* S3 for storing the state file and dynamodb for locking the state
```
# terraform.tf
terraform {
    backend "s3" { # TO USE S3 AS BACKEND
        bucket = "BUCKET_NAME"
        key = "PATH_TO_STATE_FILE_IN_S3_BUCKET"
        region = "us-west-1"
        dynamodb = "state-locking" # THE TABLE MUST HAVE A PRIMARY KEY AS LockID
    }
}

# main.tf
CREATE RESOURCES AS USUAL
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
}
```
### STATE COMMANDS
* `terraform state list` list all the resources
* `terraform state show RESOURCE` print info of resource, `RESOURCE` taken by `list` command
* `terraform state mv RESOURCE_TYPE.RESOURCE_NAME RESOURCE_TYPE.NEW_RESOURCE_NAME` renames the resource
* `terraform state pull` to see the remote state file
* `terraform state rm RESOURCE_TYPE.RESOURCE_NAME` removes a resource, but remove it from the config file manually too

## TAINT
* Sometimes terraform apply fails due to wrong path or command in provisioner sub block or attribute
* Tainted resources are entirely recreated
* Terraform marks those resources as tainted and then they are re created when next apply command is run
* When we have to make change to a resource we may recreate a resource by running terraform apply command but the better way is to taint the resource to upgrade or make required changes using `terraform taint RESOURCE_TYPE.RESOURCE_NAME`
* To untain the resource run `terraform untaint RESOURCE_TYPE.RESOURCE_NAME`

## DEBUG
* Terraform have a lot of debug levels: `INFO`, `WARNING`, `ERROR`, `DEBUG` and `TRACE`
* To use debugging use: `export TF_LOG=DEBUG_LEVEL`

## IMPORT
* Import externally managed or created resources
* terraform import RESOURCE_TYPE.RESOURCE_NAME RESOURCE_ID_ON_CLOUD_CONSOLE
    - example: `terraform import aws_instance.web i-12321434242344`
* This will give an error and also it will give a code, paste that in the `main.tf`
```
resource "aws_instance" "web" {
    # (resource arguments)
}
```
* Then again run the same command above, this time , no error reported and also theresource is imported
* Now we can examine `terraform.tfstate` and then change the resource block in the `main.tf`
* Now the resource block is in control of terraform

## MODULE
* A configuration directory containing .tf files is a module in itself
```
root_proj_dir
    dir1
        main.tf
        variables.tf
    dir2
        main.tf
        variables.tf
```
* If we use the following lines in following files
```
# root_proj_dir/dir2/main.tf
module "huhu" {
    source = "../dir1" # path/to/another/config/directory

}

# root_proj_dir/dir1/main.tf
resource "aws_instance" "webserver" {
    ami = var.ami
    instance_type = "t2.micro"
    key_name = var.key
}

# root_proj_dir/dir1/variables.tf
variable ami {
    type = string
    default = "ami-3453452342"
}
```
* `dir2` becomes the root module and `dir1` becomes the child module or imported module

* Another example:
```tcl
root_proj_dir
    modules
        payroll_app
            app_server.tf
            s3_bucket.tf
            dynamodb_table.tf
            variables.tf
    us_server
        main.tf 
    uk_server
        main.tf

# root_proj_dir/modules/payroll_app/app_server.tf
resource "aws_instance" "app_server" {
    ami = var.ami
    instance_type = "t2.medium"
    tags = {
        Name = "${var.app_region}-app-server"
    }
    depends_on = [
                    aws_dynamodb_table.payroll_db
                    aws_s3_bucket.payroll_data
                ]

# root_proj_dir/modules/payroll_app/s3_bucket.tf
resource "aws_s3_bucket" "payroll_data" {
    bucket = "${var.app_region}-${var.bucket}"
}

# root_proj_dir/modules/payroll_app/dynamodb_table.tf
resource "aws_dynamodb_table" "payroll_db" {
    name = "user_data"
    billing_mode = "PAY_PER_REQUEST"
    hash_key = "EmployeeID"

    attributes = {
        name = "EmployeeID"
        type = "N"
    }
}

# root_proj_dir/modules/payroll_app/variables.tf
variable "app_region" {
    type = string
}
variable "bucket" {
    default = "flexit-payroll-alpha-22001c"
}
variable "ami" {
    type = string
}

# root_proj_dir/us_server/main.tf
module "us_payroll" {
    source = "../modules/payroll_app"
    app_region = "us-east-1" # PASSING VARIABLES TO MODULE VARIABLES
    ami = "ami-12314234243"
}

resource "aws_instance" "showpiece_ec2" {
    ami = "ami-123131131231"
    instance_type = "t2.micro"
    depends_on = [
                    module.us_payroll.aws_instance.app_server
                ]
}

# root_proj_dir/us_server/provider.tf
provider "aws" {
    region = "us-east-1"
}
```

### MODULES FROM TERRAFORM REGISTRY
* Search terraform registry for usage and availability
* Install terraform module using `terraform get`, but make sure that it has been used by the configuration files
* Using the `terraform-aws-modules/security-group/aws/modules/ssh`
```tcl
# main.tf
module "sec_grp_ssh" {
    source = "terraform-aws-modules/security-group/aws/modules/ssh"
    version = "3.16.0"
    vpc_id = "vpc-1231243"
    ingress_cidr_blocks = [
                            "10.10.0.0/16"
                        ]
    name = "ssh-access"
}
```

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
* `&&` and, `||` or, `!` not
* `8 == '8'` gives true
#### IF CONDITIONS
* `condition ? true_val : false_val`
```
# main.tf
resource "random_password" "pass_gen" {
    length = var.length < 8 ? 8 : var.length
}
output password {
    value = random_password.pass_gen.result
}

# variables.tf
variable length {
    type = number
    desription = "huh"
}

$ > terraform apply --var=length=5
```

## WORKSPACES
* Allow to use configurration files within a directory to be reused multiple times for different purposes
```
root_proj_dir
    main.tf
    variables.tf
```
* To init or create a new workspace we use `terraform workspce new WORKSPACE_NAME`
* To list existing workspaces `terraform workspace list`
* To switch to a workspae `terraform workspace select WORKSPACE_NAME`
```tcl
# variables.tf
variable region {
    default = "us-east-1"
}
variable instance_type {
    default = "t2.micro"
}
variable ami {
    type = map
    default = {
        "p1" = "ami-12324"
        "p2" = "ami-45656"
    }
}

# main.tf
resource "aws_instance" "huhu" {
    ami = lookup(var.ami, terraform.workspace)
    instance_type = var.instance_type
    tags = {
        name = terraform.workspace
    }
}

$ > terraform apply
```
* Now here is a catch, we do not hacce any terraform.tfstate file in the main directory, now we have a new subdirectory structure where all the state files are stored named `terraform.tfstate.d`
```
terraform.tfstate.d
    WORKSPACE1
        terraform.tfstate
    WORKSPACE2
        terraform.tfstate
```

* Terraform considers any file with `.tf` config as terraform in the project directory

## TERRAFORM PROVIDERS
* Official, partnered, community providers
* Provider full names follow the convention `HOSTNAME/ORGANIZATIONAL_NAMESPACE/PLUGIN_NAME: version = "~> VERSION"`, where the `HOST_NAME` is the hostname of the plugin

* Use specific version of the plugins
```tcl
terraform {
    required_providers {
        PROVIDER_NAME = {
            source = "SOURCE"
            version = "VERSION"
        }
    }
}
```
* `~> x.x.x` means any version less than `x.x+1.0` and greater than or equal to `x.x.x`

* Some times in the script we need to work with multi ple regions in aws
* We can create multiple aws providers of aws type with different regions and call them according to usage
```tcl
provider "aws" {
    region = "us-east-1"
    alias = "aws_us"
}

provider "aws" {
    region = "ap-south-1"
    alias = "aws_mumbai"
}

# CREATE EC2 IN US
resource "aws_instance" "testing_nfs" {
    ami = "AMI"
    instance_type = "INSTANCE_TYPE"
    provider = aws.aws_us #PROVIDER FORMAT IS "PROVIDER_NAME.ALIAS_NAME"
}

# CREATE EC2 IN MUMBAI
resource "aws_instance" "testing_nfs" {
    ami = "AMI"
    instance_type = "INSTANCE_TYPE"
    provider = aws.aws_mumbai #PROVIDER FORMAT IS "PROVIDER_NAME.ALIAS_NAME"
}
```

## VARIABLES
* If we do not provide any default value to a variable or override it programatically, it prompts while creating resource
* Variables format
```tcl
variable "VARIABLE_NAME" {
    default = VALUE
    description = "DESCRIPTION"
    type = TYPE
    sensitive = SENSITIVITY
    validation {
        condition = CONDITION
        error_message = "ERROR_MESSAGE"
    }
}
```
* `default` parameter contains the default/value of the variable
* `description` is just a description
* `type` is the type of data we are storing
* `sensitive` to define the sensitivity of the data, this decides wheather this variable is visible or not in the terraform state file
    - If `true`: not visible in the state file
    - If `false`: visible in the state file
* `validation is to make sure that the variable is following a convention or have a right value and many more`
* Even the output resource is not able to print the sensitive info/variable, but at the same time, we have a sensitive clause for output resource too which is just to avoid any mishaps
```tcl
output "OUTPUT_NAME" {
    value = var.SECRET_VARIABLE_NAME
    sensitive = true
}
```
* We can securely access sensitive variables by using
```bash
terraform output SECRET_VARIABLE_NAME
```

### Examples:
* For checking that a variable that we are defining is starting with `ami-` or not
```tcl
variable "ami_name" {
    default = "SOME_NAME"
    type = string
    validation {
        condition = substr(var.ami_name, 0, 4) == "ami-"
        error_message = "This variable must start with 'ami-'"
          }
}
```
* For checking if a number is not exceeding a range
```tcl
variable "just_a_number" {
    default = 123
    type = number
    validation {
        condition = var.just_a_number >= var.lesser_number && var.just_a_number < var.greater_number
        error_message = "This number must be in a range \[ ${var.lesser_number} , ${var.greater_number} )"
    }
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


#### RESOURCE TARGETING
* There are times when there are changes in config file but only few are needed to implemented at the momment, to solve that issue, we have this feature
* This feature should be rarely used in case of urgency
```bash
terraform apply -target RESOURCE_TYPE.RESOURCE_NAME
```

#### DATA SOURCES
* Generally all data and config for resources are defined in the terraform file itself, but what if one already exist and we want to reuse it.
* A resource is used to create infrastructure but a data source is created to read infrastructure
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

## STATE
* Terraform refreshes its state before taking any actions
* Terraform can be forced not to refresh the state using `terraform apply -refresh=false`
    - This could come in handy when working in large infrastructure where 1000s of resources are being managed

* **State Locking**: Terraform feature to avoid corruption of state file when two terraform workflows are running simultaneously
* **Remote State**: Project are worked upon as a team, everybody needs access to the state file while working on the infrastructure
* Remote state files provide : `team collaboration`, `security`, `reliability`

#### REMOTE STATE and STATE LOCKING (REMOTE BACKEND)
* Types of backend:
    - s3
    - AzureRM
    - GCS
    - Consul
    - Artifactory
    - HCP
* Along with the `main.tf` also create one more file `terraform.tf` that contains remote backend configurations
* Configure `terraform.tf` as below
```tcl
terraform {
    backend "s3" {
        bucket = "BUCKET_NAME"
        key = "PATH_OF_TERRAFORM_FILE_ON_S3"
        region = "BUCKET_REGION"
        dynamodb_table = "TABLE_NAME"
    }
}
```
* Before using the remote backend, run `terraform init` to initialize it

#### DEPENDENCY LOCK FILE (`terraform.lock.hcl`)
* It helps managing externally provided dependencies
* Ensures all providers are being used in the same version accross all the environments and operations 
* It provides:
    - Consistency
    - Reproducibility
    - Compatibility
* It contains:
    - Exact provider versions
    - Provider checksums
    - Information
* To update the state lock file when upgrading to new versions of providers, run `terraform init -upgrade`
* Always commit the state lock file

## COMMANDS
| COMMAND | EFFECT |
| ------- | ------ |
| `terraform validate` | Validate the syntactical and logical errors in the file |
| `terraform fmt` | Reformats the scripts |
| `terraform show` | Show current infrastructure |
| `terraform show -json` | See current infrastructure in json format |
| `terraform providers` | See all the providers being used |
| `terraform output` | See all output resources |
| `terraform output RESOURCE_TYPE.RESOURCE_NAME ...` | Print specific variables |
| `terraform refresh` | Sync terraform with real world infrastructure |
| `terraform graph` | Generate a visual representation of dependencies in terraform execution plan, this output can be used with `graphwiz` |
| `terraform state COMMAND RESOURCE_TYPE.RESOURCE_NAME` | Edit state file, where `COMMAND` can be `list`, `mv`, `pull`, `rm`, `show`, `push` |

> [!IMPORTANT]
> ### STATE SUBCOMMANDS
> * `pull`: Pulling the state file from remote backend
> * `mv`: Renaming the resources `terraform state mv RESOURCE_TYPE.RESOURCE_NAME_OLD RESOURCE_TYPE.RESOURCE_NAME_NEW`
> * `rm`: Removing the resources from state file (does not delete resource), once removed from state file, one must remove the associated resource blocks from config file too
> * `push`: Override remote state file with local state file, use with caution (very dangerous) `terraform state push PATH_OF_TERRAFORM_FILE`

## LIFECYCLE RULES
* create_before_destroy
* prevent_destroy - prevents resource from being destroyed such as data bases
* ignore_changes - prevents resource from updating
```tcl
resoruce "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
    lifecycle {
        # ATTRIBUTES CAN BE `tags` `instance_type` etc
        # OR WE CAN SET ignore_changes = all to avoid change due to any parameter
        ignore_changes = [
          ATTRIBUTE_1
          ATTRIBUTE_2
          ...
          ]
    }
}
```

```tcl
resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
    lifecycle {
        LIFECYCLE_RULE = VALUE
    }
}
```

### TAINTING
* When terraform is unable to create the resource, terraform taints it
* Tainting means marking to recreate
* There are situations when we want to recreate a reource forcefuly, we taint it using
```bash
terraform taint RESOURCE_TYPE.RESOURCE_NAME
```
* To untaint the resource
```bash
terraform untaint RESOURCE_TYPE.RESOURCE_NAME
```

## LOGGING
* To enable logging, run `export TF_LOG=LOG_LEVEL` before terraform commands
* Store all the logs in files `export TF_LOG_PATH=PATH`
* There are few `LOG_LEVEL` in terraform:
    - `INFO`
    - `WARNING`
    - `ERROR`
    - `DEBUG`
    - `TRACE`

## TERRAFORM WORKSPACES
* Logical sections in terraform project

| COMMAND | EFFECT |
| ------- | ------ |
| `terraform workspace list` | List workspaces |
| `terraform workspace new WORKSPACE_NAME` | Create a new workspace |
| `terraform workspace select WORKSPACE_NAME` | Select a workspace |

* State files for different workspaces are found in `terraform.tfstate.d/`

> [!TIP]
> * Manage resources of different environments of infrastructure
> * EXAMPLE
>     - `main.tf`
> ```tcl
> resource "aws_instance" "RESOURCE_NAME" {
>     ami = "AMI"
>     instance_type = lookup(var.instance_type, terraform.workspace)
> }
> ```
>     - `variable.tf`
> ```tcl
> variable "instance_type" {
>     type = map
>     default = {
>         "development" = "t2.micro"
>         "production" = "m5.large"
>     }
> }
> ```

## META ARGUMENTS
### COUNT
* Count issues the number of instances to be created for that resource block
* Instances are created as a list
```tcl
variable "VARIABLE_NAME" {
    type = list
    default = ["R1", "R2", "R3"]
}

resource "RESOURCE_TYPE" "RESOURCE_NAME" {
    ...
    count = length(var.VARIABLE_NAME)
    tags = {
        Name = var.VARIABLE_NAME[count.index]
    }
}
```

> [!CAUTION]
> * If "R1" removed from the resource list, terraform will show, 1 resource to be deleted and 2 to be modified
> * This is because when R1 removed, R2 and R3 shift right in index, so they are recreated

### FOR_EACH
* An iterator that works with `map` and `set`
* Instances are created as a map instead of a list
```tcl
variable "VARIABLE_NAME" {
    type = set
    default = ["R1", "R2", "R3"]
}

resource "INSTANCE_TYPE" "INSTANCE_NAME" {
    ...
    for_each = var.VARIABLE_NAME
    tags = {
        Name = each.value
    }
}
```

> [!NOTE]
> * When we remove a resource from the set or map, say R1, only R1 is destroyed, none is recreated

### PROVISIONERS
* Provides features to run tasks and commands on remote resources and locally on the machine

> [!TIP]
> * Try avoiding provisioners and use native solutions provided by cloud platforms like `aws user data`

#### REMOTE EXEC
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

#### LOCAL EXEC
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

## BUILTIN FUNCTIONS

### NUMERIC FUNCTIONS
* **max()**
    - The `...` at the end of variable name is for expansion
```tcl
variable "VARIABLE_NAME" {
    type = list(number)
    default = [ 125, 6, 7 ]
}

variable "NAME" {
    type = number
    default = max(var.VARIABLE_NAME...)
}
```
* Similarly we use `ceil()`, `floor()`

### STRING FUNCTIONS
* **split()**
```tcl
variable "VARIABLE_NAME" {
    type = string
    default = "abc,aws,qwe"
}

variable "NAME" {
    default = split("SPLIT_CHARACTER", var.VARIABLE_NAME)
}
```
* Similarly we use `upper()`, `lower()`, `title()` functions

### COLLECTION FUNCTIONS
* `length(var.VARIABLE_NAME)`
* **index()**
```tcl
variable "VARIABLE_NAME" {
    default = ["qwe", "asd", "zxc"]
    type = list(string)
}

variable "NAME" {
    default = index(var.VARIABLE_NAME, "qwe")
}
```
* **element()**
```tcl
variable "VARIABLE_NAME" {
    default = ["qwe", "asd", "zxc"]
    type = list(string)
}

variable "NAME" {
    default = element(var.VARIABLE_NAME, 2)
}
```
* **contains()**
```tcl
variable "VARIABLE_NAME" {
    default = ["qwe", "asd", "zxc"]
    type = list(string)
}

variable "NAME" {
    default = contains(var.VARIABLE_NAME, "qwe")
}
```

### MAP FUNCTIONS
* **keys()**
```tcl
variable "VARIABLE_NAME" {
    default = {
        "us-east-1" = "qwe",
        "ap-south-1" = "asd",
        "ca-central-1" = "zxc"
    }
    type = map
}

variable "NAME" {
    default = keys(var.VARIABLE_NAME)
}
```
* **values()**
```tcl
variable "VARIABLE_NAME" {
    default = {
        "us-east-1" = "qwe",
        "ap-south-1" = "asd",
        "ca-central-1" = "zxc"
    }
    type = map
}

variable "NAME" {
    default = values(var.VARIABLE_NAME)
}
```
* **lookup()**
```tcl
variable "VARIABLE_NAME" {
    default = {
        "us-east-1" = "qwe",
        "ap-south-1" = "asd",
        "ca-central-1" = "zxc"
    }
    type = map
}

variable "NAME" {
    default = lookup(var.VARIABLE_NAME, "ap-south-1", DEFAULT_VALUE)
}
```

## OPERATORS AND CONDITIONALS
* **Comparison operators**: `==`, `!=`, `<`, `>`, `<=`, `>=`
* **Logical operators**: `&&`, `||`, `!`
* **Condition**
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

## MODULES
* Every directory containing terraform files is a module
* Root module is the module which is called by the user
```tcl
# IMPORTING AND USING MODULES
module "NAME" {
    source = "MODULE_PATH"
    version = "VERSION"
    
    ARGUMENT1 = VALUE1
    ARGUMENT2 = VALUE2
    ...
}
```

* Example
