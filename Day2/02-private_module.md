# Lab 2: Private Modules

Duration: 20 minutes

Configuration files can be separated out into modules to better organize your configuration. This makes your code easier to read and reusable across your organization.

- Task 1: Utilize a Private Module
- Task 2: Refresh and rerun your Terraform configuration

## Task 1: Explore the Private Module Registry

### Step 2.1.1

Terraform Cloud / Enterprise hosts a private module registry.  The registry contains modules that are private and can be used within your organization.

### Step 2.1.2

Search for "server" in the private registry and uncheck the "Verified" checkbox. You should then see a module called "server" that is marked as `Private`

Select this module and read the content on the Readme, Inputs, Outputs, and Resources tabs. This module will generate an EC2 server on AWS.

### Step 2.1.3

To integrate this module into your configuration, add this to the end of your `main.tf`:

```hcl
module "server" {
  source  = "app.terraform.io/training-nyl/server/aws"
  version = "0.1.1"
  environment = "dev"
  identity = var.identity
  key_name = module.keypair.key_name
  private_key = module.keypair.private_key_pem
  server_count = var.num_webs
  subnet_ids = [var.subnet_id]
  ubuntu_version = "18"
  vpc_security_group_ids = var.vpc_security_group_ids
}
```

Validate the configuration by running a `terraform init` and `terraform validate`.

To provision the server using the private module run `terraform apply`, and answer `yes` to the confirmation prompt.

### Step 1.1.4

Our outputs should be defined in the root module as well. At the bottom of your
`main.tf` configuration, modify the public IP and public DNS outputs to match
the following. Notice the difference in interpolation now that the information
is being delivered by a module.

```hcl
output "public_ip_module" {
  value = module.server.public_ip
}

output "public_dns_module" {
  value = module.server.public_dns
}
```
