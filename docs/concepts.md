# Concepts

## HCL

Hashicorp Configuration Language is a superset of JSON. It's a
declarative way to say which resources with which configuration it
should create, update and delete.

The JSON format is also accepted but it's less human-friendly to write.

HCL is more flexible, supports more types, internal function calls like
`value = lookup(map, key [, default])` and `local` blocks where data can
be manipulated before being used in resource definitions.

Terraform amalgamates all `.tf` files in
a directory and the configuration of the
resources inside all `.tf` files is considered the **desired state**.


## Resources

Each
[provider](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L32)
exposes a list of
[resources](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L235),
each of them has a
[schema](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/ec2/ec2_instance.go#L61).
That schema controls what fields are supported and can be used to
control what type of values are allowed/valid and how changes should be
handled. For example, if changing field called `name` requires a
[recreation](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/ec2/ec2_instance.go#LL63C5-L63C24)
(destroy + create) or just an update (for example, as in a tag).


## Data Sources

Resources are things you want to **manage**. Data sources are things you
want to **load**.

Data sources are used to make existing resources' properties available
in a configuration. For example if you need the name of a VPC that
already exists in your Terraform configuration,
you can load its fields into a `aws_vpc`, and from there you can use its
`id` or any other fields (below we also use `cidr_block`) like this:

```hcl
data "aws_vpc" "workshop" {
  name = "workshop"
}

resource "aws_subnet" "example" {
  vpc_id            = data.aws_vpc.workshop.id
  availability_zone = "eu-west-1a"
  cidr_block        = cidrsubnet(data.aws_vpc.workshop.cidr_block, 4, 1)
}
```


## Modules

A local file system directory with terraform files is considered a
module. A git repo also is considered a module. There's always a root
module, which is your current working directory. But you can from this
root module can "call" other modules. And those modules can in turn
"call" other modules.

One passes inputs/configuration to other modules via `variable`s. A
module returns values to other modules via `output`s. This way you can
gather common terraform code into shared modules and extract the
differences into variables.

Example:

```hcl
# File main.tf
module "local_module" {
  source = "./hello"

  name = "World"
}

# An output from the root module will be shown in Terraform's output stream.
output "result" {
  value = module.local_module.greeting
}
```

```hcl
# File hello/this-name-doesnt-matter.tf:
variable "name" {}

variable "prefix" {
  type = string
  default = " there"
}

output "greeting" {
  value = "Hello${var.prefix} ${var.name}!"
}
```


## Types

Terraform exposes a few familiar types:

Simple:

- null: `null`
- booleans: `true`, `false`
- number: `12`, `1.2` -- yes, it's a float
- string: `"hello"`

Collections:
- `list`
- `map`
- `object`
