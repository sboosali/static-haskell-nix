# static-haskell-nix

With this repository you can easily build most Haskell programs into fully static Linux executables.

* results are fully static executables (`ldd` says `not a dynamic executable`)
* to make that possible, it and all dependencies (including ghc) are built against [`musl`](https://www.musl-libc.org/) instead of glibc

## History

`glibc` encourages dynamic linking to the extent that correct functionality under static linking is somewhere between difficult and bug-ridden.
For this reason, static linking, despite its many advantages (details [here](https://github.com/NixOS/nixpkgs/issues/43795)) has become less and less common.

Due to GHC's dependency on a libc, and many libraries depending on C libraries for which Linux distributions often do not include static library archive files, this situation has resulted in Haskell programs being extremely hard to produce for the common Haskeller, even though the language is generally well-suited for static linking.

This project set out to solve this.

It was inspired by a [blog post by Vaibhav Sagar](https://vaibhavsagar.com/blog/2018/01/03/static-haskell-nix/),
and a [comment by Will Dietz](https://github.com/NixOS/nixpkgs/pull/37598#issuecomment-375117019) about musl.

By now we have a nixpkgs issue on [Fully static Haskell executables](https://github.com/NixOS/nixpkgs/issues/43795) (progress on which is currently this repo, with plans to later merge it into nixpkgs), and [a merged nixpkgs overlay for static nixpkgs in general](https://github.com/NixOS/nixpkgs/pull/48803).

## Building a minimal example

`default.nix` builds an example executable (originally from https://github.com/vaibhavsagar/experiments). Run:

```
NIX_PATH=nixpkgs=https://github.com/NixOS/nixpkgs/archive/2c07921cff84dfb0b9e0f6c2d10ee2bfee6a85ac.tar.gz nix-build --no-out-link
```

This prints a path that contains the fully linked static executable in the `bin` subdirectory.

This example is so that you get the general idea.
In practice, you probably want to use the approach in the "Building arbitrary packages" section below.

## Binary caches for faster building (optional)

Install [cachix](https://cachix.org) and run `cachix use static-haskell-nix` before your `nix-build`.

If you get a warning during `cachix use`, read [this](https://github.com/cachix/cachix/issues/56#issuecomment-423820198).

If you don't want to install `cachix` for some reason or `cachix use` doesn't work, you should also be able to manually set up your `nix.conf` to have contents like this:

```
substituters = https://cache.nixos.org https://static-haskell-nix.cachix.org
trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= static-haskell-nix.cachix.org-1:Q17HawmAwaM1/BfIxaEDKAxwTOyRVhPG5Ji9K3+FvUU=
```

Note that you may not get cached results if you use a different `nix` version than I used to produce the cache (I used `2.0.4` as of writing, which you can get from [here](https://nixos.org/releases/nix/nix-2.0.4/install)).

## Building arbitrary packages

The [`static-stack`](./static-stack) directory shows how to build a fully static `stack` executable (a Haskell package with many dependencies), and makes it reasonably easy to build other packages as well.

The [`survey`](./survey) directory maintains a select set of Haskell executables that are known and not known to work with this approach; contributions are welcome to grow the set of working executables.
Run for example:

```
NIX_PATH=nixpkgs=https://github.com/NixOS/nixpkgs/archive/88ae8f7d.tar.gz nix-build --no-link survey/default.nix -A working
```

There are multiple package sets available in the survey (select via `-A`):

* `working` -- build all exes known to be working
* `notWorking` -- build all exes known to be not working (help welcome to make them work)
* `haskellPackages.somePackage` -- build a specific package from our overridden package set
