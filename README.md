### Reviving Xeon Phi 5110P for AI & HPC on Modern Linux Distributions

In Lima, Peru's basic education sector, there is a growing demand for cost-effective AI solutions to support personalized learning, automated assessments, and intelligent tutoring systems. However, deploying AI models, such as 7B and 14B parameter models, requires substantial computational resources, often beyond the financial reach of institutions with limited budgets.

The Intel Xeon Phi 5110P offers a potential low-cost alternative for AI workloads due to its high parallel computing capabilities. However, its adoption on modern operating systems, such as AlmaLinux 8.1, is hindered by the lack of readily available drivers and utilities. Since Intel has discontinued support for the Xeon Phi architecture, newer Linux distributions no longer include the necessary software components to fully utilize these coprocessors.

This challenge limits educators and developers from repurposing existing Xeon Phi hardware for AI inference and training. Addressing this issue requires developing or adapting driver support and utilities to ensure full compatibility with AlmaLinux 8.1. This effort would enable low-budget institutions to leverage older hardware for AI-based educational tools, fostering technological inclusion and innovation in Lima’s education sector.

This research explores the feasibility of recompiling Xeon Phi 5110P drivers on AlmaLinux 8.1 to support AI workloads in Lima’s educational sector. AlmaLinux 8.1 is chosen for its long-term support until 2029, binary compatibility with RHEL, and stability, making it a practical solution for overcoming the absence of official drivers. Ensuring compatibility with modern AI frameworks would provide low-cost, scalable computing resources, allowing educational institutions to maximize their existing hardware investments for AI-powered learning solutions.

**Literature Review**

Keijser (2024) provides a comprehensive guide on implementing the Manycore Platform Software Stack (MPSS) for the Xeon Phi on CentOS 8, addressing compatibility and configuration challenges in modern operating systems. This study builds upon Keijser’s work by adapting it for AlmaLinux 8.1, leveraging its binary compatibility with RHEL and extended support. The methodology outlined in Keijser (2024) has served as a critical reference for adapting MPSS software in AlmaLinux, ensuring a functional solution for educational environments in the U.S.

**References**

Keijser, J. J. (2024). Xeon Phi MPSS on CentOS 8. Retrieved from https://jjkeijser.github.io/mpss/xeon-phi-mpss-centos8.html
