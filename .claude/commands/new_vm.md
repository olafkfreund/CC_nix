# Create New NixOS Virtual Machine

Create a declarative NixOS virtual machine configuration with custom resource allocation and network settings.

## Quick Start

Just tell me:
1. **VM purpose** (e.g., "development", "testing", "web server", "database")
2. **Memory** (e.g., "4GB", "8192" - default: 2GB)
3. **CPU cores** (e.g., "4" - default: 2)
4. **Network type** (e.g., "internal", "external", "both" - default: internal)

Or just say: "Create a VM" and I'll ask for details interactively!

## What I'll Do

### 1. Gather VM Requirements (Interactive)

I'll ask about:
- **Operating system type**: Desktop, Server, or Minimal
- **Resource allocation**: Memory and CPU
- **Network configuration**: Internal only or external access
- **Services needed**: Web server, database, monitoring, etc.
- **Packages required**: Development tools, utilities, applications

If you don't specify services/packages initially, I'll ask!

### 2. Create VM Configuration File

**Location options:**
- `vms/{vm-name}/configuration.nix` (recommended)
- `vm-configuration.nix` (standalone)
- Custom path you specify

### 3. Generate Complete Configuration

Using this proven template:

```nix
{ config, lib, pkgs, modulesPath, ... }:
{
  imports = [
    (modulesPath + "/virtualisation/qemu-vm.nix")
  ];

  # VM Configuration
  virtualisation = {
    # Memory allocation
    memorySize = 2048;  # MB

    # CPU cores
    cores = 2;

    # Disk configuration
    diskSize = 20480;  # MB (20GB)

    # Graphics and display
    graphics = true;

    # Resolution
    resolution = { x = 1920; y = 1080; };

    # QEMU options
    qemu = {
      options = [
        # Network configuration
        "-net nic,model=virtio"
        "-net user,hostfwd=tcp::2222-:22"  # SSH forwarding
      ];

      # Enable KVM if available
      virtualisation.useKVM = true;
    };

    # Shared folders (optional)
    sharedDirectories = {
      share = {
        source = "$HOME/vm-share";
        target = "/mnt/share";
      };
    };

    # Forward ports
    forwardPorts = [
      { from = "host"; host.port = 8080; guest.port = 80; }
    ];
  };

  # Boot configuration
  boot.loader = {
    systemd-boot.enable = true;
    efi.canTouchEfiVariables = true;
  };

  # Networking
  networking = {
    hostName = "nixos-vm";
    firewall = {
      enable = true;
      allowedTCPPorts = [ 22 ];
    };
  };

  # Users
  users.users.nixos = {
    isNormalUser = true;
    initialPassword = "nixos";
    extraGroups = [ "wheel" "networkmanager" ];
  };

  # Enable sudo for wheel group
  security.sudo.wheelNeedsPassword = false;

  # Services
  services = {
    openssh = {
      enable = true;
      settings.PasswordAuthentication = true;
    };
  };

  # System packages
  environment.systemPackages = with pkgs; [
    vim
    git
    curl
    htop
  ];

  # System version
  system.stateVersion = "24.11";
}
```

### 4. Configure Based on VM Type

**Desktop VM (GNOME/X11):**
```nix
services.xserver = {
  enable = true;
  displayManager.gdm.enable = true;
  desktopManager.gnome.enable = true;
};

# Additional packages for desktop
environment.systemPackages = with pkgs; [
  firefox
  gnome.gnome-terminal
  gnome.nautilus
];
```

**Desktop VM (Sway/Wayland):**
```nix
programs.sway.enable = true;

# Optional: Better QEMU display with Wayland
virtualisation.qemu.options = [
  "-device virtio-vga-gl"
  "-display gtk,gl=on"
];

environment.systemPackages = with pkgs; [
  firefox
  foot
  wofi
];
```

**Server VM (Headless):**
```nix
# Minimal server setup
services.openssh.enable = true;

# No graphics
virtualisation.graphics = false;

# Serial console access
boot.kernelParams = [ "console=ttyS0" ];
```

