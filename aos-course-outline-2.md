### 课程组织与基本目的

操作系统是计算机系统中负责管理各种软硬件资源的核心系统软件，掌握操作系统的经典研究成果和当前研究热点是研究型大学计算机专业研究生的基本要求。

本课程从计算机系统的视角进行内容组织与调整，以教学操作系统uCore/rCore和RISC-V CPU为实验环境，讲授操作系统的核心原理和创新研究，介绍当前操作系统研究热点和论文，帮助学生了解和掌握大型复杂系统的问题分析方法和核心设计思路，并为学生充分利用操作系统功能进行系统研究和开发打下扎实的基础。


### 评分
本课程没有考试。评分基于学生的实验情况和论文阅读情况
- 实验部分： 基于u/rCore的实验一~五：40% 
- 论文阅读部分： 4篇选定论文的阅读分析报告：60%

### 教学内容

下面的每一讲是两个课时

- Lec 1 Advanced OS Overview 

    1. Course Overview 
    2. Course Scheduling 
    3. Rethink OS Components
    4. Tendency of OS -- Performance
    5. Tendency of OS -- Reliability
    6. Tendency of OS -- Correctness
    7. Summary
   
- Lec 2 OS Architecture

    1. History of OS Architecture -- THE
    2. Monolithic Kernel -- UNIX
    3. Micro Kernel -- L4
    4. ExoKernel 
    5. Extensible Kernel
    6. Summary

- Lec 3+4 System Virtualization Overview

    1. Introduction
    2. Traditional Virtualization Challenges
    3. Virtualization Technologies -CPU
    4. Virtualization Technologies -Mmeory
    5. Virtualization Technologies -I/O
    6. Some VMMs & VMM Optimization
    7. Summary

- Lec 5+6 OS/System API/Interface

    1. Introduction
    2. Rethinking the Library OS from the Top Down
    3. DPDK: Accelerating the I/O Path 
    4. Dune: Safe User-level Access to Privileged CPU Features
    5. Safe and Secure Drivers in High-Level Languages
    6. Summary

- Lec 7+8 OS for MultiCore Architecture
    1. Introduction
    2. How to analyze the OS bottleneck for multicore arch
    3. How to optimize the OS for multicore arch
    4. Optimizing the OS performance from MIT's research 
    5. Scalable Kernel TCP Design and Implementation for Short-Lived Connections
    6. Summary

- Lec 9+10 OS/System Security

    1. Introduction
    2. Improving Integer Security for Systems with KINT
    3. PF-Miner: A new paired functions mining method for Android kernel in error paths
    4. RID: Finding Reference Count Bugs with Inconsistent Path Pair Checking
    5. Summary

- Lec 11+12 Correctness: OS/System Verification

    1. Introduction
    2. seL4: Formal Verification of an OS Kernel
    3. Jitk: A trustworthy in-kernel interpreter infrastructure
    4. Hyperkernel: Push-Button Verification of an OS Kernel
    5. Summary

- Lec 13+14 OS Kernel and HLL

    1. Introduction
    2. Multiprogramming a 64 kB Computer Safely and Efficiently
    3. The benefits and costs of writing a POSIX kernel in a high-level language
    4. Summary

- Lec 15+16 Invited Talks From Visitors & Students

    1. High-Performance Network Optimization on Data Center
    2. Security OS Design for  Multi-tenancy
    3. OS Performance Optimization for Serverless Service 
