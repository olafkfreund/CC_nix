# Package Existing Software for Nix

Create a new Nix package derivation for existing software with automatic dependency detection and validation.

## Quick Start

Just tell me:
1. **Software name** (e.g., "mycli", "awesome-tool")
2. **Source location** (GitHub URL, tarball URL, or "help me find it")
3. **Programming language** (C/C++, Python, Rust, Go, etc.)

I'll guide you through the rest interactively.

## What I'll Do

### 1. Research & Planning (Automatic)
- Analyze the software's build system (autoconf, cmake, meson, cargo, etc.)
- Identify likely dependencies from build files
- Determine appropriate fetcher (fetchFromGitHub, fetchurl, etc.)
- Check if package already exists in nixpkgs

### 2. Choose Package Location
**Options:**
- `pkgs/` directory in your project (recommended for custom packages)
- Overlay in `overlays/` (for extending nixpkgs)
- Standalone `default.nix` (for testing)

### 3. Create Package Derivation

Using this proven structure:
```nix
{ lib
, stdenv
, fetchFromGitHub
, # ... build dependencies
}:

stdenv.mkDerivation rec {
  pname = "package-name";
  version = "1.0.0";

  src = fetchFromGitHub {
    owner = "owner-name";
    repo = "repo-name";
    rev = "v${version}";
    hash = "sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=";
  };

  nativeBuildInputs = [
    # Build-time dependencies (compilers, build tools)
  ];

  buildInputs = [
    # Runtime dependencies (libraries)
  ];

  # Custom build phases if needed
  # configurePhase, buildPhase, installPhase

  meta = with lib; {
    description = "Short description";
    homepage = "https://example.com";
    license = licenses.mit;
    maintainers = with maintainers; [ your-name ];
    platforms = platforms.all;
    mainProgram = "binary-name";
  };
}
```

### 4. Hash Calculation (Automatic)
- Generate correct hash using `nix-prefetch-url` or `nix hash`
- Handle hash format conversion (sha256 to SRI format)
- Update derivation with correct hash

### 5. Dependency Detection (Interactive)
- Build the package and capture error messages
- Identify missing dependencies from build failures
- Search nixpkgs for required packages
- Add dependencies iteratively

### 6. Build Phase Customization (If Needed)
- Override phases for non-standard build systems
- Add custom install steps
- Handle special cases (no install target, custom paths)

### 7. Testing & Validation
- Build the package: `nix-build -A package-name`
- Test the binary: `./result/bin/program --version`
- Verify all dependencies are included
- Check for runtime errors

### 8. Integration
- Add to `pkgs/default.nix` or overlay
- Create usage example
- Document any special requirements

## Packaging Workflow

### Phase 1: Source Fetching

**For GitHub repositories (recommended):**
```nix
src = fetchFromGitHub {
  owner = "username";
  repo = "project";
  rev = "v1.0.0";  # or git commit hash
  hash = "";  # Leave empty initially
};
```

**For release tarballs:**
```nix
src = fetchurl {
  url = "https://example.com/software-1.0.0.tar.gz";
  hash = "";  # Leave empty initially
};
```

**For other sources:**
- `fetchgit` - Git repositories (non-GitHub)
- `fetchzip` - ZIP archives
- `fetchPatch` - Patches

### Phase 2: Hash Generation

I'll automatically:
1. Build with empty hash
2. Extract correct hash from error message
3. Convert to SRI format if needed
4. Update derivation

**Manual command:**
```bash
nix-prefetch-url --unpack https://github.com/owner/repo/archive/v1.0.0.tar.gz
```

### Phase 3: Dependency Discovery

**Build and iterate:**
1. Run `nix-build`
2. Read error messages for missing files/libraries
3. Search nixpkgs: `nix search nixpkgs package-name`
4. Add to `buildInputs` or `nativeBuildInputs`
5. Repeat until build succeeds

**Common dependencies:**
- **C/C++**: `pkg-config`, `cmake`, `autoconf`, `gcc`, `clang`
- **Python**: `python3`, `python3Packages.*`
- **Rust**: `cargo`, `rustc`, `rustPlatform.buildRustPackage`
- **Go**: `buildGoModule`
- **Node**: `nodejs`, `nodePackages.*`

### Phase 4: Build System Detection

I'll identify and configure:

**Autotools (./configure && make):**
```nix
nativeBuildInputs = [ pkg-config ];
configureFlags = [ "--enable-feature" ];
```

**CMake:**
```nix
nativeBuildInputs = [ cmake ];
cmakeFlags = [ "-DOPTION=ON" ];
```

**Meson:**
```nix
nativeBuildInputs = [ meson ninja ];
mesonFlags = [ "-Dfeature=enabled" ];
```