**Development VM:**
```nix
# Development tools
environment.systemPackages = with pkgs; [
  git
  vim
  tmux
  gcc
  gnumake
  python3
  nodejs
  docker
];

# Enable Docker
virtualisation.docker.enable = true;
users.users.nixos.extraGroups = [ "docker" ];
```

### 5. Network Configuration Options

**Internal Only (Default):**
```nix
# User-mode networking (NAT)
# VM can access internet but not accessible from host
virtualisation.qemu.networkingOptions = [
  "-net nic,model=virtio"
  "-net user"
];
```

**External Access (Port Forwarding):**
```nix
# Forward specific ports to VM
virtualisation.forwardPorts = [
  { from = "host"; host.port = 8080; guest.port = 80; }    # HTTP
  { from = "host"; host.port = 8443; guest.port = 443; }   # HTTPS
  { from = "host"; host.port = 2222; guest.port = 22; }    # SSH
  { from = "host"; host.port = 5432; guest.port = 5432; }  # PostgreSQL
];

networking.firewall.allowedTCPPorts = [ 80 443 22 5432 ];
```

**Bridged Network (Full Access):**
```nix
# Bridge to host network (requires root)
virtualisation.qemu.networkingOptions = [
  "-net nic,model=virtio"
  "-net bridge,br=virbr0"
];
```

**Multiple Networks:**
```nix
virtualisation.qemu.networkingOptions = [
  # Internal network
  "-net nic,vlan=0,model=virtio"
  "-net user,vlan=0"
  # External network
  "-net nic,vlan=1,model=virtio"
  "-net tap,vlan=1,ifname=tap0"
];
```

### 6. Service Templates

**Web Server:**
```nix
services.nginx = {
  enable = true;
  virtualHosts."localhost" = {
    root = "/var/www";
  };
};

networking.firewall.allowedTCPPorts = [ 80 443 ];
```

**Database (PostgreSQL):**
```nix
services.postgresql = {
  enable = true;
  package = pkgs.postgresql_15;
  ensureDatabases = [ "mydb" ];
  ensureUsers = [{
    name = "myuser";
    ensureDBOwnership = true;
  }];
  authentication = ''
    host all all 0.0.0.0/0 md5
  '';
};

networking.firewall.allowedTCPPorts = [ 5432 ];
```

**Monitoring (Prometheus + Grafana):**
```nix
services.prometheus = {
  enable = true;
  port = 9090;
  exporters = {
    node = {
      enable = true;
      port = 9100;
    };
  };
};

services.grafana = {
  enable = true;
  settings.server = {
    http_addr = "0.0.0.0";
    http_port = 3000;
  };
};

networking.firewall.allowedTCPPorts = [ 9090 3000 ];
```

**Container Runtime (Docker):**
```nix
virtualisation.docker = {
  enable = true;
  enableOnBoot = true;
};

users.users.nixos.extraGroups = [ "docker" ];
```

**Container Runtime (Podman):**
```nix
virtualisation.podman = {
  enable = true;
  dockerCompat = true;
  defaultNetwork.settings.dns_enabled = true;
};
```

### 7. Build VM

I'll provide commands to build and run your VM:

```bash
# Build the VM
nix-build '<nixpkgs/nixos>' -A vm \
  -I nixos-config=./vm-configuration.nix \
  -I nixpkgs=channel:nixos-24.11

# Or using flakes
nix build .#nixosConfigurations.my-vm.config.system.build.vm
```

### 8. Run VM

**Graphical Mode:**
```bash
# Run with GUI
./result/bin/run-nixos-vm

# With specific resolution
QEMU_OPTS="-vga virtio -display gtk,gl=on" ./result/bin/run-nixos-vm
```

**Headless Mode (Server):**
```bash
# Run without graphics
./result/bin/run-nixos-vm -nographic

# Run in background
./result/bin/run-nixos-vm -nographic &

# Access via serial console
# Use Ctrl-A then X to exit
```

**With Custom Options:**
```bash
# Custom memory and CPU
QEMU_OPTS="-m 4096 -smp 4" ./result/bin/run-nixos-vm

# Enable KVM acceleration
QEMU_OPTS="-enable-kvm" ./result/bin/run-nixos-vm
```

