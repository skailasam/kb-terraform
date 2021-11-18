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

- _Blocks_ are containers for other content and usually represent the
  configuration of some kind of object, like a resource. Blocks have a _block
  type_, can have zero or more _labels_, and have a _body_ that contains any
  number of arguments and nested blocks. Most of Terraform's features are
  controlled by to-level blocks in a configuration file.
- _Arguments_ assign a value to a name. They appear within blocks.
- _Expressions_ represent a value, either literally or by referencing and
  combining other values. They appear as values for arguments, or within other
  expressions.

The Terraform language is declarative, describing an intended goal rather than
the steps to reach that goal. The ordering of blocks and the files they are
organized into are generally not significant; Terraform only considers implicit
and explicit relationships between resources when determining an order of
operations.
