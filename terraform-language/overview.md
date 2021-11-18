# Terraform Language Documentation

## About the Terraform Language

The main purpose of the Terraform language is declaring resources, which
represent infrastructure objects. All other language features exist only to
make the definition of resources more flexible and convenient.

A _Terraform configuration_ is a complete document in the Terraform language
that tells Terraform how to manage a given collection of infrastructure. A
configuration can consist of multiple files and directories.

The syntax of the Terraform language consists of only a few basic elements:

```terraform
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```