### 9. VM Management Commands

**Access VM:**
```bash
# SSH into VM (if port forwarding configured)
ssh -p 2222 nixos@localhost

# Serial console (headless mode)
# Ctrl-A, X to exit
```

**Reset VM:**
```bash
# Delete persistent state
rm nixos.qcow2

# Rebuild and restart fresh
./result/bin/run-nixos-vm
```

**Shutdown VM:**
```bash
# From inside VM
sudo shutdown now

# Or via QEMU monitor (Ctrl-Alt-2)
# Type: system_powerdown
```

## Resource Allocation Guidelines

### Memory Allocation

| Use Case | Recommended RAM | Minimal RAM |
|----------|----------------|-------------|
| Minimal Server | 512MB | 256MB |
| Basic CLI | 1GB | 512MB |
| Development | 4GB | 2GB |
| Desktop (GNOME) | 4GB | 2GB |
| Desktop (Sway) | 2GB | 1GB |
| Database Server | 4GB | 2GB |
| Container Host | 4GB | 2GB |
| Build Server | 8GB | 4GB |

### CPU Allocation

| Use Case | Recommended Cores | Minimal Cores |
|----------|------------------|---------------|
| Minimal/Testing | 1 | 1 |
| Development | 4 | 2 |
| Desktop | 4 | 2 |
| Server | 2-4 | 1 |
| Build Server | 8 | 4 |

### Disk Allocation

| Use Case | Recommended Size | Minimal Size |
|----------|-----------------|--------------|
| Minimal | 10GB | 8GB |
| Development | 40GB | 20GB |
| Desktop | 40GB | 30GB |
| Server | 30GB | 20GB |
| Database | 50GB+ | 30GB |

## Advanced Configuration

### Shared Folders

**Host to Guest:**
```nix
virtualisation.sharedDirectories = {
  home = {
    source = "$HOME/vm-projects";
    target = "/home/nixos/projects";
  };
  data = {
    source = "/data";
    target = "/mnt/data";
    readOnly = true;
  };
};
```

### USB Passthrough

```nix
virtualisation.qemu.options = [
  "-usb"
  "-device usb-host,vendorid=0x1234,productid=0x5678"
];
```

### GPU Passthrough (Basic)

```nix
virtualisation.qemu.options = [
  "-vga virtio"
  "-display gtk,gl=on"
];
```

### Nested Virtualization

```nix
# Enable nested KVM
boot.kernelModules = [ "kvm-intel" "kvm-amd" ];
boot.extraModprobeConfig = ''
  options kvm_intel nested=1
  options kvm_amd nested=1
'';
```

### Multiple Network Interfaces

```nix
networking = {
  interfaces = {
    eth0 = {
      useDHCP = true;
    };
    eth1 = {
      ipv4.addresses = [{
        address = "192.168.100.10";
        prefixLength = 24;
      }];
    };
  };
};
```

### Custom Kernel Parameters

```nix
boot.kernelParams = [
  "console=ttyS0"
  "nomodeset"
  "vga=791"
];
```

## Common VM Patterns

### Pattern 1: Quick Test VM

```nix
# Minimal VM for quick testing
{ pkgs, ... }: {
  imports = [ <nixpkgs/nixos/modules/virtualisation/qemu-vm.nix> ];

  virtualisation = {
    memorySize = 2048;
    cores = 2;
    graphics = false;
  };

  users.users.test = {
    isNormalUser = true;
    initialPassword = "test";
    extraGroups = [ "wheel" ];
  };

  security.sudo.wheelNeedsPassword = false;

  system.stateVersion = "24.11";
}
```

### Pattern 2: Development VM

