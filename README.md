
**Contributions by Each Team Member**

**1.For each member in your team, provide 1 paragraph detailing what parts of the lab that member implemented / researched.**

- **Revanth:** I took the lead on setting up the outer VM environment, including cloning the Linux kernel repository and configuring the
 kernel build process. I identified the correct KVM source code files and implemented the logic for the VM exit counters in the vmx.c file.
 Additionally, I ensured that the custom kernel was compiled and installed successfully in the outer VM**.
Researched VM exit types and their relevance to different operations in a virtualized environment.

- **Akshara:** I focused on testing and validating the implementation. I configured/created the inner VM, verified its compatibility with the custom kernel,
 and analyzed the printk logs to collect VM exit statistics.
 Tested the modified kernel by performing a variety of operations in the inner VM to generate different types of VM exits.
 I also worked on handling troubleshooting issues during the setup and testing phases.Researched the CPUID instruction to understand.



**2.Describe in detail the steps you used to complete the assignment.
   Consider your reader to be someone skilled in software development but otherwise unfamiliar with the assignment.

Revanth's Contributions:**

I primarily focused on setting up the environment for the assignment and implementing the core logic for tracking VM exit statistics. 
Below are the detailed steps of the work I contributed:

1. **Setting Up the Outer VM Environment**:
   1.1. Researched how to configure a nested virtualization environment in Google Cloud and ensured that the outer VM was configured with KVM support.
   1.2. Cloned the forked Linux kernel repository into the outer VM and verified its compatibility with the existing setup:

      **git clone https://github.com/revanth2802/linux.git**

      **cd linux**

   1.3. Configured the Linux kernel using the existing kernel configuration from /boot.
       I had to research the correct tools (libelf-dev, ncurses) to ensure the build process worked smoothly:

      **cp /boot/config-$(uname -r) .config**

      **make menuconfig**

   2. Resolved multiple dependency issues during the kernel compilation process (installing flex, bison, and libssl-dev).
	2.1. **KVM Source Code Modifications**:
        2.2. Spent considerable time identifying the correct source files (vmx.c and cpuid.c) in the Linux kernel. 
             Researched the structure of the KVM hypervisor to understand where VM exits were handled.
        2.3. Added the logic to increment per-exit-type counters and a global counter in arch/x86/kvm/vmx/vmx.c:
        2.4. Used atomic\_t to safely update counters in a multi-threaded environment.
        2.5. Implemented logic to print statistics every 10,000 exits using printk.
   3. Modified arch/x86/kvm/cpuid.c to enable reporting of VM exit statistics through the CPUID instruction.
   I 3.1 spent time understanding the CPUID emulation mechanism in KVM and correctly mapped the exit statistics to specific CPUID leaves.
  4. **Kernel Compilation and Installation**:
   4.1. Built the kernel with the modified source code and resolved multiple compilation errors during the process.
   4.2.. Installed the compiled kernel in the outer VM and ensured it booted correctly:

      **sudo make modules\_install**

      **sudo make install**

      **sudo reboot**

   4.3. Conducted preliminary testing in the outer VM to verify the functionality of the new kernel before proceeding to the inner VM setup.
5. **Research and Validation**:
   5.1. Researched VM exit types and their relevance to different operations in a virtualized environment.
 This helped in interpreting the printk logs and validating the accuracy of the implemented counters.
   5.2. Verified that the printk output in dmesg provided meaningful and consistent data.


**Akshara's Contributions:**

I focused on creating, testing, validating, and troubleshooting the implementation. Below is a detailed breakdown of my work:

1. **Inner VM Configuration**:
   1.1. Researched the process of configuring an inner VM using virt-manager and virsh.
 This included enabling nested virtualization in the outer VM and installing the necessary tools (virt-manager).
   1.2. Set up the inner VM with Ubuntu and ensured it was compatible with the custom kernel in the outer VM. Used commands such as:

**sudo virt-install \**

`    `**--name inner-vm1 \**

`    `**--vcpus 2 \**

`    `**--memory 2048 \**

`    `**--disk size=200GB \**

`    `**--cdrom ubuntu-22.04.iso \**

`    `**--os-variant ubuntu22.04**

2. Troubleshot multiple issues with the inner VM setup, including disk configuration errors.
3. **Testing and Validation**:
   3.1. Tested the modified kernel by performing a variety of operations in the inner VM to generate different types of VM exits:
      3.1.1. Booted the inner VM multiple times to measure the number of VM exits during startup.
      3.1.2. Ran CPU- and I/O-intensive workloads (e.g file downloads, disk writes) to trigger different VM exit types.
   3.2. Monitored the printk output in the outer VM to collect and analyze VM exit statistics:

**dmesg | grep "VM Exit Statistics"**

4. **Troubleshooting and Debugging**:
   4.1. Investigated issues with printk logs not displaying as expected by verifying the modifications in the source code. 
     Worked closely with Revanth to refine the logic for counting and printing exit statistics.
   4.2.. Resolved kernel module loading errors by unloading and reloading the kvm and kvm-intel modules:

      **sudo rmmod kvm\_intel**

      **sudo insmod kvm/kvm.ko**

      **sudo insmod kvm/kvm-intel.ko**

5. **Research**:
   5.1. Explored the relationship between VM operations and exit types I/O operations causing IO\_INSTRUCTION exits.
   5.1.. Researched the CPUID instruction to understand its use in reporting hypervisor-specific information.

**3.Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are theremore exits performed during certain VM operations?
 Approximately how many exits does a full VM boot entail?**

The number of VM exits does not increase at a stable rate. From the logs:

- Exits are significantly higher during specific operations such as **MSR\_WRITE** (10,645 times) and **EPT\_MISCONFIG** (2,130 times),
 which typically occur during VM boot and memory-intensive tasks.
- Other types of exits, such as **EXCEPTION\_NMI** (2 times) and **UNKNOWN\_EXIT** (ranging from 2 to a few hundred times), occur far less frequently,
 indicating these events are tied to less common or specific operations.

**Approximate Exits for a Full VM Boot**

The VM boot process, including initialization and other tasks, entailed approx. **49,100 exits**,
 as evident from the high counts of operations like MSR\_WRITE and EPT\_MISCONFIG.
 These operations dominate the logs during the boot phase, contributing to the majority of the exits.

**4.Of the exit types, which are the most frequent? Least?**

**MSR\_WRITE**: This exit type occurred 10,645 times most frequent.

**EPT\_MISCONFIG**: This exit type occurred 2,130 times least.

