# bazel_env.bzl

The `bazel_env` rule creates a "virtual environment" for Bazel-managed tools and toolchains by making them available under stable, platform-independent paths as well as on `PATH`.
This allows all developers to share the same tool versions as used in the Bazel build for IDEs and local usage.

`bazel_env` relies on the [`direnv`](https://direnv.net/) tool to automatically set up `PATH` when entering the project directory.
When you run the `bazel_env` target, it will print instructions on how to set up `direnv` and its `.envrc` file.

## Example

The [example](examples/) includes some commonly used tools and toolchains.

## Setup

### Setup per project

1. Add a dependency on `bazel_env.bzl` to your `MODULE.bazel` file:

```starlark
bazel_dep(name = "bazel_env.bzl", dev_dependency = True)
git_override(
    module_name = "bazel_env.bzl",
    remote = "https://github.com/buildbuddy-io/bazel_env.bzl.git",
    commit = "<latest commit>",
)
```

2. Add a `bazel_env` target to a `BUILD.bazel` file (e.g. top-level or in a `tools` directory):

```starlark
load("@bazel_env.bzl", "bazel_env")

bazel_env(
    name = "bazel_env",
    toolchains = {
        "jdk": "@rules_java//toolchains:current_host_java_runtime",
    },
    tools = {
        # Tools can be specified as labels.
        "buildifier": "@buildifier_prebuilt//:buildifier",
        "go": "@rules_go//go",
        # Tool paths can also reference the Make variables provided by toolchains.
        "jar": "$(JAVABASE)/bin/jar",
        "java": "$(JAVA)",
    },
)
```

3. Run the `bazel_env` target and follow the instructions to install `direnv` and set up the `.envrc` file:

```shell
$ bazel run //:bazel_env
====== bazel_env ======

✔ direnv is installed
⚠️ bazel_env's bin directory is not in PATH. Please add the following snippet to a .envrc file next to your MODULE.bazel file:

    PATH_add bazel-out/bazel_env-opt/bin/bazel_env/bin

Then allowlist it with 'direnv allow .envrc'.
```

Multiple `bazel_env` targets can be added per project.
Note that each target will eagerly fetch and build all tools and toolchains when built, so consider splitting them up into workflow-specific targets if necessary.

### Setup for individual developers

1. Run the `bazel_env` target and follow the instructions to install `direnv` and allowlist the `.envrc` file:

```shell
$ bazel run //:bazel_env
====== bazel_env ======

✔ direnv is installed
⚠️ bazel_env's bin directory is not in PATH. Please add the following snippet to a .envrc file next to your MODULE.bazel file:

    PATH_add bazel-out/bazel_env-opt/bin/bazel_env/bin

Then allowlist it with 'direnv allow .envrc'.
```

2. Run the target again to get a list of all tools and toolchains:

```shell
====== bazel_env ======

✔ direnv is installed
✔ direnv added bazel-out/bazel_env-opt/bin/bazel_env/bin to PATH

Tools available in PATH:
  * buildifier: @buildifier_prebuilt//:buildifier
  * go:         @rules_go//go
  * jar:        $(JAVABASE)/bin/jar
  * java:       $(JAVA)

Toolchains available at stable relative paths:
  * jdk: bazel-out/bazel_env-opt/bin/bazel_env/toolchains/jdk
```

## Usage

Running the `bazel_env` target updates the tools and toolchains when necessary. In fact, building the target is sufficient, but doesn't clean up tools that were removed from the rule.

## Documentation

See the [generated documentation](docs-gen/bazel_env.md) for more information on the `bazel_env` rule.
