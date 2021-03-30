# Lab 1: Modules

Duration: 20 minutes

Configuration files can be separated out into modules to better organize your configuration. This makes your code easier to read and reusable across your organization. You can also use the Public Module Registry to find pre-configured modules.

- Task 1: Explore the Pubic Module Registry and install a module
- Task 2: Refresh and rerun your Terraform configuration

## Task 1: Explore the Public Module Registry

### Step 1.1.1

HashiCorp hosts a public module registry at: https://registry.terraform.io/

The registry contains a large set of community-contributed modules that you can
use in your own configurations. Explore the registry to see what is available to
you.

### Step 1.1.2

Search for "dynamic-keys" in the public registry and uncheck the "Verified" checkbox. You should then see a module called "dynamic-keys" created by one of HashiCorp's founders, Mitchell Hashimoto. Alternatively, you can navigate directly to https://registry.terraform.io/modules/mitchellh/dynamic-keys/aws/2.0.0.

Select this module and read the content on the Readme, Inputs, Outputs, and Resources tabs. This module will generate a public and private key pair so you can SSH into your instance.

### Step 1.1.3

To integrate this module into your configuration, add this after your provider
block in `main.tf`:

```hcl
module "keypair" {
  source  = "mitchellh/dynamic-keys/aws"
  version = "2.0.0"
  path    = "${path.root}/keys"
  name    = "<INITIALS>-key"
}
```

**__This module exposes the private key information in the Terraform state and should not be used in production!__**

Now you're referring to the module, but Terraform will need to download the
module source before using it. Run the command `terraform init` to download it.

To provision the resources defined by the module, run `terraform apply`, and
answer `yes` to the confirmation prompt.

### Step 1.1.4

View the resources that were created by your module by running a `terraform state list`

```shell
terraform state list

data.aws_ami.ubuntu_16_04
aws_instance.web[0]
aws_instance.web[1]
module.keypair.aws_key_pair.generated
module.keypair.local_file.private_key_pem[0]
module.keypair.local_file.public_key_openssh[0]
module.keypair.null_resource.chmod[0]
module.keypair.tls_private_key.generated
```

## Task 2: Refresh and rerun your Terraform configuration

### Step 1.2.1
Now we'll use the _keypair_ module to install a public key on our server. In `main.tf`, add the necessary output from our key module to our server module using Terraform interpolation syntax:

```hcl
resource "aws_instance" "web" {
  count         = var.num_webs
  ami           = data.aws_ami.ubuntu_16_04.image_id
  instance_type = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  key_name               = module.keypair.key_name
  tags = {
    "Identity" = var.identity
  }
}
```

To provision the server using the public key defined by the module, run `terraform apply`, and
answer `yes` to the confirmation prompt.
