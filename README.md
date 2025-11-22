# Claude Code Overlay

A Nix flake overlay that provides pre-built Claude Code CLI binaries from official Anthropic releases.

This overlay downloads binaries directly from Anthropic's distribution servers, similar to [mitchellh/zig-overlay](https://github.com/mitchellh/zig-overlay).

## Features

- ✅ Automatic updates via GitHub Actions (hourly checks)
- ✅ Multi-platform support: Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- ✅ Direct downloads from official Anthropic servers
- ✅ SHA256 checksum verification
- ✅ Flake and non-flake support

## Usage

> **Note:** Claude Code has an unfree licence. You need to allow unfree packages to use this overlay.

### With Flakes

#### Run directly

```bash
NIXPKGS_ALLOW_UNFREE=1 nix run --impure github:ryoppippi/claude-code-overlay
```

#### Add to your flake

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    claude-code-overlay.url = "github:ryoppippi/claude-code-overlay";
  };

  outputs = { self, nixpkgs, claude-code-overlay, ... }: {
    nixosConfigurations.yourhostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        ({ pkgs, ... }: {
          nixpkgs.config.allowUnfree = true;
          nixpkgs.overlays = [ claude-code-overlay.overlays.default ];
          environment.systemPackages = [ pkgs.claudepkgs.default ];
        })
      ];
    };
  };
}
```

#### Using in `home-manager`

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    claude-code-overlay.url = "github:ryoppippi/claude-code-overlay";
  };

  outputs = { nixpkgs, home-manager, claude-code-overlay, ... }: {
    homeConfigurations."user@hostname" = home-manager.lib.homeManagerConfiguration {
      pkgs = import nixpkgs {
        system = "x86_64-linux";
        config.allowUnfree = true;
        overlays = [ claude-code-overlay.overlays.default ];
      };

      modules = [
        {
          home.packages = [ pkgs.claudepkgs.default ];
        }
      ];
    };
  };
}
```

### Without Flakes

```nix
let
  claude-code-overlay = import (builtins.fetchTarball {
    url = "https://github.com/ryoppippi/claude-code-overlay/archive/main.tar.gz";
  });
  pkgs = import <nixpkgs> {
    config.allowUnfree = true;
    overlays = [ claude-code-overlay.overlays.default ];
  };
in
  pkgs.claudepkgs.default
```

## Available Packages

- `default` - Latest stable version
- `<version>` - Specific version (e.g., `2.0.50`)

## How It Works

1. The `update` script fetches the latest stable version from Anthropic's release server
2. It retrieves the manifest.json containing SHA256 checksums for all platforms
3. GitHub Actions runs the update script hourly and commits any changes
4. The flake provides pre-built binaries for all supported platforms

## Supported Platforms

- `x86_64-linux`
- `aarch64-linux`
- `x86_64-darwin` (macOS Intel)
- `aarch64-darwin` (macOS Apple Silicon)

## Development

### Update sources manually

```bash
nix develop
./update
```

### Test the overlay

```bash
NIXPKGS_ALLOW_UNFREE=1 nix build --impure
./result/bin/claude --version
```

## Credits

- Inspired by [mitchellh/zig-overlay](https://github.com/mitchellh/zig-overlay)
- Claude Code CLI by [Anthropic](https://anthropic.com)

## Licence

MIT
