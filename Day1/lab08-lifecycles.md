# Lab 8: Lifecycles

Duration: 15 minutes

This lab demonstrates how to use lifecycle directives to control the order in which Terraform creates and destroys resources.

- Task 1: Use `create_before_destroy` with a simple AWS security group and instance
- Task 2: Use `prevent_destroy` with an instance

## Task 1: Use `create_before_destroy` with a simple AWS security group and instance

You'll create a simple AWS configuration with a security group and an associated EC2 instance. Provision them with `terraform`, then make a change to the security group. Observe that `apply` fails because the security group can not be destroyed and recreated while the instance lives.

You'll solve this situation by using `create_before_destroy` to create the new security group before destroying the original one.

### Step 8.1.1: Create a security group and an instance

Start by adding the following to your `main.tf`

Create an S3 security group and an instance that uses it.

```bash

resource "aws_security_group" "training" {
  name_prefix = "demo"

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.training.id]

}
```

Provision these resources.

```shell
terraform init
```

```shell
terraform apply -auto-approve
```

Update the outputs
```hcl
output "security_group_name" {
  value = aws_security_group.training.name
}

output "security_group_id" {
  value = aws_security_group.training.id
}

output "web_server_id" {
  value = aws_instance.web.*.id
}
```

The commands should succeed without error.

### Step 8.1.2: Change the name of the security group

In order to see how some resources cannot be recreated under the default `lifecyle` settings, change the name of the security group from `demo` to something like `demo-modified`.

```bash
resource "aws_security_group" "training" {
  name_prefix = "demo-modified"
  # ...
}
```

Apply this change.

```shell
terraform apply -auto-approve
```

**NOTE:** This action takes many minutes and eventually shows an error. You may choose to terminate the `apply` action with `^C` before the 15 minutes elapses. You may have to terminate twice to exit.

```shell
terraform apply -auto-approve
```

```
aws_security_group: Still destroying... (14m00s elapsed)
aws_security_group: Still destroying... (14m10s elapsed)

Error: Error applying plan:

1 error(s) occurred:

* aws_security_group.training (destroy): 1 error(s) occurred:

* aws_security_group.training: DependencyViolation: resource sg-6d36fc1c has a dependent object
	status code: 400, request id: 4665247f-165b-46fc-b8ea-9c01a23dd4e9
```

In the next step, we'll solve this problem with a `lifecycle` directive.

### Step 8.1.3: Use `create_before_destroy`

Add a `lifecycle` configuration to the `aws_security_group` resource. Specify that this resource should be created before the existing security group is destroyed.

```bash
resource "aws_security_group" "training" {
  name_prefix = "demo-modified"

  # ...

  lifecycle {
    create_before_destroy = true
  }
}
```

Now provision the new resources with the improved `lifecycle` configuration.

```shell
terraform apply -auto-approve
```

It should succeed within a short amount of time.

## Task 2: Use `prevent_destroy` with an instance

We'll demonstrate how `prevent_destroy` can be used to guard an instance from being destroyed.

### Step 8.2.1: Use `prevent_destroy`

Add `prevent_destroy = true` to the same `lifecycle` stanza where you added `create_before_destroy`.

```bash
resource "aws_security_group" "training" {
  name_prefix = "demo-modified"

  # ...

  lifecycle {
    create_before_destroy = true
    prevent_destroy = true
  }
}
```

Attempt to destroy the existing infrastructure. You should see the error that follows.

```shell
terraform destroy -force
```

```
Error: Instance cannot be destroyed

  on main.tf line 4:
   4: resource "aws_security_group" "training" {

Resource aws_security_group.training has lifecycle.prevent_destroy set, but
the plan calls for this resource to be destroyed. To avoid this error and
continue with the plan, either disable lifecycle.prevent_destroy or reduce the
scope of the plan using the -target flag.
```

### Step 8.2.2: Destroy cleanly

Now that you have finished the steps in this lab, destroy the infrastructure you have created.

Remove the `prevent_destroy` attribute.

```bash
resource "aws_security_group" "training" {
  name_prefix = "demo-modified"

  # ...

  lifecycle {
    create_before_destroy = true
    # Comment out or delete this line
    # prevent_destroy = true
  }
}
```

Finally, run `destroy`.

```shell
terraform destroy -force
```

The command should succeed and you should see a message confirming `Destroy complete! Resources: 2 destroyed.`
