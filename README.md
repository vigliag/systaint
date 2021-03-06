# SysTaint

In my thesis work, SysTaint, we explored the use of full-system record-replay approaches to study and reverse engineer malware, with the goal of easing the analysis of malware whose behavior depends on
network communications with external servers under the control of malicious actors.

We developed our solution on top of the PANDA reverse engineering platform, writing plugins to perform data-flow analysis on the data the sample exchanges with the operating system and to collect in-depth
data about the sample's internal functions.

My thesis is available here
([text](https://vigliag.github.io/files/systaint-thesis.pdf),
[defense slides](https://vigliag.github.io/files/systaint-thesis-defense.pdf)).

The main changes with respect to the PANDA codebase are detailed below. All plugins target Windows 7 x86 SP1:

#### New plugins

- [SysTaint](panda/plugins/systaint/) collects in-depth data about the inner functions of a sample under analysis. It uses taint analysis to track the flow of data between system calls and previously identified cryptographic and compression functions, and annotates the inputs to the sample's inner functions with provenance information.
- [Fnmemlogger](panda/plugins/fnmemlogger) collects statistics about the functions called by the sample, which we use to recognize cryptographic functions by using heuristics
- [Procinfodump](panda/plugins/procinfodump) collects information about the processes in the recording at a given list of points in time. It embeds a python interpreter to efficiently use Rekall on the guest's memory, dumping the memory pages and the memory maps, and extracting the list of functions exported by all DLLs in a process's address space.
- [Tcgtaint](panda/plugins/tcgtaint) Is an alternative, tcg-based, taint tracking implementation, ported from the one used in [Qtrace](https://github.com/rpaleari/qtrace). It's fast and flexible, but works at byte-level and does not instrument the whole x86 instruction set. It requires minor changes to  QEMU's TCG, which have been added under the `CONFIG_QTRACE_TAINT` ifdef.
- [SysEvent](panda/plugins/sysevent) Receives hypercalls signalling the start and end of an event of a given thread. Used for Cuckoo Monitor integration.

#### Changes to existing plugins

- [StringSearch2](panda/plugins/stringsearch2) adds functionalities to the original stringsearch plugin: loading of the search terms from a directory (with automatic chunking), richer output in json format. 
- [CallstackInstr](panda/plugins/callstack_instr) was refactored. It assigns a unique identifier to the individual function calls, and uses the thread identifier to distinguish threads in Windows7 x86

#### Other code:

- [Analysis script](systaint_scripts) containing python functions to use in the various phases of the analysis (still need some cleanup)
- [Patched version of Cuckoo Monitor](https://github.com/vigliag/cuckoo_monitor_panda) that communicates with the SysEvent plugins via hypercalls.
- [Cuckoo with very minor edits](https://github.com/vigliag/cuckoo) to QEMU machinery module and to the script processing logs about Windows analyses.

---


# PANDA

[![Build Status](https://travis-ci.org/panda-re/panda.svg?branch=master)](https://travis-ci.org/panda-re/panda)

PANDA is an open-source Platform for Architecture-Neutral Dynamic Analysis. It
is built upon the QEMU whole system emulator, and so analyses have access to all
code executing in the guest and all data. PANDA adds the ability to record and
replay executions, enabling iterative, deep, whole system analyses. Further, the
replay log files are compact and shareable, allowing for repeatable experiments.
A nine billion instruction boot of FreeBSD, e.g., is represented by only a few
hundred MB. PANDA leverages QEMU's support of thirteen different CPU
architectures to make analyses of those diverse instruction sets possible within
the LLVM IR. In this way, PANDA can have a single dynamic taint analysis, for
example, that precisely supports many CPUs. PANDA analyses are written in a
simple plugin architecture which includes a mechanism to share functionality
between plugins, increasing analysis code re-use and simplifying complex
analysis development.

It is currently being developed in collaboration with MIT Lincoln
Laboratory, NYU, and Northeastern University.

## Building

Because PANDA has a few dependencies, we've encoded the build instructions into
a script, [panda/scripts/install\_ubuntu.sh](panda/scripts/install\_ubuntu.sh).
The script should actually work on Debian 7/8 and Ubuntu 14.04, and it
shouldn't be hard to translate the `apt-get` commands into whatever package
manager your distribution uses. We currently only vouch for buildability  on
Debian 7/8 and Ubuntu 14.04, but we welcome pull requests to fix issues with
other distros.

Note that if you want to use our LLVM features (mainly the dynamic taint
system), you will need to install LLVM 3.3 from OS packages or compiled from
source. On Ubuntu 14.04 this will happen automatically via `install_ubuntu.sh`.
Alternatively, we have created an Ubuntu PPA at `ppa:phulin/panda`. You can use
the following commands to install all dependencies on 14.04 or 16.04:

```
sudo add-apt-repository ppa:phulin/panda
sudo apt-get update
sudo apt-get build-dep qemu
sudo apt-get install python-pip git protobuf-compiler protobuf-c-compiler \
  libprotobuf-c0-dev libprotoc-dev python-protobuf libelf-dev \
  libcapstone-dev libdwarf-dev python-pycparser llvm-3.3 clang-3.3 libc++-dev
git clone https://github.com/panda-re/panda
mkdir -p build-panda && cd build-panda
../panda/build.sh
```

Building on Mac is less well-tested, but has been known to work. There is a script,
[panda/scripts/install\_osx.sh](panda/scripts/install\_osx.sh) to build under OS X.

Finally, if you want to skip the build process altogether, there is a 
[Docker image](https://hub.docker.com/r/pandare/panda). You can get it by running:

    docker pull pandare/panda

Alternatively, you can pull the [latest build from an unofficial](https://hub.docker.com/r/thawsystems/panda) third party.

    docker pull thawsystems/panda

## Support

If you need help with PANDA, or want to discuss the project, you can join our
IRC channel at #panda-re on Freenode, or join the [PANDA mailing
list](http://mailman.mit.edu/mailman/listinfo/panda-users).

We have a basic manual [here](panda/docs/manual.md).

## PANDA Plugins

Details about the architecture-neutral plugin interface can be found in
[panda/docs/PANDA.md](panda/docs/PANDA.md). Existing plugins and tools can be found in
[panda/plugins](panda/plugins) and [panda](panda).

## Record/Replay

PANDA currently supports whole-system record/replay execution of x86, x86\_64,
and ARM guests. Documentation can be found in
[the manual](panda/docs/manual.md#recordreplay-details).

## Publications

* [1] B. Dolan-Gavitt, T. Leek, J. Hodosh, W. Lee.  Tappan Zee (North) Bridge:
Mining Memory Accesses for Introspection. 20th ACM Conference on Computer and
Communications Security (CCS), Berlin, Germany, November 2013.

* [2] R. Whelan, T. Leek, D. Kaeli.  Architecture-Independent Dynamic
Information Flow Tracking. 22nd International Conference on Compiler
Construction (CC), Rome, Italy, March 2013.

* [3] B. Dolan-Gavitt, J. Hodosh, P. Hulin, T. Leek, R. Whelan.
Repeatable Reverse Engineering with PANDA. 5th Program Protection and Reverse
Engineering Workshop, Los Angeles, California, December 2015.

* [4] M. Stamatogiannakis, P. Groth, H. Bos. Decoupling Provenance
Capture and Analysis from Execution. 7th USENIX Workshop on the Theory
and Practice of Provenance, Edinburgh, Scotland, July 2015.

* [5] B. Dolan-Gavitt, P. Hulin, T. Leek, E. Kirda, A. Mambretti,
W. Robertson, F. Ulrich, R. Whelan. LAVA: Large-scale Automated Vulnerability
Addition. 37th IEEE Symposium on Security and Privacy, San Jose,
California, May 2016.

## License

GPLv2.

## Acknowledgements

This material is based upon work supported under Air Force Contract No.
FA8721-05-C-0002 and/or FA8702-15-D-0001. Any opinions, findings,
conclusions or recommendations expressed in this material are those of
the author(s) and do not necessarily reflect the views of the U.S. Air
Force.
