### Reviving Xeon Phi 5110P for AI & HPC on Modern Linux Distributions
![image](https://github.com/user-attachments/assets/86a4ec63-80c9-4bd2-a853-65fa5d1c69e6)
In Lima, Peru's basic education sector, there is a growing demand for cost-effective AI solutions to support personalized learning, automated assessments, and intelligent tutoring systems. However, deploying AI models, such as 7B and 14B parameter models, requires substantial computational resources, often beyond the financial reach of institutions with limited budgets.

The Intel Xeon Phi 5110P offers a potential low-cost alternative for AI workloads due to its high parallel computing capabilities. However, its adoption on modern operating systems, such as AlmaLinux 8.10 (AlmaLinux, 2025), is hindered by the lack of readily available drivers and utilities. Since Intel has discontinued support for the Xeon Phi architecture, newer Linux distributions no longer include the necessary software components to fully utilize these coprocessors.

This challenge limits educators and developers from repurposing existing Xeon Phi hardware for AI inference and training. Addressing this issue requires developing or adapting driver support and utilities to ensure full compatibility with AlmaLinux 8.10. This effort would enable low-budget institutions to leverage older hardware for AI-based educational tools, fostering technological inclusion and innovation in Lima’s education sector.

This research explores the feasibility of recompiling Xeon Phi 5110P drivers on AlmaLinux 8.10 to support AI workloads in Lima’s educational sector. AlmaLinux 8.10 is chosen for its long-term support until 2029, binary compatibility with RHEL, and stability, making it a practical solution for overcoming the absence of official drivers. Ensuring compatibility with modern AI frameworks would provide low-cost, scalable computing resources, allowing educational institutions to maximize their existing hardware investments for AI-powered learning solutions.

**Literature Review**

Keijser (2024) provides a comprehensive guide on implementing the Manycore Platform Software Stack (MPSS) for the Xeon Phi on CentOS 8, addressing compatibility and configuration challenges in modern operating systems. This study builds upon Keijser’s work by adapting it for AlmaLinux 8.1, leveraging its binary compatibility with RHEL and extended support. The methodology outlined in Keijser (2024) has served as a critical reference for adapting MPSS software in AlmaLinux, ensuring a functional solution for educational environments in the U.S.

**Installing the mic.ko Module on AlmaLinux 8.10** 

First, you may choose to use the precompiled module provided on GitHub. If you prefer or require a custom build, ensure to follow the instructions for compiling the module yourself, particularly if your kernel version differs. After successfully building the module, proceed with installing the newly compiled kernel module.

   **Step 1: Installing the Compiled Module**  

To install the precompiled module, use the following command:  

```rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/releases/download/v0.1.0-alpha/mpss-modules-4.18.0-553.40.1.el8_10.x86_64-3.8.6-8.x86_64.rpm```

This registers the `mic.ko` module with the system. Note: If your kernel version is different from the one mentioned here, you'll need to follow the steps in the next section to build the kernel module from source.

   **Step 2: Creating a Symbolic Link for mic.ko**  

Navigate to the kernel module directory and create a symbolic link to the `mic.ko` module:  

```
cd /lib/modules/4.18.0-553.40.1.el8_10.x86_64/weak-updates/
ln -s /lib/modules/4.18.0-553.40.1.el8_10.x86_64/extra/mic.ko ./mic.ko
```

This ensures the system recognizes the module in the correct location.  

   **Step 3: Loading and Verifying the Module**  

Finally, load the `mic.ko` module into the kernel using:  

```modprobe mic```

To confirm that the module is successfully loaded, check with:  

```lsmod | grep mic```

If the module is properly installed and loaded, you should see `mic` listed in the output.

   **Step 4: Installing Required Libraries and Daemon**

Install the required dependencies for running the MPSS daemon:

```
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif0-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif-dev-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif-doc-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh mpss-daemon-3.8.6-4.el8.x86_64.rpm
rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/blob/main/mpss-daemon-3.8.6-4.el8.x86_64.rpm
```

**Building the mic.ko Module on AlmaLinux 8.10**  

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

   **Conclusion**  

By following these steps, you have successfully compiled, installed, and loaded the `mic.ko` kernel module on AlmaLinux 8.10. This process enables the continued use of Xeon Phi 5110P hardware despite the discontinuation of official support. With this module in place, you can now explore AI and HPC applications using this hardware.

**References**

AlmaLinux. (2025). Get AlmaLinux. AlmaLinux Foundation. Recuperado el 26 de febrero de 2025, de https://almalinux.org/get-almalinux/

Keijser, J. J. (2023). MPSS for Xeon Phi 5110P KNC drivers and utilities (v3.8.6). GitHub. https://github.com/jjkeijser/mpss

Keijser, J. J. (2022). Xeon Phi MPSS on CentOS 8. Retrieved from https://jjkeijser.github.io/mpss/xeon-phi-mpss-centos8.html

xdsopl. (2015). mpss-modules [GitHub repository]. GitHub. Retrieved February 26, 2025, from https://github.com/xdsopl/mpss-modules
