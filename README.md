### Reviving Xeon Phi 5110P for AI & HPC on Modern Linux Distributions

In Lima, Peru's basic education sector, there is a growing demand for cost-effective AI solutions to support personalized learning, automated assessments, and intelligent tutoring systems. However, deploying AI models, such as 7B and 14B parameter models, requires substantial computational resources, often beyond the financial reach of institutions with limited budgets.

The Intel Xeon Phi 5110P offers a potential low-cost alternative for AI workloads due to its high parallel computing capabilities. However, its adoption on modern operating systems, such as AlmaLinux 8.1, is hindered by the lack of readily available drivers and utilities. Since Intel has discontinued support for the Xeon Phi architecture, newer Linux distributions no longer include the necessary software components to fully utilize these coprocessors.

This challenge limits educators and developers from repurposing existing Xeon Phi hardware for AI inference and training. Addressing this issue requires developing or adapting driver support and utilities to ensure full compatibility with AlmaLinux 8.1. This effort would enable low-budget institutions to leverage older hardware for AI-based educational tools, fostering technological inclusion and innovation in Lima’s education sector.

This research explores the feasibility of recompiling Xeon Phi 5110P drivers on AlmaLinux 8.1 to support AI workloads in Lima’s educational sector. AlmaLinux 8.1 is chosen for its long-term support until 2029, binary compatibility with RHEL, and stability, making it a practical solution for overcoming the absence of official drivers. Ensuring compatibility with modern AI frameworks would provide low-cost, scalable computing resources, allowing educational institutions to maximize their existing hardware investments for AI-powered learning solutions.

**Literature Review**

Keijser (2024) provides a comprehensive guide on implementing the Manycore Platform Software Stack (MPSS) for the Xeon Phi on CentOS 8, addressing compatibility and configuration challenges in modern operating systems. This study builds upon Keijser’s work by adapting it for AlmaLinux 8.1, leveraging its binary compatibility with RHEL and extended support. The methodology outlined in Keijser (2024) has served as a critical reference for adapting MPSS software in AlmaLinux, ensuring a functional solution for educational environments in the U.S.

**Building and Installing the mic.ko Module on AlmaLinux 8.10**  

The Intel Xeon Phi 5110P requires the `mic.ko` kernel module to function properly, but since official support for the Manycore Platform Software Stack (MPSS) has been discontinued, manually building and installing this module is necessary. This guide walks through the process of compiling and installing the `mic.ko` module on AlmaLinux 8.10.  

  **Step 1: Preparing the System**  

Before beginning, ensure that your system has the necessary development tools installed. Start by installing the required packages:  

```
dnf groupinstall -y "Development Tools"

dnf install -y rpm-build kernel-devel-$(uname -r) kernel-headers-$(uname -r) elfutils-libelf-devel gcc make dnf-utils
```

This installs essential compilers, kernel development files, and utilities required to build kernel modules.  

   **Step 2: Setting Up the RPM Build Environment**  

Next, create the necessary directory structure for building RPM packages:  

```mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}```

This ensures that all necessary folders exist before proceeding with the RPM build process.  

   **Step 3: Installing the Source RPM**  

Now, install the MPSS source RPM package:  

```rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/releases/download/v0.1.0-alpha/mpss-modules-3.8.6-8.src.rpm```

This extracts the source files into the `rpmbuild` directory, which will be used for compiling the module.  

   **Step 4: Editing the SPEC File**  

Navigate to the `SPECS` directory and open the SPEC file for editing:  

```
cd ~/rpmbuild/SPECS
vi mpss-modules-3.8.6-rhel85.spec
```

In this step, verify that the kernel version matches the one currently running on your system. You can check your kernel version with:  

```uname -r```

For example, if the output is `4.18.0-553.40.1.el8_10.x86_64`, update the SPEC file accordingly.  

   **Step 5: Applying Kernel Patch**  

Copy the appropriate kernel patch file to the `SOURCES` directory:  

```cp /root/rpmbuild/SOURCES/mpss-modules-4.18.0-553.37.1.el8_10.x86_64.patch /root/rpmbuild/SOURCES/mpss-modules-4.18.0-553.40.1.el8_10.x86_64.patch```

This ensures that the patch aligns with the kernel version currently installed.  

   **Step 6: Building the RPM Package**  

Once the necessary modifications are in place, build the RPM package with:  

```rpmbuild -bb mpss-modules-3.8.6-rhel85.spec```

This command compiles the kernel module and packages it into an installable RPM.  

   **Step 7: Installing the Compiled Module**  

After a successful build, install the newly compiled kernel module:  

```rpm -ivh mpss-modules-4.18.0-553.40.1.el8_10.x86_64-3.8.6-7.x86_64.rpm```

This registers the `mic.ko` module with the system.  

   **Step 8: Creating a Symbolic Link for mic.ko**  

Navigate to the kernel module directory and create a symbolic link to the `mic.ko` module:  

```
cd /lib/modules/4.18.0-553.40.1.el8_10.x86_64/weak-updates/
ln -s /lib/modules/4.18.0-553.40.1.el8_10.x86_64/extra/mic.ko ./mic.ko
```

This ensures the system recognizes the module in the correct location.  

   **Step 9: Loading and Verifying the Module**  

Finally, load the `mic.ko` module into the kernel using:  

```modprobe mic```

To confirm that the module is successfully loaded, check with:  

```lsmod | grep mic```

If the module is properly installed and loaded, you should see `mic` listed in the output.  

   **Conclusion**  

By following these steps, you have successfully compiled, installed, and loaded the `mic.ko` kernel module on AlmaLinux 8.10. This process enables the continued use of Xeon Phi 5110P hardware despite the discontinuation of official support. With this module in place, you can now explore AI and HPC applications using this hardware.

**References**

Keijser, J. J. (2024). Xeon Phi MPSS on CentOS 8. Retrieved from https://jjkeijser.github.io/mpss/xeon-phi-mpss-centos8.html
