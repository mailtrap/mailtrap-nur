# mailtrap-nur

Mailtrap [NUR](https://github.com/nix-community/NUR)-style repository — Nix packages for Mailtrap tooling.

Layout follows [nix-community/nur-packages-template](https://github.com/nix-community/nur-packages-template).

## Packages

| Attribute | Description |
| --- | --- |
| `mailtrap-local` | Local email sandbox + catcher ([mailtrap/mailtrap-local](https://github.com/mailtrap/mailtrap-local)) |

`mailtrap-local` installs the prebuilt release binary (same artifacts as Homebrew / GitHub Releases), so you get the real embedded Web UI. Platforms: `x86_64`/`aarch64` on Linux and Darwin.

`pkgs/mailtrap-local/default.nix` is owned by goreleaser in [mailtrap-local](https://github.com/mailtrap/mailtrap-local): each tagged release opens a PR here (same flow as the Homebrew tap).

## Install

Flake (overlay into your own flake):

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    mailtrap-nur.url = "github:mailtrap/mailtrap-nur";
  };

  outputs = { nixpkgs, mailtrap-nur, ... }:
    let
      system = "aarch64-darwin"; # or x86_64-linux, …
      pkgs = import nixpkgs {
        inherit system;
        overlays = [ mailtrap-nur.overlays.default ];
      };
    in
    {
      packages.${system}.default = pkgs.mailtrap-local;
      # or: packages.${system}.mailtrap-local = pkgs.mailtrap-local;
    };
}
```

One-shot:

```bash
nix profile add github:mailtrap/mailtrap-nur#mailtrap-local
# Nix older than 2.25: nix profile install …
# or, without installing:
nix run github:mailtrap/mailtrap-nur#mailtrap-local
```

Both `mailtrap-local` and the `mailtrap-sendmail` symlink land in the profile.

Without flakes (`shell.nix`):

```nix
let
  pkgs = import <nixpkgs> { };
  mailtrap = import (builtins.fetchTarball "https://github.com/mailtrap/mailtrap-nur/archive/main.tar.gz") {
    inherit pkgs;
  };
in
  mailtrap.mailtrap-local
```

Or as a nixpkgs overlay (e.g. NixOS / home-manager `nixpkgs.overlays`):

```nix
[
  (import "${builtins.fetchTarball "https://github.com/mailtrap/mailtrap-nur/archive/main.tar.gz"}/overlay.nix")
]
```

## Develop

```bash
nix-build -A mailtrap-local
nix build .#mailtrap-local
```

## Add another package

1. Create `pkgs/<name>/default.nix` (use `pkgs.callPackage` — take deps from the `pkgs` argument, do not `import <nixpkgs>` inside the package).
2. Export it from root `default.nix`.
3. Mark broken packages with `meta.broken = true` so CI stays green.
4. If the package only supports some platforms, keep `flake.nix` `systems` aligned with those binaries.
