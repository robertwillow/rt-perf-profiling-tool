# RT Profiling Tool User Guide

</br>

## Table of Contents

* [1 Introduction](#1)
  * [1.1 Overview](#1.1)
* [2 Step by step instructions on RT-Profiling tool](#2)
  * [2.1 Setup environment](#2.1)
    * [2.1.1 Install software dependencies](#2.1.1)
    * [2.1.2 Get the source code](#2.1.2)
    * [2.1.3 Build latency/cyclictest utilities](#2.1.3)
    * [2.1.4 Make nmon/nmonchart executable](#2.1.4)
  * [2.2 Generate RT performance data](#2.2)
    * [2.2.1 Generate Jitter/Cycle data with latency](#2.2.1)
    * [2.2.2 Generate Jitter/Cycle data with cyclictest](#2.2.2)
  * [2.3 Collect RT performance data](#2.3)
    * [2.3.1 Observe the performance data in interactive mode](#2.3.1)
    * [2.3.2 Capture the performance data in data collector mode](#2.3.2)
  * [2.4 Generate html page](#2.4)
  * [2.5 Analyze the result](#2.5)
* [3 Roadmap](#3)
  * [3.1 Supported features](#3.1)
  * [3.2 In progress features](#3.2)
  * [3.3 Todo Features](#3.3)    
* [4 License](#4)

<br/>

# <a name="1"/>1 Introduction

## <a name="1.1"/>1.1 Overview

In industrial controller scenarios, the real-time performance indicators like jitter & cycle time are the two key criteria to evaluate the performance of RT control system. However, how to troubleshoot and tune real-time performance is always a challenge to developers. There are many factors like IRQs, network traffic could affecting the overall performance of RT control system.

This document provides a step-by-step guidance for RT control system developers/analyzers on how to setup, collect and analyze real-time control related jitter/cycle parameters as well as other system characters of the real-time control system like CPU utilization, disks accessing, network transferring, IRQs using customized nmon/nmonchart toolset.

<br/>

# <a name="2"/>2 Step by step instructions on RT-Profiling tool

## <a name="2.1"/>2.1 Setup environment

The RT profiling tool should be able to run on all the latest Linux distributions theoretically. We will use Linux Ubuntu 16.04 x64 LTS as the default environment throughout this guide in order to minimize clutters caused by various Linux distributions.

### <a name="2.1.1"/>2.1.1 Install software dependencies

Please install *gcc*, *make*, *git* utilities and *ncurses* dev lib on the target system if any of these packages is unavailable on the system.

1. Connect to the internet.

2. Open a command prompt terminal window.

3. Type following instruction in the terminal, input root password as prompt. Please adjust the command accordingly to install the corresponding dependency if you use Fedora or other Linux distributions.

```Shell
sudo apt install gcc libncurses5-dev make git
```

### <a name="2.1.2"/>2.1.2 Get the source code

Get the latest source code of RT-Profiling tool from the git server or from package file offline.

1. Get the source code from the git server.

```Shell
git clone https://github.com/intel/rt-perf-profiling-tool
```

### <a name="2.1.3"/>2.1.3 Build latency/cyclictest utilities

Type following commands in the terminal to build **latency** and **cyclictest** utilities.

1. Apply the patch. Where *xxx* stands for the absolute path of patch **0001-capture-jitter-performance-in-share-memory.patch** under *collection/latency* folder you got at step 2.

```Shell
cd testsuite/latency/
git apply --reject xxx
```

2. Build **latency** utility.

```Shell
make
```

3. Get rt-tests source code v1.0.

```Shell
git clone https://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git
cd rt-tests/
git checkout stable/v1.0
```

4. Apply the patch. Where *yyy* stands for the absolute path of patch **0001-capture-latency-in-share-memory.patch** under collection/cyclictest folder you got at step 2.

```Shell
git apply --reject yyy
```

5. Build **cyclictest** utility.
```Shell
make
```

### <a name="2.1.4"/>2.1.4 Make nmon/nmonchart executable

Build **nmon** utility, and then make **nmon** utility and **nmonchart** script executable at any place.

1. Move to the nmon directory.

```Shell
cd rt_profiling-tool/agent/nmon
```

2. Execute following command to build nmon binary. Where *arch* stands for the CPU architecture of the system under test, *os* stands for the target operating system nmon will be executed. We'll use *nmon_AMD64_ubuntu1604* as an example here. Please specify the designated configuration combination value as the parameter of the **make** according to the real condition. 

```Shell
make <nmon_arch_os>
```

3. Setup the symbolic link of nmon utility. Here we use *nmon_AMD64_ubuntu1604* as the sample, please adjust it for your self. 

```Shell
sudo ln -s rt_profiling-tool/agent/nmon/nmon_AMD64_ubuntu1604 /usr/local/bin/nmon
```

4. Setup the symbolic link of nmonchart script.

```Shell
sudo ln -s rt_profiling-tool/analysis/nmonchart/nmonchart /usr/local/bin/nmonchart
```

## <a name="2.2"/>2.2 Generate RT performance data

We can use latency or cyclictest utility to generate RT performance data.

### <a name="2.2.1"/>2.2.1 Generate Jitter/Cycle data with latency

You can launch **latency** in terminal to generate RT performance data.

```Shell
sudo ./latency -r
```

> ***-r***: update jitter performance in share memory

### <a name="2.2.2"/>2.2.2 Generate Jitter/Cycle data with cyclictest

You can launch **cyclictest** in terminal to generate RT performance data.

```Shell
sudo ./cyclictest -N -t 4 -e
```
>
> ***-N***: show results in nanoseconds
>
> ***-t 4***: start 4 threads to measure latency
>
> ***-e***: update latency performance in share memory

## <a name="2.3"/>2.3 Collect RT performance data

Nmon can gives you a huge amount of important performance information in one go. It can output the data on the screen in interactive mode or save the data to a comma separated file for analysis in data collector mode.

### <a name="2.3.1"/>2.3.1 Observe the performance data in interactive mode

You can observe the Jitter time, Cycle time as well as CPU, memory, network, disks etc. data directly on the screen and updated every second if you launch the nmon without arguments.

1. Open a terminal, type *nmon*. Nmon with help info page will pop up.

![Nmon terminal window](https://github.com/intel/rt-perf-profiling-tool/blob/master/doc/images/image1.png "nmon terminal window")

2. You can press the corresponding keys to toggle on or off statistics. Say, you can press "ecCUd" to show RT info, CPU Utilization, CPU Utilization Stats, CPU Utilization Wide View and Disk I/O info.

![Nmon terminal menu](https://github.com/intel/rt-perf-profiling-tool/blob/master/doc/images/image2.png "nmon terminal menu")

3. You can press "+" or "-" key to increase or decrease nmon sampling
intervals in real time. By default, the screen refresh time is two
seconds. The minimum refresh time is one second.

### <a name="2.3.2"/>2.3.2 Capture the performance data in data collector mode

1. Open a terminal, launch **nmon** utility with parameters specified. The .nmon data files it captured will be saved under *YYMMDD_HHMM* folder(where *YYMMDD* and *HHMM* are the beginning time of the collection).

```Shell
nmon -feiTU -s 1 -c 36000
```

> Notes: Following parameters are supported when ***nmon*** is launched.
>
> ***-f***: Must be the first option on the line (switches off
> interactive mode). Saves data to a CSV Spreadsheet format .nmon file
> in the local directory
>
> ***-e***: Specify RT Performance info to be collected
>
> ***-i***: Specify IRQ info to be collected.
>
> ***-t***: Collect Top Process info also
>
> ***-T***: Collect command line info also
>
> ***-U***: Collect CPU Utils info als；
>
> ***-s <seconds>***: Time between data snapshots
>
> ***-c <count>***: Of snapshots before exiting。
>
> Eg: -feiTU -s 1 -c 36000 stands for collect the data for 36000 times with time interval for one second. IRQ and Top process info are collected also. The .nmon file will be saved on current folder for further investigation by default.

## <a name="2.4"/>2.4 Generate html page

Open a terminal, move to the folder where nmon utilities are saved. Execute nmonchart script to generate html files.

```Shell
nmonchart HHMMDD__HHMMM
```
> *HHMMDD_HHMMM* is the folder name where .nmon data files are stored.


The webpage files *xxxx.html* will be generated under ***HHMMDD_HHMMM*** folder.

## <a name="2.5"/>2.5 Analyze the result

Double click the xxxx.html file generated by nmonchart to open the dynamic webpage as Figure 2.5.1.

![nmonchart webpage](https://github.com/intel/rt-perf-profiling-tool/blob/master/doc/images/image3.png "nmonchart webpage")

> Figure 2.5.1 Homepage of nmonchart

You can click **Configuration**, **Top Summary**, **Top Command** and **Top Disk** buttons separately on the first row to get the overall information, overall CPU consumptions and disk usage.

You can click on **CPU util.** **CPU Use** and **CPU All Utils,** button to monitor CPU related criteria.

![nmonchart cpu webpage](https://github.com/intel/rt-perf-profiling-tool/blob/master/doc/images/image4.png "nmonchart cpu webpage")

> Figure 2.5.2 nmonchart

# <a name="3"/>3 Roadmap
## <a name="3.1"/>3.1 Supported features
* Nmon based data agent
    * Support cyclic data source
    * Support latency data source
    * Support real-time data metrics
    * Support disk/mem/net/interrupt etc.. Data metrics
* Web based visualization
    * Charting for misc. data metrics
    * Real-time data metrics  showing with defined & selected other metrics

## <a name="3.2"/>3.2 In progress features
* Web based visualization
    * Change code base to echarts based chart
    * Dual-Y axis to show data in different granularity

## <a name="3.3"/>3.3 Todo Features
* Nmon based data agent
    * support msi_latency data source
* Web based visualization
    * Charting for misc. data metrics
    * Real-time data metrics showing with defined & selected other metrics
    * jitter statistics analysis
    * Scoring
    * normal distribution
    * Sorting
    * Plugin based value-adding analysis module framework & sample code
    * Correlation
    * Abnormal
    * Value added modules

# <a name="4"/>4 License
The source code is licensed under GNU General Public License v3.0 or later. See [LICENSE](LICENSE) file for details.
It includes software developed from 3rd-party, See [NOTICE](NOTICE) file for details.
