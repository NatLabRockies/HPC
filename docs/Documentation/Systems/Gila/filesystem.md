# Gila Filesystem Architecture Overview

## Home Directories: /home

`/home` directories are mounted as `/home/<username>`. To check your usage in your /home directory, visit the [Gila Filesystem Dashboard](https://influx.hpc.nlr.gov/d/ch4vndd/ceph-filesystem-quotas?folderUid=fexgrdi5pt91ca&orgId=1&from=now-1h&to=now&timezone=browser&tab=queries). You can also check your home directory usage and quota by running the following commands: 

```
# Check usage
getfattr -n ceph.dir.rbytes <directory path>
# Check quota
getfattr -n ceph.quota.max_bytes <directory path>
```

If you need a quota increase in your home directory, please contact [HPC-Help@nlr.gov](mailto:HPC-Help@nlr.gov).

## Project Storage: /projects

Each active project is granted a subdirectory under `/projects/<projectname>`. There are currently no quotas on `/projects` directories. Please monitor your space usage at the [Gila Filesystem Dashboard](https://influx.hpc.nlr.gov/d/ch4vndd/ceph-filesystem-quotas?folderUid=fexgrdi5pt91ca&orgId=1&from=now-1h&to=now&timezone=browser&tab=queries). 

Note that there is currently no `/projects/aurorahpc` directory. Data can be kept in your `/home` directory. 

## Scratch Storage

The scratch filesystem on Gila is a spinning disk Ceph filesystem, and is accessible from login and compute nodes. The default writable path for scratch use is `/scratch/<username>`. 

!!! warning
    Data in `/scratch` is subject to deletion after 28 days. It is recommended to store your important data, libraries, and programs in your project or home directory. 

## Temporary space: $TMPDIR 

When a job starts, the environment variable `$TMPDIR` is set to `/scratch/<username>/<jobid>` for the duration of the job. This is temporary space only, and should be purged when your job is complete. Please be sure to use this path instead of `/tmp` for your tempfiles.

There is no expectation of data longevity in the temporary space, and data is purged once a job has completed. If desired data is stored here during the job, please be sure to copy it to a project or home directory as part of the job script before the job finishes.

## Mass Storage System

There is no Mass Storage System for deep archive storage from Gila.

## Backups and Snapshots

There are no backups or snapshots of data on Gila. Though the system is protected from hardware failure by multiple layers of redundancy, please keep regular backups of important data on Gila, and consider using a Version Control System (such as Git) for important code. 
