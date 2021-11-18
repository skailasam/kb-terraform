# Files and Directories

## File extensions

Code in the Terraform language is stored in plain text files with the `.tf` file
extension. There is also a JSON-based variant of the language that is named
with the `.tf.json` file extension.

Files containing Terraform code are often called _configuration files_.

## Text encoding

Configuration files must always use UTF-8 encoding, and by convention usually
use Unix-style line endings (LF) rather than Windows-style line endings (CRLF),
though both are accepted.

## Directories and modules

A _module_ is a collection of `.tf` and/or `.tf.json` files kept together in a
directory.

A Terraform module only consists of the top-level configuration files in a
directory; nested directories are treated as completely separate modules, and
are not automatically include in the configuration.

Terraform evaluates all of the configuration files in a module, effectively
treating the entire module as a single document. Separating various blocks into
different files is purely for the convenience of readers and maintainers, and
has no effect on the module's behavior.

A Terraform module can use module calls to explicitly include other modules into
the configuration. These child modules can come from local directories (nested
in the parent module's directory, or anywhere else on disk), or from external
sources like the Terraform Registry.

## The root module

Terraform always runs in the context of a single _root module_. A complete
_Terraform configuration_ consists of a root module and the tree of child
modules (which includes the modules called by the root module, any modules
called by those modules, etc.).

- In Terraform CLI, the root module is the working directory where Terraform is
  invoked. (You can use command line options to specify a root module outside
  the working directory, but in practice this is rare.)
- In Terraform Cloud and Terraform Enterprise, the root module for a workspace
  defaults to the top level of the configuration directory (supplied via version
  control repository or direct upload), but the workspace settings can specify a
  subdirectory to use instead.

## Override files

Terraform normally loads all of the `.tf` and `.tf.json` files within a
directory and expects each one to define a distinct set of configuration
objects. If two files attempt to define the same object, Terraform returns an
error.

In some rare cases, it is convenient to be able to override specific portions of
an existing configuration object in a separate file. For example, a
human-edited configuration file in the Terraform language native syntax could be
partially overridden using a programatically-generated file in JSON syntax.

For these rare solutions, Terraform has special handling of any configuration
file whose name ends in `_override.tf` or `_override.tf.json`. This special
handling also applies to a file named literally `override.tf` or
`override.tf.json`.

Terraform initially skips these _override files_ when loading configuration, and
then afterwards processes each one in turn (in lexicographical order). For each
top-level block defined in an override file, Terraform attempts to find an
already-defined object corresponding to that block and then merges the override
block contents into the existing object.

Use override files only in special circumstances. Over-use of override files
hurts readability, since a reader looking only at the original files cannot
easily see that some portions of those files have been overridden without
consulting all of the override files that are present. When using override
files, use comments in the original files to warn future readers about which
override files apply changes to each block.

### Merging behavior

THe merging behavior is slightly different for each block type, and some special
constructs within certain blocks are merged in a special way.

The general rule, which applies in most cases, is:

- A top-level block in an override file merges with a block in a normal
  configuration file that has the same block header. The block _header_ is the
  block type and any quotes labels that follow it,
- Within a top-level block, an attribute argument within an override block
  replaces any argument of the same name in the original block.
- Within a top-level block, any nested blocks within an override block replace
  _all_ blocks of the same type in the original block. Any block types that do
  not appear in the override block remain from the original block.
- The contents of nested configuration blocks are not merged.
- The resulting _merged block_ must still comply with any validation rules that
  apply to the given block type.

If more than one override file defines the same top-level block, the overriding
effect is compounded, with later blocks taking precedence over earlier blocks.
Overrides are processed in order first by filename (in lexicographical order)
and then by position in each file.

The following sections describe the special merging behaviors that apply to
specific arguments within certain top-level block types.

#### Merging `resource` and `data` blocks

Within a `resource` block, the contents of any `lifecycle` nested block are
merged on an argument-by-argument basis. For example, if an override block sets
only the `create_before_destroy` argument then any `ignore_changes` argument in
the original block will be preserved.

If an overriding `resource` block contains one or more `provisioner` blocks then
any `provisioner` blocks in the original block are ignored.

If an overriding `resource` block contains a `connection` block then it
completely overrides any `connection` block present in the original block.

The `depends_on` meta-argument may not be used in override blocks, and will
produce an error.

#### Merging `variable` blocks

The arguments within a `variable` block are merged in the standard way described
above, but some special considerations apply due to the interactions between the
`type` and `default` arguments.

If the original block defines a `default` value and an override block changes
the variable's `type`, Terraform attempts to convert the default value to the
overridden type, producing an error if this conversion is not possible.

Conversely, if the original block defines a `type` and an override block changes
the `default`, the overridden default value must be compatible with the original
type specification.

#### Merging `output` blocks

The `depends_on` meta-argument may not be used in override blocks, and will
produce an error.

#### Merging `locals` blocks

Each `locals` block defines a number of named values. Overrides are applied on
a value-by-value basis, ignoring which `locals` block they are defined in.

#### Merging `terraform` blocks

The settings within `terraform` blocks are considered individually when merging.

If the `required_providers` argument is set, its value is merged on an
element-by-element basis, which allow an override block to adjust the constraint
for a single provider without affecting the constraints for other providers.

In both the `required_version` and `required_providers` settings, each override
constraint entirely replaces the constraints for the same component in the
original block. If both the base block and the override block both set
`required_version` then the constraints in the base block are entirely ignored.
