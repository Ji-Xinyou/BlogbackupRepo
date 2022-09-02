---
title: about
date: 2022-05-18 01:11:45
---

(update on: 08/30/2022)
### About me
I am Ji-Xinyou(Jerry), a senior student majoring Computer Science [at] Shanghai Jiao Tong University. Currently, my interests mostly lie in system programming, and I'm working on oci compatible runtime (kata containers).

To check my cv - **[View CV](./cv_en.pdf)**.

### Internship and Open Source Contribution
#### Alibaba Cloud OS Group - R&D Intern (June 2022 - Now)
- Working on Safe Sandbox Container, which runs light-weight VMs just like containers, but with a second layer of defense by hardware virtualization.

#### Kata Containers
- Responsible for kata containters 3.0 architecture development. Building extensible framework, refactoring the codes and implementing runtime features including multiple
network endpoints support, static resource management, core scheduling, cpu/memory hotplug, sandbox/container sizing.

### Academic Performance
**Major GPA**: 93.90/100, **RANK**: 2/118

**OCW**: I have finished all the labs and lectures in following OCWs
- 6.824, 6.828 @MIT
    - kernel/user utilities in xv6 (COW fork, lazy page allocation, mmap etc....)
    - a sharded kv store service on Raft consensus algorithm

- cs231n, cs144 @Stanford
    - NN w/ pure numpy, saliency map, Neural Style Transfer, etc....
    - A complete TCP/IP stack with Ethernet Inferface(support ARP), usable in real world network.

### Personal Projects:

#### System, utility-related
- **J(i)OS**
  - a toy operating system running on baremedal, with self-written bootloader and kernel
  - currently only able to handle timer interrupts and keyboard interrupts (with uart)
- **Rum**
  - a Vim-like text editor in pure Rust

#### DL-related
- **FL-NonIID emulation for benchmark and optimizing algorithms**
  - Giving benchmark for current SOTA FL algorithms on different skewing scheme, including quantity skew, feature skew and label skew
  - Proposed an easy way to help FedBN, which is one of the SOTA algorithms, overcome the unavailability on label skew
- **Wasserstein GAN for image colorization**
  - A project using Wasserstein GAN to colorize gray-scale images, easier to train and relatively good performance
  - Could be trained with 1/15 size of imagenet, but have relatively close performance comparing with most algorithms