**Make only:**
```nix
makeFlags = [ "PREFIX=$(out)" ];
```

**Custom build:**
```nix
buildPhase = ''
  runHook preBuild
  ./custom-build.sh
  runHook postBuild
'';
```

### Phase 5: Custom Installation

When software lacks standard install:
```nix
installPhase = ''
  runHook preInstall

  mkdir -p $out/bin
  cp program $out/bin/

  mkdir -p $out/share/doc/${pname}
  cp README.md LICENSE $out/share/doc/${pname}/

  runHook postInstall
'';
```

**Common install patterns:**
- Binaries → `$out/bin/`
- Libraries → `$out/lib/`
- Headers → `$out/include/`
- Documentation → `$out/share/doc/`
- Man pages → `$out/share/man/`

## Language-Specific Helpers

### Rust Packages
```nix
{ rustPlatform, fetchFromGitHub }:

rustPlatform.buildRustPackage rec {
  pname = "rust-tool";
  version = "1.0.0";

  src = fetchFromGitHub {
    owner = "owner";
    repo = "repo";
    rev = "v${version}";
    hash = "sha256-...";
  };

  cargoHash = "sha256-...";  # Run with empty string first

  meta = { ... };
}
```

### Python Packages
```nix
{ python3Packages, fetchPypi }:

python3Packages.buildPythonPackage rec {
  pname = "python-package";
  version = "1.0.0";

  src = fetchPypi {
    inherit pname version;
    hash = "sha256-...";
  };

  propagatedBuildInputs = with python3Packages; [
    requests
    click
  ];

  meta = { ... };
}
```

### Go Packages
```nix
{ buildGoModule, fetchFromGitHub }:

buildGoModule rec {
  pname = "go-tool";
  version = "1.0.0";

  src = fetchFromGitHub {
    owner = "owner";
    repo = "repo";
    rev = "v${version}";
    hash = "sha256-...";
  };

  vendorHash = "sha256-...";  # or null for no vendor

  meta = { ... };
}
```

## Validation & Testing

### 1. Syntax Check (< 5s)
```bash
nix-instantiate --parse default.nix
```

### 2. Build Package (1-5min)
```bash
nix-build -A package-name

# Or in flake
nix build .#package-name
```

### 3. Test Executable
```bash
./result/bin/program --version
./result/bin/program --help

# Check runtime dependencies
ldd ./result/bin/program
```

### 4. Verify Outputs
```bash
ls -la result/
tree result/

# Check for unnecessary files
nix path-info -rsSh ./result
```

### 5. Check Meta Information
```bash
nix eval .#package-name.meta --json | jq
```

## Advanced Features

### Patches
Apply patches before building:
```nix
patches = [
  ./fix-build.patch
  (fetchpatch {
    url = "https://github.com/project/commit/abc123.patch";
    hash = "sha256-...";
  })
];
```

### Conditional Dependencies
```nix
buildInputs = [ dependency1 ]
  ++ lib.optional stdenv.isDarwin darwin.apple_sdk.frameworks.Security
  ++ lib.optional withFeature dependency2;
```

### Multiple Outputs
```nix
outputs = [ "out" "dev" "doc" ];

# In installPhase:
# Binaries → $out
# Headers → $dev
# Docs → $doc
```

### Post-Install Fixup
```nix
postInstall = ''
  wrapProgram $out/bin/program \
    --prefix PATH : ${lib.makeBinPath [ dependency1 dependency2 ]}
'';
```

## Common Issues & Solutions

### Issue 1: Hash Mismatch
**Error:** "hash mismatch in fixed-output derivation"
**Solution:** Copy correct hash from error message, convert to SRI format

### Issue 2: Missing Dependencies
**Error:** "library not found" or "command not found"
**Solution:** Search nixpkgs, add to buildInputs or nativeBuildInputs

### Issue 3: Install Phase Fails
**Error:** "No rule to make target 'install'"
**Solution:** Override installPhase with custom installation

### Issue 4: Binary Not Found
**Error:** "cannot find binary after build"
**Solution:** Check build output, manually copy to $out/bin

### Issue 5: Runtime Library Errors
**Error:** "cannot open shared object file"
**Solution:** Add missing runtime dependency to buildInputs

## Integration Options

### Option 1: Standalone Package (Testing)
```bash
# Create pkgs/mypackage/default.nix
nix-build pkgs/mypackage
```

### Option 2: Project Overlay
```nix
# overlays/default.nix
final: prev: {
  mypackage = final.callPackage ../pkgs/mypackage { };
}

# flake.nix
overlays.default = import ./overlays;
```

### Option 3: Direct in flake.nix
```nix
packages.mypackage = pkgs.callPackage ./pkgs/mypackage { };
```

## Best Practices

