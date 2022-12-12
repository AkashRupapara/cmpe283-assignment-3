# cmpe283-assignment-3

The task is carried out on a Google Cloud VM that supports nested virtualization (`—enable-nested-virtualization`).

Configuration:
  - CPU PLatform: Intel Cascade Lake 
  - Machine Type: n2-standard-8

Assignment 2 modifies the behavior of the cpuid instruction for the following cases:
  - CPUID lead node(%eax=0x4ffffffe)
    - Return the high 32 bits of the total time spent processing all exits in %ebx and %ecx. Return values are measured in processor cycles, across all           vCPUs.
  - CPUID leaf node(%eax=0x4fffffff)
    - Return the total number of exits for all types in %eax.
  
<h2> Question 1: Team Details: </h2> 
   - `Done by Myself`
   
<h2> Question 2: Describe in detail the steps used to complete the assignment:</h2>
   - Download kernel from https://www.kernel.org/  and extract source code (Run following commands)
  
```
sudo apt-get install wget
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.0.7.tar.xz
tar xvf linux-6.0.7.tar.xz
```

- Install required packages before intiating installation:
```
sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
```
- Configure kernel
```
cd linux-6.0.7
cp -v /boot/config-$(uname -r) .config
```

- Build the kernel
```
sudo make modules
sudo make
sudo make modules_install
sudo make install
```
- Bootloader is automatically updated because of the make install and now we can reboot the system and check the kernel version
```
sudo reboot
uname -mrs
```
<img width="441" alt="Screen Shot 2022-12-05 at 7 24 33 AM" src="https://user-images.githubusercontent.com/29530515/205539152-91c80941-4f9e-4243-8ce8-7dcf63bb07e2.png">

- Make code changes and overwrite in given location below.(vmx.c and cpuid.c)
```
/linux-6.0.7/arch/x86/kvm/vmx/vmx.c
/linux-6.0.7/arch/x86/kvm/cpuid.c
```

- Build and install modules again
```
sudo make modules && sudo make modules_install
```
- Load newly build modules
```
sudo rmmod kvm_intel
sudo rmmod kvm
sudo modprode kvm
sudo modprobe kvm_intel
```

