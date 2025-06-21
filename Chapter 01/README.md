
# Getting Familiar with the Ubuntu Ecosystem

## Introduction

In this chapter, we will be introduced to Linux and Ubuntu. We'll start with the history of Linux, its evolution, and then dive into Ubuntu — including its releases, versions, and how it differs from other Linux distributions.

### Objectives:
- Learn how Linux started.
- Understand the Linux stack and how it works.
- Recognize the differences between Linux distributions and how Ubuntu is versioned and released.

---

## Linux History

Linux was created by Linus Torvalds, a computer science student at the University of Helsinki, in 1991 at the age of 21. Initially called *Freax*, the project was later renamed to *Linux* by an administrator at FUNET, where it was first hosted.

- **Tux**: The penguin mascot was chosen after Linus was allegedly bitten by a small penguin during a visit to the National Zoo and Aquarium in Canberra.

You can follow the Linux kernel's development on GitHub: [Linux on GitHub](https://github.com/torvalds/linux)

---

## The Linux Stack

The Linux stack is composed of the following components:

- **Kernel**: The Linux kernel is the core component, handling hardware interactions and system resources.
- **Operating System**: Distributions like Ubuntu and Red Hat are built on top of the Linux kernel.
- **Window Manager/Desktop Environment**: For graphical desktop experiences, window managers like **i3** or full desktop environments like **GNOME** are used.

---

## Linux Usage and Statistics

Linux is widely adopted across various industries:

- **Supercomputers**: Linux powers nearly all of the world’s top supercomputers, including those used by NASA.
- **IoT Devices**: It is the dominant OS in the IoT space.
- **Cloud**: Linux runs approximately 90% of cloud systems, including Kubernetes clusters.
- **Web Servers**: Around 96.3% of the top 1 million web servers run on Linux.
- **Popular Linux Distros**: 
  - Ubuntu: 32.8%  
  - Debian: 14.4%  
  - CentOS: 10.8%

---

## Ubuntu History

Ubuntu is based on Debian and is developed by Canonical Ltd. The first release, Ubuntu 4.10 ("Warty Warthog"), was launched in October 2004. Ubuntu was created to be user-friendly and accessible, and it's available in multiple editions:

- **Ubuntu Desktop**: For personal and office computers with a graphical interface.
- **Ubuntu Server**: Optimized for cloud and physical server environments.
- **Ubuntu Core**: A minimal version designed for IoT and embedded systems.

---

## Ubuntu Releases

Ubuntu releases occur twice a year: in **April (xx.04)** and **October (xx.10)**. Each release is identified by the year and month. **LTS (Long-Term Support)** versions are released every two years and are supported for up to 10 years.

- Learn more about Ubuntu's release cycle: [Ubuntu Release Cycle](https://ubuntu.com/about/release-cycle)

---

## Ubuntu Versions and Variants

Several operating systems are based on Ubuntu, tailored for specific use cases:

1. **Linux Mint**: A user-friendly Ubuntu-based OS that often uses the Cinnamon, MATE, or Xfce desktop environments. Great for newcomers.  
   [Linux Mint Website](https://linuxmint.com/)
2. **Pop!_OS**: Developed by System76, it emphasizes productivity and streamlined workflows.  
   [Pop!_OS Website](https://pop.system76.com/)
3. **LXLE**: A lightweight distribution based on Ubuntu/Lubuntu, aimed at reviving older hardware.  
   [LXLE Website](https://lxle.net/)

---

## Other Popular Linux Distributions

Beyond Ubuntu, there are many other Linux distributions, each serving different purposes:

- **Debian**: The upstream distribution Ubuntu is based on.
- **Red Hat Enterprise Linux (RHEL)**: A stable and enterprise-grade distro used in corporations.
- **Fedora**: A cutting-edge distro often used for testing new Red Hat technologies.
- **Alpine Linux**: A security-oriented, lightweight distro popular for containerized environments.  
  [Alpine Linux Website](https://www.alpinelinux.org/)
- **Flatcar & CoreOS**: Distributions optimized for running containers and Kubernetes workloads.
- **AWS Bottlerocket**: An Amazon-maintained distro built specifically for hosting containers.
- **Talos Linux**: A minimal OS for Kubernetes, with no SSH and immutable infrastructure design.

Explore more:
- [Fedora CoreOS](https://getfedora.org/en/coreos?stream=stable)
- [Flatcar Linux](https://flatcar-linux.org/)
- [Talos Linux](https://www.talos.dev/)
- [AWS Bottlerocket](https://aws.amazon.com/bottlerocket/)

---

## Managing Packages in Linux

Different Linux distributions use different package managers. Here’s how to install the Apache web server across several distros:

1. **Ubuntu/Debian** (uses `apt`):
   ```bash
   sudo apt-get install apache2
   ```

2. **Red Hat/CentOS** (uses `yum` or `dnf`):
   ```bash
   sudo yum install httpd
   # or with dnf:
   sudo dnf install httpd
   ```

3. **Alpine** (uses `apk`):
   ```bash
   sudo apk add apache2
   ```

Although the software may be the same, the commands and package managers differ.

---

## Create GitHub and Blogger Accounts

To track and share your progress, create accounts on the following platforms:

1. **GitHub**: Host your code, configuration files, and version-controlled projects.  
   [GitHub](https://github.com)

2. **Blogger**: A free platform to blog about your learning journey and experiences.  
   [Blogger](https://www.blogger.com)

You can also clone this book’s GitHub repository for code examples:
- [Book GitHub Repo](https://github.com/bpbpublications/Ubuntu-Linux-in-30-days)

---

## Conclusion

What started as a personal project by Linus Torvalds has grown into a global phenomenon. Linux is the backbone of modern computing, and Ubuntu is one of its most accessible and popular distributions. This book will guide you through using Ubuntu effectively for both personal and professional projects.

In the next chapter, we’ll walk through how to **install Ubuntu** and get started with your system.

---

## References
- [Fedora CoreOS](https://getfedora.org/en/coreos?stream=stable)
- [Flatcar Linux](https://flatcar-linux.org/)
- [Talos Linux](https://www.talos.dev/)
- [AWS Bottlerocket](https://aws.amazon.com/bottlerocket/)
```