### DO ✅
1. **Use semantic versioning** in pname and version
2. **Document all options** in meta attributes
3. **Test the built package** before committing
4. **Use SRI hash format** (sha256-...) for modern Nix
5. **Include license** in meta
6. **Specify platforms** explicitly
7. **Set mainProgram** for packages with binaries
8. **Run hooks** (runHook pre/post) in custom phases
9. **Use lib.makeBinPath** for PATH dependencies
10. **Check upstream** for build instructions

### DON'T ❌
1. **Hardcode paths** - use $out and derivation outputs
2. **Skip meta attributes** - always include description, license
3. **Use `rec`** - prefer let bindings
4. **Copy-paste hashes** without verifying
5. **Forget nativeBuildInputs** - separate build-time dependencies
6. **Ignore build warnings** - they often indicate real issues
7. **Skip testing** - always test the built package
8. **Use outdated hash formats** - migrate to SRI format

## Success Checklist

- [ ] Source fetcher configured (fetchFromGitHub, fetchurl, etc.)
- [ ] Correct hash calculated and set
- [ ] All dependencies identified and added
- [ ] Build succeeds without errors
- [ ] Binary/library outputs verified
- [ ] Runtime test passes
- [ ] Meta attributes complete (description, license, homepage)
- [ ] mainProgram set if applicable
- [ ] Integrated into project structure
- [ ] Documentation and usage example provided

## Speed Optimization

**Typical workflow:**
- Setup and research: 2-3 minutes
- Initial derivation: 1-2 minutes
- Hash calculation: 30 seconds
- Dependency iteration: 3-5 minutes (depends on complexity)
- Testing: 1-2 minutes
- Integration: 1 minute

**Total Time**: 10-15 minutes for simple packages, 20-30 for complex ones

## Common Packaging Patterns

### Pattern 1: Simple Binary
```nix
stdenv.mkDerivation {
  installPhase = ''
    install -Dm755 binary $out/bin/binary
  '';
}
```

### Pattern 2: Multiple Binaries
```nix
installPhase = ''
  mkdir -p $out/bin
  for bin in tool1 tool2 tool3; do
    install -Dm755 $bin $out/bin/$bin
  done
'';
```

### Pattern 3: With Data Files
```nix
installPhase = ''
  mkdir -p $out/{bin,share/${pname}}
  cp program $out/bin/
  cp -r data/* $out/share/${pname}/
'';
```

### Pattern 4: Library Package
```nix
installPhase = ''
  mkdir -p $out/{lib,include}
  cp *.so $out/lib/
  cp *.h $out/include/
'';
```

## Finding Dependencies

### Method 1: Search nixpkgs
```bash
# Command line
nix search nixpkgs package-name

# Or use search.nixos.org
```

### Method 2: Error-Driven Discovery
Build and read errors:
- "file not found: pkg-config" → add `pkg-config`
- "library not found: -lssl" → add `openssl`
- "header not found: zlib.h" → add `zlib.dev`

### Method 3: Check Nixpkgs
```bash
# Clone nixpkgs
git clone https://github.com/NixOS/nixpkgs.git

# Search for similar packages
rg "pname = \"similar-tool\"" pkgs/
```

## Meta Attributes Reference

```nix
meta = with lib; {
  description = "Short one-line description";
  longDescription = ''
    Detailed multi-line description
    with more information.
  '';
  homepage = "https://example.com";
  changelog = "https://example.com/releases/v${version}";
  license = licenses.mit;  # or licenses.gpl3, etc.
  maintainers = with maintainers; [ your-name ];
  platforms = platforms.linux;  # or platforms.all, platforms.unix
  mainProgram = "binary-name";
  broken = false;
};
```

## Real-World Example Walkthrough

**Example: Package a simple C CLI tool from GitHub**

1. **Initial info:**
   - Name: `hello-world`
   - Source: https://github.com/example/hello-world
   - Language: C with Makefile

2. **I create:**
```nix
{ lib, stdenv, fetchFromGitHub }:

stdenv.mkDerivation rec {
  pname = "hello-world";
  version = "1.0.0";

  src = fetchFromGitHub {
    owner = "example";
    repo = "hello-world";
    rev = "v${version}";
    hash = "";  # Will fix
  };

  makeFlags = [ "PREFIX=$(out)" ];

  meta = with lib; {
    description = "Simple hello world CLI tool";
    homepage = "https://github.com/example/hello-world";
    license = licenses.mit;
    platforms = platforms.unix;
    mainProgram = "hello";
  };
}
```

3. **Build to get hash:**
```bash
nix-build -A hello-world
# Error provides correct hash
```

4. **Update hash, rebuild, test:**
```bash
nix-build -A hello-world
./result/bin/hello --version
```

Done! ✅

Ready to package your software? Just tell me the name and where to find it!
