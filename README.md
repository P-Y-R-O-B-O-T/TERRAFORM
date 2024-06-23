# TERRAFORM
* Infrastructure as code is the process of managing and provisioning computer data center resources through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

#### PROBLEMS WITH TRADITIONAL IT INFRASTRUCTURE DEPLOYMENTS
    - Slow deployments
    - Expensive
    - Limited automation
    - Human error
    - Inconsistencies

#### TYPES OF IAC TOOLS
    - Configuration managenent: Ansible, Puppet, Salt Stack
        - Install and manage software
        - Maintains standard structure
        - Version control
        - Idempotent
    - Server templating: Docker, Vagrant, Packer
        - Creation of a custom image of a virtual machine or a container
        - Pre installed dependencies and softwares
        - Virtual machine or docker images
        - Immutable infrastructure
    - Provisioning tools: Terraform, Cloudformation
        - Deploy immutable infrastructure resources
        - Servers, databases, Networks, Components etc
        - Multiple providers

#### WHY TERRAFORM ?
    - Can handle multiple cloud platforms, using providers
    - HCL - declarative language (.tf)
    - Resource lifecycle managenent (provisioning, configuring, decommissioning)
    - Terraform state is the blueprint of what is deployed by terraform, it can read attributes of existing infrastructure components by configuring data sources
    - Terraform can import other resources inside it if any were created manually

#### HCL BASICS
```
<block> <resource_type> <resource_name> <parameters> {
    key1 = val1
    key2 = val2
}
```
* A block contains information about the infrastructure platform and resources within the platform that we want to create
```
EXAMPLE: CREATING A LOCAL FILE

resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "HUH"
}

resource "local_sensitive_file" "secret" { # store sensitive data in a file and do not print sensitive info when we run terraform plan
    filename = "/root/data.txt"
    content = "HUH"
}
```
```
EXAMPLE: CREATING A EC2 INSTANCE

resource "aws_instance" "webserver" {
    ami = "AMI_ID"
    instance_type = "t2.micro"
}
```
```
EAMPLE: CREATING AWS S3 BUCKET

resource "aws_s3_bucket" "data" {
    bucket = "BUCKET_NAME"
    acl = "private" # access control list for the bucket
}
```
* Terraform workflow has 4 steps:
    - Write configuration
    - Run `terraform init`: installs plugins
    - Review using terraform using `terraform plan`: see execution plan made by terraform
    - Apply changes using `terraform apply`: apply the execution plan

#### UPDATE RESOURCE
* Change the config and run `terraform plan` and then run `terraform apply`
* To destroy the infrastructure we can run `terraform destroy`

## TERRAFORM BASICS
* **PROVIDERS**
* AWS, GCP, Azure, etc
* Official providers, partners, community
* Providers are downloaded as hidden directory in the configuration directory
* Random provider is also important, it is a logical provider **See more in official docs**
* If we change config and use a new provider that is not installed, the `terraform apply` command won't work as the provider is not installed, we first need to install it using `terraform init`
```
resource "random_pet" "pet_name_obj" {
    length = 1
    prefix = "Mr"
    seperator = " "
}
```

### CONFIGURATION DIRECTORY
* Resources can be defined by multiple files but all resources should be in the `main.tf` file
* There are other files too named: `variables.tf`, `outputs.tf`, `providers.tf`

### VARIABLES FILE
* Store dynamic variables for configurations
* Variable block has 3 properties, available types are : `bool`, `string`, `number`, `list`, `map`, `object`, `tuple` and `any` 
```
# variables.tf
variable "NAME" {
    default = 2
    type = number
    description = "HUHU HUHU"
}

variable "zzz" {
    default = true
    type = bool
}

variable "zzzz" {
    default = "HUHU"
    type = string
}

variable "zzzzz" {
    default = 9
    type = number
}

variable "zzzzzzz" {
    default = ["hu", "huhu", "huhuhu", "huhuhuhu", "123"]
    type = list(string)
}

variable "zzzzzzzz" {
    default = {
        "key1" = "val1"
        "key2" = "val2"
    }
}

variable "zzzzzzzzz" {
    default = {
        "key1" = 23
        "key2" = 0
    }
    type = map(number)
}

variable "zzzzzzzzzz" {
    type = object({
        name = string
        color = string
        age = number
        food = list
        healthy = bool
    })
    default = {
        name = "ballu"
        color = string
        age = 123
        food = ["omlet", "mirch masala murg"]
        healthy = true
    }
}

variable "zzzzzzzzzzz" {
    default = [1, true, "qwe"]
    type = tuple([number, bool, string])
}

# main.tf
resource local_file "qwe" {
    filename = "/qwe/asd.txt"
    content = var.zzzzzzzz["key1"]
}
```
```
# variables.tf
variable "filename" {
    default = "/root/pets.txt"
}

variable "content" {
    defult = "HUHU"
}

variable "prefix" {
    default = "Mr"
}

variable "seperator" {
    default = "."
}

variable "length" {
    default = "2"
}

# main.tf
resource "local_file" "pet" {
    filename = var.filename
    caontent = var.content
}
resource "random_pet" "my-pet" {
    prefix = var.prefix
    seperator = var.seperator
    length = var.length
}
```
* We can also leav a variable like below
    - In this case we can export variables in shell `export TF_VAR_VARNAME="VALUE"` and then run the apply or plan command
    - Or we can give variables in run time `terraform apply -var "var1=val1" -var "var2=val2"`
```
variable "VARIABLE_NAME" {

}
```

#### .tfvars FILE
* We can define variables in a `.tfvars` file in following format
```
var1 = "val1"
var2 = "2"
seperator = "."
```
* Then use command like `terraform apply -var-file variables.tfvars`

* **Variable Precedence:** `main.tf > -var or -var-file > *.auto.tfvars > terraform.tfvars > environment variables`

#### USING OUTPUT OF ONE VARIABLE IN ANOTHER
```
resource "local_file" "zzz" {
    filename = var.filename
    content = "HUHU ${random_pet.zzzz.id}"
}

resource "random_pet" "zzzz" {
    prefix = var.prefix
    seperator = var.seperator
    length = var.length
}
```
> [!NOTE]
> We can use `terraform show` to get variable states and properties

### RESOURCE DEPENDENCIES
* If output of one variable `A` is required by another variable `B` then `B` is dependent on `A`
* While creating a resource, terraform creates `A` forst then `B`, but deletes in reverse order. This is called inplicit dependency
* But we can also specify external dependency
```
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
        model = "S"
    }
}
```

## REMOTE STATE

