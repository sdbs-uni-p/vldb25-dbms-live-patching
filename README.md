# Reproduction Package for the VLDB paper: ***The Case for DBMS Live Patching***

![VLDB Badge](https://img.shields.io/badge/VLDB-2025-blue.svg)
[![DOI: 10.5281/zenodo.11370683](https://zenodo.org/badge/doi/10.5281/zenodo.11370683.svg)](https://doi.org/10.5281/zenodo.11370683) 
![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)


This repository contains all scripts and additional material referenced in the paper. It also contains instructions on how to reproduce the results. The original data, a VM prepared for reproducing our results, etc., can be found in [Zenodo](https://zenodo.org/records/12166690).

## Supplementary Material
The directory [supplementary-material](supplementary-material) contains the additional charts referenced in the paper.

## Hardware Requirements

To be able to execute this reproduction package, the following hardware is recommended:

- At least 1 TB of free disk space
- At least 384 GB of main memory
- At least 30 CPU cores (40 cores or more are recommended)

The entire reproduction pipeline takes about 160 hours.

## Reproduction Pipeline

The following steps can be used to reproduce this research. Each subdirectory, however, contains detailed instructions, including all executed commands, how to control the scripts, and other implementation-specific information.

0. **Initial**: Prepare the system.

   This reproduction package can either be executed on a prepared server (highly recommended), or in a prepared QEMU VM. Please see the README of the [qemu](qemu) directory for details. 

1. **Crawl Development History**: Identify live patchable source code changes (see directory [patch-crawler](patch-crawler) for implementation details).

   Patch crawling must be done using the ***unmodified*** Linux kernel.

   ```
   cd ~
   ./kernel-regular
   sudo reboot
   ```

   Perform the experiment:

   ```
   cd ~/dbms-live-patching
   ./reproduce-crawling
   ```

   Results are stored in the `~/dbms-live-patching/commits` directory. Please see the [commits](commits) directory for the evaluation of the results (Table 1 and the *patches* used in Figure 10 of the paper).

2. **Perform Experiments**: Conduct the experiments (see directory [experiments](experiments) for implementation details). 

   > If you run the experiments on hardware different from ours (see the [Hardware](#hardware) section), please read the implementation details carefully. Some aspects of the experiments are system-specific, for example, CPU pinning or preset queries-per-second (QPS) values, which may need to be adjusted to match your setup.

   The experiments must be executed using the ***MMView*** Linux kernel.

   ```
   cd ~
   ./kernel-mmview
   sudo reboot
   ```

   Perform the experiment:

   ```
   cd ~/dbms-live-patching
   ./reproduce-experiments
   ```

   The results (the raw experiment data) will be stored in the respective directories in the [data](data) directory. The experiment data is transformed into DuckDB database files and analyzed in the last step (see next step).

3. **Transform Experiment Data & Analyze Results**: Convert experiment data into a DuckDB database (see directory [transformation](transformation) for implementation details) and analyze the results and generate plots (directory [plotting](plotting) for implementation details).

   Data transformation and analysis must be done using the ***unmodified*** Linux kernel.

   ```
   cd ~
   ./kernel-regular
   sudo reboot
   ```

   Transform the experiment data and analyze the results:

   ```
   cd ~/dbms-live-patching
   ./reproduce-experiments
   ```

   The output data corresponds to the following statements in the paper:

   - Figure 1 and 5 - 10:

     All *plots* are stored in the [plots/reproduction](plots/reproduction) directory. Please see the README of the [plots](plots) directory for the file mapping, which file corresponds to which figure in the paper.

   - Table 2:

     The data of the table can be evaluated as follows:

     ```
     cd ~/dbms-live-patching/plotting/latency-breakdown
     ./new-as
     ./patch-application
     ./reach-q
     ./switch-as
     ```

   - Section "6.1.1 OLTP Workloads" :

     > While we did do not encounter deadlocks in any run, this is not the case for the earlier implementation [49] by Rommelet al. for the MariaDB thread pool policy, which does not employthe priority-based quiescence proposed by us. When we replicate our experiment using the YCSB and TPC-C benchmark with their implementation, every patch request inevitably causes a deadlock (in 100 out of 100 runs).

     The experiment can be evaluated as follows:

     ```
     cd ~/dbms-live-patching/plotting/comparison
     ./evaluate-comparison
     ```

     The version `wfpatch.patch-18502f99eb24f37d11e2431a89fd041cbdaea621` is our implementation, while `mariadb-wf-10.3.15` is the version of Rommel et al. [2]. 

     Example output:

     ```
     /home/repro/dbms-live-patching/plotting/comparison/../../data/threadpool-comparison/result-ycsb/wfpatch.patch-18502f99eb24f37d11e2431a89fd041cbdaea621-ycsb
     Total Experiments: 100
     Total 'Deadlocks': 0
     Total #Recorded Latencies: 164335571
     Maximum #Recorded Latencies: 1685122
     Total Patchings during all Experiments: 2900
     ```

     - The path contains the respective version.
     - Lines `Total #Recorded Latencies` and `Maximum #Recorded Latencies` can be ignored.
     - `Total 'Deadlocks':` should be `0` and `Total Patchings during all Experiments:` should be `2900`. This means that no deadlock occurred, and all patches were successfully applied. When `Total 'Deadlocks'` shows a value of `100`, this means that every patch request caused a deadlock.

We also provide the results from our experiments. This means experiments can be performed directly, or plots can be generated immediately (but you may have to move/rename some directories to match the expected names/locations). See the [original data](#original-data) section for more details.

## Hardware

We conducted our experiments on a Dell PowerEdge R640 server equipped with:

- CPU: 2x Intel Xeon Gold 6248R
  - 24 cores/48 threads per CPU
  - 3.0 GHz
- Main Memory: 384 GB (12x 32 GB)
- Disk: 3.0 TB SSD

> **_NOTE:_** Our experiments use CPU pinning tailored to the specific number of CPU cores in our hardware. You may need to adjust the pinning to match your hardware configuration. Detailed instructions for these adjustments are provided in the steps as needed.

## System Optimizations

To precisely measure (tail) latencies, we performed the following system optimizations:

- Disable Intel Hyper-Threading (in BIOS)
- Operate all CPU cores at maximum frequency (see commands below)
- Map the `/tmp` directory to main memory

### CPU Core Frequency

Execute the following commands on the host system (Linux) to set the CPU frequency of all cores to maximum (even when idle):

```
sudo cpupower frequency-set -g performance

```

Once all experiments were performed, the CPU can be reset:

```
sudo cpupower frequency-set -g schedutil
```

## Original Data

We provide our results in Zenodo (https://doi.org/10.5281/zenodo.11370683). 

- Raw experiment data: 
  - `./download-raw-data`
- Transformed experiment data:
  -  `./download-transformed-data`
- Download MariaDB database directories:
  -  `./experiments/mariadb/download-mariadb-dataset`

> **_NOTE:_** When using one of the `download-` scripts, the data is downloaded, extracted and the downloaded archive gets deleted.

### raw vs. transformed data

The raw data from most experiments is transformed into a DuckDB database, which is then used for all subsequent analysis and plotting. However, some experiment data is directly used for analysis and plotting without being converted into a DuckDB database. All data used for analysis and plotting is included in the "transformed" data.

 The "raw" data includes the original data from which the DuckDB databases were created and is provided for reference.

## WfPatch Extension

Please refer to the [wfpatch-extension](wfpatch-extension) directory for a detailed overview of our extensions to the WfPatch user space library, as well as the original work by Rommel et al. [2]. To ensure clarity regarding the distinctions between the original work and our contributions, we have created this directory. Here, we explicitly outline the nature and extent of our modifications, as it is not always feasible to differentiate these changes directly within the reproduction package.

[2] Florian Rommel, Christian Dietrich, Daniel Friesel, Marcel Köppen, ChristophBorchert, Michael Müller, Olaf Spinczyk, and Daniel Lohmann. 2020. *From Global to Local Quiescence: Wait-Free Code Patching of Multi-Threaded Processes*. In Proc. OSDI. 651–666.

## Directory Overview

- `supplementary-material/`
  - Contains all supplementary material mentioned/referenced in the paper.
- `qemu/`
  - Location for the QEMU VM.
- `patch-crawler/`
  - Contains all scripts to crawl the development history of MariaDB and Redis for live patchable commits.
- `commits/`
  - Contains lists of live patchable commits of MariaDB and Redis, crawled by the scripts in  `patch-crawler/`
- `experiments/`
  - Contains all scripts to run experiments.
- `data/`
  - Contains empty directories in which the results of the experiments will be stored.
- `transformation/`
  - Contains scripts to transform experiment data into DuckDB databases.
- `plotting/`
  - Contains all scripts to analyze the (transformed) benchmark data and to create plots.
- `plots/`
  - Contains a directory in which all plots will be stored.
- `wfpatch-extension`/
  - Highlights our extensions to the WfPatch user space library and other scripts.
- `utils/`
  - Utility scripts used in different steps of the reproduction pipeline.

---

## Zenodo Change History
### v1 -> v2
- Added pre-built Docker image for crawling patches.
- Updated QEMU VM:
    - Fixed ccache config.
    - Installed Docker.
    - Added tmpfs mapping for `/tmp`

### v2 -> v3
- Updated link to repository in QEMU VM.

