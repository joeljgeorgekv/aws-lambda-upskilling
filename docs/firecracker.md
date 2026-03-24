# Firecracker

## Overview

Firecracker is an open-source virtual machine monitor (VMM) that uses the Linux Kernel-based Virtual Machine (KVM) to create and manage lightweight virtual machines called **microVMs**. It was developed at **Amazon Web Services** to improve the customer experience of services like **AWS Lambda** and **AWS Fargate**.

## Built with Rust

Firecracker is **written in Rust**, a modern programming language specifically chosen for its safety guarantees:

- **Thread safety** - Rust's ownership model prevents data races at compile time
- **Memory safety** - Prevents many types of buffer overrun errors that lead to security vulnerabilities
- **Zero-cost abstractions** - High-level safety without runtime overhead
- **Systems programming** - Ideal for VMM development with direct hardware interaction

Firecracker was built starting from **[crosvm](https://chromium.googlesource.com/chromiumos/platform/crosvm/)** (Chrome OS's VMM), reimagined with a minimalist device model for serverless workloads.

## Key Benefits

### Enhanced Security
- Firecracker provides workload isolation through virtualization barriers
- Each microVM runs with a minimalist design that excludes unnecessary devices
- **Jailer** companion program provides a second line of defense using Linux user-space security barriers
- Reduced attack surface compared to traditional VMs

### Speed & Efficiency
- **Fast startup times** - microVMs start in **as little as 125 milliseconds** (and even faster in later versions)
- **Low memory overhead** - consumes only **~5 MiB of memory per microVM**
- **High density** - run **thousands of secure microVMs** on a single machine
- **Resource efficiency** comparable to containers with VM-level isolation

### Built-in Resource Management
- **RESTful API** for creating and configuring microVMs
- Built-in **rate limiters** for granular control of network and storage resources
- Configurable vCPU allocation
- Flexible rate limiting with burst support
- Metadata service for secure configuration sharing between host and guest

## Why Firecracker is Faster & More Lightweight

### 1. Minimalist Design Philosophy
Unlike traditional VMMs designed for general-purpose use, Firecracker was purpose-built for **serverless workloads**. It excludes unnecessary devices and guest functionality:

| Firecracker Provides | Traditional VMMs (QEMU) |
|---------------------|------------------------|
| Network device (virtio-net) | Full PCI bus emulation |
| Block I/O device (virtio-blk) | USB controllers |
| Programmable Interval Timer | Audio devices |
| KVM clock | Graphics cards |
| Serial console | Multiple disk controllers |
| Partial keyboard (reset only) | Legacy BIOS services |

### 2. Security-First Architecture
Firecracker's security model reduces overhead while maintaining isolation:

- **Process Jail** - Firecracker processes are isolated using **cgroups** and **seccomp BPF**, with access to a small, tightly controlled list of system calls
- **Static Linking** - The firecracker binary is statically linked, ensuring a clean host environment
- **Minimal Attack Surface** - The simple guest model exposes fewer vectors for exploitation

### 3. Language-Level Safety
Being written in **Rust** provides performance benefits:
- No garbage collection pauses
- Compile-time memory management eliminates runtime overhead
- Thread safety without locks in many cases
- Prevention of entire classes of bugs at compile time

### 4. Optimized for Short-Lived Workloads
Traditional VMs assume long-running processes. Firecracker optimizes for **transient, short-lived workloads**:
- No full hardware emulation overhead
- Fast path for common operations
- Minimal initialization code paths

```
┌─────────────────────────────────────────────────────────┐
│                    Host OS (Linux)                       │
│  ┌───────────────────────────────────────────────────┐  │
│  │           Firecracker VMM (User Space)            │  │
│  │         ┌─────────────┐    ┌─────────────┐          │  │
│  │         │  microVM 1  │    │  microVM 2  │  ...     │  │
│  │         │  (Guest OS) │    │  (Guest OS) │          │  │
│  │         └─────────────┘    └─────────────┘          │  │
│  └───────────────────────────────────────────────────┘  │
│                      KVM (Kernel)                       │
└─────────────────────────────────────────────────────────┘
```

Firecracker runs in user space and uses KVM to create microVMs. The minimalist design excludes unnecessary devices and guest functionality, reducing memory footprint and improving security.

## Firecracker vs Traditional VMMs

| Feature | Firecracker | QEMU (Traditional VMM) |
|---------|-------------|------------------------|
| **Purpose** | Serverless/container workloads | General purpose |
| **Startup Time** | **~125 ms** | Seconds (often 10-30s) |
| **Memory Footprint** | **~5 MiB** | Hundreds of MB |
| **Attack Surface** | Minimal (6 devices only) | Larger (full hardware emulation) |
| **Process Isolation** | cgroups + seccomp BPF | Limited |
| **Language** | **Rust** (memory safe) | C (vulnerable to overflows) |
| **Device Support** | Minimal (virtio-only) | Broad (PCI, USB, VGA, etc.) |

## AWS Services Using Firecracker

- **AWS Lambda** - Serverless compute service
- **AWS Fargate** - Serverless compute for containers

### Scale at AWS

Firecracker powers massive scale at AWS:

- **AWS Lambda** processes **trillions of executions** for hundreds of thousands of active customers every month
- **AWS Fargate** runs **tens of millions of containers** for AWS customers every week

### Evolution at AWS

Firecracker was born from AWS's own journey with serverless:

1. **2014 (Lambda Launch)** - Initially used dedicated EC2 instances per customer for isolation, meeting security goals but with backend efficiency tradeoffs
2. **Growing Adoption** - As customers increasingly adopted serverless, AWS revisited the efficiency problem
3. **Invent and Simplify** - AWS asked: "What would a virtual machine look like if designed for today's world of containers and functions?"
4. **Firecracker** - The answer: a purpose-built VMM optimized for short-lived, secure, multi-tenant workloads

Firecracker enables these services to provide:
- Rapid function/container initialization (125ms startup)
- Secure multi-tenant isolation
- Efficient resource utilization at massive scale

## Adopters & Integrations

Firecracker is used by various platforms (alphabetical order):

- **appfleet** - Edge hosting platform
- **containerd** (via firecracker-containerd) - Container runtime
- **Fly.io** - Application hosting platform
- **Kata Containers** - Secure container runtime
- **Koyeb** - Serverless platform
- **Northflank** - Cloud native deployment
- **OpenNebula** - Cloud management platform
- **Qovery** - Developer platform
- **UniK** - Unikernel compiler
- **webapp.io** - CI/CD platform
- **microvm.nix** - Nix-based microVMs

## Supported Platforms

Firecracker runs on **64-bit Intel, AMD, and Arm CPUs** with hardware virtualization support (KVM).

### Supported Guest Operating Systems
- Linux
- OSv (unikernel)

## Getting Started

Firecracker is open-source under the **Apache License 2.0**.

### Resources

- **GitHub**: [github.com/firecracker-microvm/firecracker](https://github.com/firecracker-microvm/firecracker)
- **Slack Community**: [Join the Firecracker Slack](https://join.slack.com/t/firecracker-microvm/shared_invite)
- **Documentation**: Available on the [Firecracker website](https://firecracker-microvm.github.io/)
- **Roadmap**: [GitHub Project](https://github.com/orgs/firecracker-microvm/projects/42)

### Key Related Projects

- **[crosvm](https://chromium.googlesource.com/chromiumos/platform/crosvm/)** - Chrome OS VMM (Firecracker is based on this)
- **[Rust-vmm](https://github.com/rust-vmm)** - Rust virtual machine monitor community

## Further Reading

- [AWS Blog: Firecracker – Lightweight Virtualization for Serverless Computing](https://aws.amazon.com/blogs/aws/firecracker-lightweight-virtualization-for-serverless-computing)
- [AWS Open Source Blog: Announcing the Firecracker Open Source Technology](https://aws.amazon.com/blogs/opensource/firecracker-open-source-secure-fast-microvm-serverless/)