- For testing the functionality, a nested VM was created in the GCP VM. The steps to create an nested VM are as follows:

  - Download the Ubuntu cloud image.img(QEMU compatible image) file from this [ubuntu cloud images site] in GCP VM(https://cloud-images.ubuntu.com/). 
  ```
  wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
  sudo apt update && sudo apt install qemu-kvm -y
  ```
  
  - Go to the folder where the.img file was downloaded. There is no default password for the virtual machine included with this Ubuntu cloud image. So, follow these instructions (terminal) to modify the password and log in to the virtual machine:
 ```
  sudo apt-get install cloud-image-utils

  cat >user-data <<EOF
    #cloud-config
    password: newpass #new password here
    chpasswd: { expire: False }
    ssh_pwauth: True
    EOF

  cloud-localds user-data.img user-data
 ```
 - Run following command to initiate nested vm. Set configuration : `username: ubuntu` and `password: newpass`
 ```
 sudo qemu-system-x86_64 -enable-kvm -hda bionic-server-cloudimg-amd64.img -drive "file=user-data.img,format=raw" -m 512 -curses -nographic
 ```
 
 - install utility:
 ```
  sudo apt-get update
  sudo apt-get install cpuid
 ```
 
 - IMPORTANT: open two terminals:
    - the GCP VM terminal(T1)
    - the nested VM terminal(logged in)(T2)
 - In T2, write a new test script(filename="test_cpuid.sh") as follows:
 ```
 #!/bin/bash

  for i in {0..76}
  do
      echo "EXIT $i"
      cpuid -l $1 -s $i
  done
 ```
 - Testing the CPUID functionality for `%eax=0x4ffffffd`
    - T2: `sudo bash test_cpuid.sh 0x4ffffffd`
    - T1: `sudo dmesg`
    
    <img width="687" alt="Screen Shot 2022-12-12 at 12 28 04 PM" src="https://user-images.githubusercontent.com/29530515/206981844-d71dfa6e-2625-44a9-b848-e1b88867c50f.png">
<img width="797" alt="Screen Shot 2022-12-12 at 12 30 12 PM" src="https://user-images.githubusercontent.com/29530515/206981880-4f976ec1-626a-44cb-b660-709a866334b8.png">
 <img width="799" alt="Screen Shot 2022-12-12 at 12 30 20 PM" src="https://user-images.githubusercontent.com/29530515/206981897-5e2b0263-e60f-49f6-b36b-3cb6320b7f9f.png">
<img width="801" alt="Screen Shot 2022-12-12 at 12 30 26 PM" src="https://user-images.githubusercontent.com/29530515/206981912-33538ca3-483a-489d-b680-a73d60740497.png">
<img width="802" alt="Screen Shot 2022-12-12 at 12 30 39 PM" src="https://user-images.githubusercontent.com/29530515/206981978-b90164bc-8087-4a89-9d87-041e38a84db0.png">

- Testing the CPUID functionality for `%eax=0x4ffffffc`

  - T2: `sudo bash test_cpuid.sh 0x4ffffffc`
  - T1: `sudo dmesg`
  <img width="709" alt="Screen Shot 2022-12-12 at 12 37 38 PM" src="https://user-images.githubusercontent.com/29530515/206982294-e2f3688f-6554-4cba-807a-0c90ad899ab0.png">
<img width="799" alt="Screen Shot 2022-12-12 at 12 38 43 PM" src="https://user-images.githubusercontent.com/29530515/206982628-1584fb36-c5c6-47ce-a7d5-4051feb46856.png">
<img width="802" alt="Screen Shot 2022-12-12 at 12 38 58 PM" src="https://user-images.githubusercontent.com/29530515/206982641-c36319b4-9f45-478a-a3dd-c37f6757193d.png">
<img width="801" alt="Screen Shot 2022-12-12 at 12 39 07 PM" src="https://user-images.githubusercontent.com/29530515/206982654-630c1a17-7827-48be-b028-1eaa451d58a7.png">
<img width="802" alt="Screen Shot 2022-12-12 at 12 39 15 PM" src="https://user-images.githubusercontent.com/29530515/206982663-1db5c636-6608-43e4-8b78-1776ba322520.png">
<img width="802" alt="Screen Shot 2022-12-12 at 12 39 24 PM" src="https://user-images.githubusercontent.com/29530515/206982672-89e64ff4-2a7c-416d-b965-202fc6c774b5.png">
<img width="801" alt="Screen Shot 2022-12-12 at 12 39 36 PM" src="https://user-images.githubusercontent.com/29530515/206982686-356726e4-1581-44a5-b00a-5fe6dd4e6381.png">
<img width="798" alt="Screen Shot 2022-12-12 at 12 39 47 PM" src="https://user-images.githubusercontent.com/29530515/206982704-bc6b8aca-d842-4287-89ab-93689462f0f8.png">

<h2>Question 3: Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are theremore exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?</h2>

- Since HLT(10) and CPUID(12) exits are called very frequently, the number of exits for these two exits grows steadily.
- Output:
    
<img width="666" alt="Screen Shot 2022-12-12 at 12 45 48 PM" src="https://user-images.githubusercontent.com/29530515/206984024-946c4276-53cf-4d3f-983a-343b3cabce3f.png">
    
<img width="416" alt="Screen Shot 2022-12-12 at 12 44 32 PM" src="https://user-images.githubusercontent.com/29530515/206984040-d5cc979f-10e9-4daf-8bf4-a5606f08119b.png">

<h2>Question 4</h2>

- Following are some of the most frequently called exits:
    - Exit #49 - EPT_MISCONFIG - (called 212807 times)
    - Exit #30 - IO_INSTRUCTION - (called 1447497 times)
    - Exit #28 - CR_ACCESS - (called 500909 times)
- Following are some of the most frequently called exits:
    - Exit #0 - EXCEPTION_NMI - (called 11 times)
    - Exit #54 - WBINVD - (called 6 times)
    - Exit #29 - DR_ACCESS - (called 1 time)     
