# Plot Storage

Contains an empty directory (`reproduction/`) in which the plots will be stored from this reproduction pipeline. The `paper/` directory contains all plots from the paper, including additional ones referenced or mentioned therein.

For easy comparison, you can use the `./run` script to start a web-server (streamlit) which allows to compare the plots of our paper (`paper/` directory) with the plots of your reproduction pipeline (`reproduction/` directory). This script starts a web-server (you may need to expose the port if running inside a VM) that allows you to compare our plots with those from your reproduction pipeline side-by-side.

```
cd ~/dbms-live-patching/plots

# Starts a web-server (streamlit) for easy chart comparison
./run
```

However, you can also compare the plots manually (the plots in the `reproduction` directory) with the figures in our paper. Please see the list below ([Paper](#paper) section) of which file corresponds to which figure in our paper.

## Paper

The following files correspond to the following figures in the paper:

- Figure 1: `Patch-One-by-One.pdf`
- Figure 5: `QPS-Time-wfpatch.patch-18502f99eb24f37d11e2431a89fd041cbdaea621.pdf`
- Figure 6: `OTPC-CH-Latency-per-Worker.pdf`
- Figure 7: `TP-Synchronization-Time-Boxplot-wfpatch.patch-18502f99eb24f37d11e2431a89fd041cbdaea621.pdf`
- Figure 8: `OTPC-Latencies-Cutout.pdf`
- Figure 9: `Redis-AS-Fork-Line.pdf`
- Figure 10: `OTPC-Redis-ELF-Size.pdf`

**Other Files**: As explained in the paper, we conducted experiments for all five real-world patches (see Table 1 of the paper), although the corresponding charts were not included in the paper. Files with a git commit suffix represent results for the other patches. Additionally, we ran the experiments under both the one-thread-per-connection (OTPC) policy and the thread pool (TP) policy; this is reflected in the file name prefixes (`OTPC` or `TP`). Finally, files prefixed with `arxiv` contain the charts included in the arXiv version of the paper ([arxiv.org/abs/2410.09925](https://arxiv.org/abs/2410.09925)).
