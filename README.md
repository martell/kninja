# Kninja

Kninja is an experimental [Ninja](https://ninja-build.org/) build file
generator for Kbuild, the Linux kernel's build system.  It does not really
attempt to understand Kbuild files but instead just processes the output
generated by GNU Make's `--print-data-base`.  Configuration changes and changes
to files with special generation rules are not currently handled, but
modifications to most normal source and header files should just work.

The use of ninja results in vastly improved incremental build times:

| Change                    | make -j8 | make -j8 objectname | ninja  |
| --------------------------| -------- | ------------------- | ------ |
| No changes                |  2.254s  |       0.731s        | 0.065s |
| One change, compile error |  1.225s  |       0.765s        | 0.077s |
| One change, full link     |  5.915s  |       NA            | 4.482s |

The link time unsuprisingly dominates when performing small changes, but as can
be seen the time until the start of compilation (and thus the time until any
compile errors are detected) is several times smaller with ninja.

These numbers were measured with the [benchmarking script](benchmark.sh)
included in the repository.  Reading that script should also give you an
example of how to use kninja.

## Usage

Python 3.4+ is required.  Install the required packages with:

    pip3 install -r requirements.txt

Run `kninja.py` to generate Ninja build files.  This must also be run manually
after any changes to .config or Makefiles.

    $ kninja.py
    [INFO] Ensuring full build: make -j 8
      CHK     include/config/kernel.release
      CHK     include/generated/uapi/linux/version.h
      CHK     include/generated/utsrelease.h
      CHK     include/generated/timeconst.h
      CHK     include/generated/bounds.h
      CHK     include/generated/asm-offsets.h
      CALL    scripts/checksyscalls.sh
      CHK     include/generated/compile.h
      Building modules, stage 2.
      MODPOST 18 modules
    Kernel: arch/x86/boot/bzImage is ready  (#6)
    [INFO] Generating make database: make -p
    [INFO] Caching make database to .makedb
    [INFO] Parsing make database (2635077 lines)
    [INFO] Wrote build.ninja (2763 rules, 2744 build statements)
    [INFO] Wrote .ninja_deps (2155 targets, 939990 deps)
    [INFO] Wrote .ninja_log (2763 commands)
    [INFO] Checking ninja status: ninja -d explain -n
    [INFO] All OK