```nix
# Full development environment
{ pkgs, ... }: {
  imports = [ <nixpkgs/nixos/modules/virtualisation/qemu-vm.nix> ];

  virtualisation = {
    memorySize = 8192;
    cores = 4;
    diskSize = 40960;
    forwardPorts = [
      { from = "host"; host.port = 8080; guest.port = 3000; }
    ];
  };

  environment.systemPackages = with pkgs; [
    vim neovim git tmux
    gcc gnumake cmake
    python3 nodejs rustc cargo
    docker-compose kubectl
  ];

  virtualisation.docker.enable = true;

  users.users.dev = {
    isNormalUser = true;
    extraGroups = [ "wheel" "docker" ];
  };

  system.stateVersion = "24.11";
}
```

### Pattern 3: Web Server VM

```nix
# Web application server
{ pkgs, ... }: {
  imports = [ <nixpkgs/nixos/modules/virtualisation/qemu-vm.nix> ];

  virtualisation = {
    memorySize = 2048;
    cores = 2;
    graphics = false;
    forwardPorts = [
      { from = "host"; host.port = 8080; guest.port = 80; }
      { from = "host"; host.port = 8443; guest.port = 443; }
    ];
  };

  services.nginx = {
    enable = true;
    virtualHosts."localhost" = {
      root = "/var/www";
      locations."/" = {
        index = "index.html";
      };
    };
  };

  services.postgresql = {
    enable = true;
    package = pkgs.postgresql_15;
  };

  networking.firewall.allowedTCPPorts = [ 80 443 22 ];

  system.stateVersion = "24.11";
}
```

### Pattern 4: CI/CD Runner VM

```nix
# Dedicated CI/CD runner
{ pkgs, ... }: {
  imports = [ <nixpkgs/nixos/modules/virtualisation/qemu-vm.nix> ];

  virtualisation = {
    memorySize = 4096;
    cores = 4;
    diskSize = 51200;
  };

  environment.systemPackages = with pkgs; [
    git nix docker
    github-runner gitlab-runner
  ];

  virtualisation.docker.enable = true;

  # Configure your CI runner
  services.github-runner = {
    enable = true;
    url = "https://github.com/your/repo";
    tokenFile = "/run/secrets/github-token";
  };

  system.stateVersion = "24.11";
}
```

## Integration with Flakes

### Flake-based VM Configuration

```nix
# flake.nix
{
  description = "NixOS VM configurations";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.11";
  };

  outputs = { self, nixpkgs }: {
    nixosConfigurations = {
      my-vm = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          "${nixpkgs}/nixos/modules/virtualisation/qemu-vm.nix"
          ./vm-configuration.nix
        ];
      };

      dev-vm = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          "${nixpkgs}/nixos/modules/virtualisation/qemu-vm.nix"
          ./vms/dev/configuration.nix
        ];
      };
    };
  };
}
```

**Build and run:**
```bash
# Build VM
nix build .#nixosConfigurations.my-vm.config.system.build.vm

# Run VM
./result/bin/run-my-vm-vm
```

## Validation & Testing

### 1. Configuration Syntax Check

```bash
# Validate configuration
nix-instantiate --parse vm-configuration.nix

# Or with flakes
nix flake check
```

### 2. Build Test

```bash
# Test build without running
nix-build '<nixpkgs/nixos>' -A vm \
  -I nixos-config=./vm-configuration.nix \
  --dry-run
```

### 3. VM Health Check

```bash
# Inside VM, verify:
systemctl status          # System health
journalctl -xe           # Recent logs
free -h                  # Memory usage
nproc                    # CPU cores
df -h                    # Disk usage
ip addr                  # Network config
```

## Troubleshooting

### Issue 1: VM Won't Start

**Check:**
- KVM available: `lsmod | grep kvm`
- QEMU installed: `which qemu-system-x86_64`
- Enough memory: `free -h`

**Solution:**
```bash
# Disable KVM if not available
virtualisation.qemu.options = [ "-no-kvm" ];

# Or reduce memory requirements
virtualisation.memorySize = 1024;
```

### Issue 2: Network Not Working

**Check:**
- Port forwarding rules
- Firewall settings
- Network configuration

**Solution:**
```nix
# Ensure basic networking
networking.useDHCP = true;
networking.firewall.enable = false;  # Temporarily for testing
```

### Issue 3: Slow Performance

