# jsonnet-bundler-ng

The jsonnet-bundler-ng is a fork of [jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler),
a package manager for [jsonnet](https://jsonnet.org/).

## Install

```
go install -a github.com/dadav/jsonnet-bundler-ng/cmd/jb-ng@latest
```

## Package Install

- [Arch Linux AUR](https://aur.archlinux.org/packages/jsonnet-bundler-ng-bin)

## Features

- Fetches transitive dependencies
- Can vendor subtrees, as opposed to whole repositories

## Current Limitations

- Always downloads entire dependent repositories, even when updating
- If two dependencies depend on the same package (diamond problem), they must require the same version

## Example Usage

Initialize your project:

```sh
mkdir myproject
cd myproject
jb-ng init
```

The existence of the `jsonnetfile.json` file means your directory is now a
jsonnet-bundler package that can define dependencies.

To depend on another package (another Github repository):
_Note that your dependency need not be initialized with a `jsonnetfile.json`.
If it is not, it is assumed it has no transitive dependencies._

```sh
jb-ng install https://github.com/anguslees/kustomize-libsonnet
```

Now write `myconfig.jsonnet`, which can import a file from that package.
Remember to use `-J vendor` when running Jsonnet to include the vendor tree.

```jsonnet
local kustomize = import 'kustomize-libsonnet/kustomize.libsonnet';

local my_resource = {
  metadata: {
    name: 'my-resource',
  },
};

kustomize.namePrefix('staging-')(my_resource)
```

To depend on a package that is in a subtree of a Github repo (this package also
happens to bring in a transitive dependency):

```sh
jb-ng install https://github.com/prometheus-operator/prometheus-operator/jsonnet/prometheus-operator
```

_Note that if you are copy pasting from the Github website's address bar,
remove the `tree/master` from the path._

If pushed to Github, your project can now be referenced from other packages in
the same way, with its dependencies fetched automatically.

## All command line flags

```bash
usage: jb-ng [<flags>] <command> [<args> ...]

A jsonnet package manager

Flags:
  -h, --help     Show context-sensitive help (also try --help-long and
                 --help-man).
      --version  Show application version.
      --jsonnetpkg-home="vendor"
                 The directory used to cache packages in.
  -q, --quiet    Suppress any output from git command.

Commands:
  help [<command>...]
    Show help.

  init
    Initialize a new empty jsonnetfile

  registry add [<name>] [<description>] [<url>] [<file>]
    Add a new registry

  registry rm [<name>]
    Remove a registry

  registry update
    Update registry data

  registry list
    List registries

  registry search [<flags>] [<query>]
    Search package in registries

  install [<flags>] [<uris>...]
    Install new dependencies. Existing ones are silently skipped

  update [<uris>...]
    Update all or specific dependencies.

  rewrite
    Automatically rewrite legacy imports to absolute ones
```

## Design

This is an implemention of the design specified in this document: https://docs.google.com/document/d/1czRScSvvOiAJaIjwf3CogOULgQxhY9MkiBKOQI1yR14/edit#heading=h.upn4d5pcxy4c
