Introduction
-------------

This package contains a simple framework that allows BG users to run multiple
independent tasks over a single outer qsub scheduler block allocation. The
current implementation is for the ALCF BlueGene systems but can be extended to
any system that supports schedulers such as Cobalt.


Motivation
-----------

Users often have relatively small but many independent tasks to run over large
systems such as BG. Submitting independent jobs for these tasks is inconvenient
and inefficient for the following reasons:

1) A new task has to wait in the queue if the system is busy.

2) It takes a considerable time to boot the compute nodes on systems like BG.
This is desirable from the power efficiency point of view for large tasks but
is a waste for many independent short tasks.

The dodone framework addresses the above problems.

Mechanism
----------

The core mechanism is as follows: Two directories are maintained: `dirtodo` and
`dirdone`; `dirtodo` contains the tasks that are needed to be done and
`dirdone` contains tasks that are already done. Two main scripts are
maintained: `startjob` and `ksub`. The `startjob` script starts an outer Cobalt
`qsub` job for a set long time period. The `startjob` script in turn invokes
another script--the `runjob` script. The `runjob` script runs in an infinite
loop until the walltime of outer `qsub` job gets exhausted. This script picks
up the tasks from `dirtodo`, runs them and moves them to `dirdone`. While the
script is running, users can drop any tasks in the `dirtodo` directory via the
`ksub` script and they will be picked up by the script and executed. If there
is nothing to be done, the script will continue to run in loop.

Users can provide arguments to their application by adding a row in the config
file. For example, if my application is "miniFE.x" and arguments are `-nx 10
-ny 20 -nz 30`, the corresponding line in config will be like so:

```bash
miniFE.x -nx 10 -ny 20 -nz 30
```

Usage
------

Initialize directories with reset command:

```bash
$ ./clean && ./reset
```

Add or link executables to the `dirtodo` directory.

Add executable parameters/args to the `config` file as shown in previous section.

Start the outer qsub command as shown in `startjob`:

```bash
$ qsub -A ATPESC2014 -n 1 -t 5 --mode script runjob
```

Make changes to the nodes (`-n`), walltime (`-t`) and/or allocation (`-A`) per your setup and choosing.

Now, to submit your "jobs" to this system, simply invoke `./ksub` like so:

```bash
$ ./ksub -e <executable>
```

For example:

```bash
$ ./ksub -e /home/ketan/MantevoApps/miniFE-2.0_ref/src/miniFE.x
```

This will put the `miniFE.x` executable in the todo directory and run it on the
compute node.
