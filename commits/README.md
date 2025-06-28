# Commit Lists

This directory contains lists of live-patchable commits for MariaDB and Redis, gathered using the [patch-crawler](../patch-crawler). The `paper/` directory holds the commit lists we crawled as references, while the `reproduction/` directory is intended for storing commit lists generated during the execution of `patch-crawler`. 

You can use the following script to compare the commit lists in the `reproduction/` and `paper/` directories:

```
cd dbms-live-patching/commits
./diff
```

This script provides the most accurate comparison, as it checks not only the number of live-patchable commits but also their exact git hashes. For a mapping between specific files and the corresponding statements in the paper, see the [Paper](#paper) section below.

 **Ideally, there should be no discrepancies between the patches of the paper and the patches crawled for the reproduction package. However, MariaDB's commit lists can vary slightly due to the use of `ccache` (which speeds up compilation) and `perf`, resulting in non-deterministic outcomes. This may lead to minor differences. In our tests of the reproduction pipeline, we only found differences in a few commits. Nevertheless, since we manually selected the five commits primarily used for the experiments, these discrepancies do not pose a limitation.**

## Paper

The following live-patchable commit lists correspond to the following statements in the paper:

- File `mariadb.commits.success.wfpatch.original`:

  - Section "6 Experiments"; ***MariaDB*** italic bold highlight; **Patches** bold highlight:

    >  Despite this, our automated pipeline, scanning versions 10.5.0–10.5.13 of MariaDB on GitHub, identified 117 live-patchable code changes.

    The following command should show `117` as a result:

    ```
    cat reproduction/mariadb.commits.success.wfpatch.original | wc -l
    ```

    

- File `mariadb.commits.success.wfpatch.perf.original`:

  - Section "6 Experiments"; ***MariaDB*** italic bold highlight; **Patches** bold highlight

    > While all 117 patches contribute to our broader analysis (see Section 6.3), we imposed two strict filters for our in-depth evaluation: (1) the patch must be officially labeled a “bug” in the MariaDB bug tracker, and (2) it should modify a function in the stack trace below the do_command function. In consequence, the patch actually affectsa function that is executed during a benchmark run (and not somedormant code, which is low risk to patch). The five selected patches are presented in Table 1.

    These patches were also used for the experiment shown in Figure 10.
    
    The first filter (1) must be done **manually** (see next bullet point), but the second filter (2) was evaluated automatically. The `mariadb.commits.success.wfpatch.perf.original` file contains all live-patchable commits on which the second filter was applied. This resulted in `17` commits. 

  ```
  cat reproduction/mariadb.commits.success.wfpatch.perf.original | wc -l
  ```

  - Table 1

    The five live-patchable commits were selected **manually** by using the MariaDB bug tracker (https://jira.mariadb.org/). The commit hashes shown in Table 1 should be contained in the `reproduction/mariadb.commits.success.wfpatch.perf.original` file. For example, check the file output and compare the commit hashes with Table 1:

    ```
    cat reproduction/mariadb.commits.success.wfpatch.perf.original
    ```

- File `redis.commits.success.wfpatch`:

  - Section "6 Experiments"; ***Redis*** italic bold highlight; **Patches** bold highlight:

    > From the development history of Redis versions 5.0.0–7.0.11 on GitHub, we extracted 529 patches.

    These patches were also used for the experiment shown in Figure 10.

    The following command should show `529` as a result:
    
    ```
    cat reproduction/redis.commits.success.wfpatch | wc -l
    ```

  ---

  ## Implementation Details
  
  The commit list files in this directory are symbolic links to those in the `paper/` directory and will be used for conducting the experiments (so please do not modify them).
