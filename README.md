# `attr-env`

Create a shell environment from Nix attribute sets.
This is what `nix-shell` and `nix shell` should always have been.


# Example

Declare a flat attribute set with environment variables in some file:

```nix
# environment.nix
let
  pkgs = import <nixpkgs> {};
{
  PATH = with pkgs; [ cowsay lolcat ];
  GREETING = "Hello world";
}
```

Enter the environment:

```shell-session
$ nix-shell https://github.com/fricklerhandwerk/attr-env -A shell
[nix-shell]$ attr-env environment.nix
$ cowsay $GREETING | lolcat
```

Convert your followers to `attr-env` by sneakily replacing `nix-shell` and `nix shell` from underneath:

```
# default.nix
let
  pkgs = import <nixpkgs> {};
  attr-env = import https://github.com/fricklerhandwerk/attr-env {};
in
{
  env = {
    PATH = with pkgs; [ cowsay lolcat ];
    GREETING = "Hello world";
  }
  shell = pkgs.mkShellNoCC {
    packages = [ attr-env.package ];
    shellHook = ''
      attr-env ${toString ./.};
    ''
  }
}
```

```shell-session
$ nix-shell
$ cowsay $GREETING | lolcat
```

# Usage

`attr-env` runs `$SHELL` with environment variables declared in the specified Nix attribute sets.

```
attr-env [[<source>] [(-A|--attr) <attrpath>]...]... [--pure] [--keep <variable>]...
```

- `<source>` can be one of:
  - File system path to a Nix expression
  - URL to a file system archive that contains a `default.nix`
  - `-E` or `--expr` followed by a Nix expression

  If not set, `default.nix` in the current directory is assumed.

- `(-A|--attr) <attrpath>`

  `<attrpath>` must be an [attribute path](https://nix.dev/manual/nix/2.19/language/operators#attribute-selection) that exists in the `<source>`.

- `--pure` discards all prior environment variables, except:

  - `DISPLAY`
  - `HOME`
  - `PWD`
  - `SHELL`
  - `TERM`
  - `USER`

- `--keep <variable>` preserves the specified prior environment variable.
  `<variable>` can specify multiple variable names separated by commas.

## Environment specification

An environment is a flat attribute set.
Attribute names are taken for environment variable names.

The environment is constructed depending on the types of attribute values:
- Lists are concatenated using the colon `:` as separator.
  List elements are treated according to their type.
  Existing environment variables are prepended with the result.

- Paths are taken verbatim, and overwrite existing environment variables.
  File contents at the path are copied to the Nix store, and the path resolves to a store path.

- Every other value type is converted to a string with [`builtins.toString`](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-toString), and overwrites existing environment variables.

Multiple environments specified on the command line are constructed on top of each other, in the given order.

## Advanced usage

To put the cherry on top of Nix confusion, alias `attr-env` to `nix env`.

# Motivation

https://github.com/NixOS/nix/issues/4715
https://github.com/NixOS/nix/pull/4702