**Solutions:**
```nix
# Enable KVM acceleration
virtualisation.qemu.options = [ "-enable-kvm" ];

# Use virtio drivers
virtualisation.qemu.networkingOptions = [
  "-net nic,model=virtio"
];

# Allocate more resources
virtualisation.memorySize = 4096;
virtualisation.cores = 4;
```

### Issue 4: Disk State Issues

**Solution:**
```bash
# Delete old disk state
rm nixos.qcow2

# Rebuild fresh
./result/bin/run-nixos-vm
```

### Issue 5: Can't SSH to VM

**Check:**
```nix
# Ensure SSH enabled
services.openssh.enable = true;

# Port forwarding configured
virtualisation.forwardPorts = [
  { from = "host"; host.port = 2222; guest.port = 22; }
];

# Firewall allows SSH
networking.firewall.allowedTCPPorts = [ 22 ];
```

**Connect:**
```bash
ssh -p 2222 nixos@localhost
```

## Best Practices

### DO ✅

1. **Use declarative configuration** - Everything in Nix files
2. **Version your configurations** - Track in git
3. **Document VM purpose** - Clear comments
4. **Allocate appropriate resources** - Don't over-provision
5. **Enable KVM** - For better performance
6. **Use port forwarding** - Instead of bridged network
7. **Regular backups** - Export important data
8. **Clean disk state** - Remove nixos.qcow2 when testing
9. **Use shared folders** - For easy file transfer
10. **Set firewall rules** - Secure your VM

### DON'T ❌

1. **Over-allocate resources** - Leave some for host
2. **Use bridged network** - Unless necessary (security risk)
3. **Run as root** - Use regular user with sudo
4. **Forget firewall** - Always configure properly
5. **Hard-code passwords** - Use initialPassword only for testing
6. **Skip state version** - Always set system.stateVersion
7. **Ignore backups** - VM disk can corrupt
8. **Mix configurations** - One purpose per VM

## Performance Tips

### Optimization Checklist

- [ ] KVM acceleration enabled
- [ ] Virtio drivers used (network, disk, graphics)
- [ ] Appropriate resource allocation
- [ ] Host has enough free resources
- [ ] Using SSD for VM disk storage
- [ ] Minimal desktop environment (if needed)
- [ ] Disabled unnecessary services
- [ ] Using latest NixOS version

### Resource Monitoring

**Inside VM:**
```bash
# Monitor resources
htop                    # Interactive process viewer
vmstat 1               # Virtual memory stats
iostat 1               # I/O statistics
iftop                  # Network bandwidth
```

**From Host:**
```bash
# Monitor QEMU process
ps aux | grep qemu
top -p $(pgrep qemu)
```

## Success Checklist

- [ ] VM configuration created
- [ ] Resources allocated (memory, CPU)
- [ ] Network configured (ports forwarded)
- [ ] Services enabled and configured
- [ ] Required packages installed
- [ ] User accounts created
- [ ] SSH access working (if needed)
- [ ] Firewall configured
- [ ] VM builds successfully
- [ ] VM runs without errors
- [ ] Services accessible
- [ ] Documentation provided

## Speed Optimization

**Typical workflow:**
- Gather requirements: 2-3 minutes
- Create configuration: 2-3 minutes
- Add services/packages: 1-2 minutes
- Build VM: 5-10 minutes (first time)
- Test and validate: 2-3 minutes

**Total Time**: ~15-20 minutes for complete VM setup

## Quick Examples

### Example 1: Minimal Server VM

```bash
/new_vm
Purpose: testing server
Memory: 1GB
CPU: 1
Network: internal only
Services: SSH only
```

### Example 2: Development VM

```bash
/new_vm
Purpose: development
Memory: 8GB
CPU: 4
Network: external (port 3000)
Services: Docker, PostgreSQL
Packages: git, vim, nodejs, python
```

### Example 3: Web Server VM

```bash
/new_vm
Purpose: web server
Memory: 2GB
CPU: 2
Network: external (ports 80, 443)
Services: Nginx, PostgreSQL
```

Ready to create your NixOS VM? Just tell me what you need! 🚀
