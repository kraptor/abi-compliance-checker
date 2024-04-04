### py-abi-compliance-checker

If you maintain packages written in C or C++, chances are you're already familiar with the following tools:
- [abi-dumper](https://github.com/lvc/abi-dumper)
- [abi-compliance-checker](https://github.com/lvc/abi-compliance-checker)

If you're not, please click the links and read more about them, they're great tools.

There are two modes in which `abi-compliance-checker` may be executed:

1. It may run in a mode that requires binaries/libraries to be compiled including debuginfo.
   This mode is more accurate, and it's **this mode that `py-abi-compliance-checker` focuses on**.

2. It may also run in a mode that doesn't require debuginfo.
   This mode is less accurate, but also good, the downsides are:
   - It requires specially crafted XML files.
   - It requires your host system to compile headers for the target OS you're trying to analyse.
     If there's enough of a gap between your system and the target system, it becomes a hassle.

**`py-abi-compliance-checker` is simply a wrapper around `abi-dumper` and `abi-compliance-checker` that makes them much easier to use for large-scale API/ABI comparisons.**

#### What does this even do?

`py-abi-compliance-checker` generates a very detailed report with the following information:

- All headers.
- All shared objects.
- All debuginfo files.
- It then relates all shared objects to their debuginfo files. If a debuginfo file can't be located, it exits.
- It then compares all shared objects across V1 and V2, the versions we're trying to compare, reporting if any shared objects are in V1, but are not in V2, and vice-versa. So we get insights if any `.so` files have been deprecated across versions.
- It then invokes `abi-dumper` and creates the ABI dump for each `.so` that is present in both V1 and V2.
- And then it invokes `abi-compliance-checker` for each ABI dump, finally giving us the HTML reports, which are all neatly named.

#### How can I use it?

1. Install `abi-compliance-checker` and `abi-dumper` using your package manager of choice.

2. Clone this repository, maybe place its scripts in your `$PATH`. Make sure they're executable. `extract-rpm` is a tiny, convenient Bash script that extracts RPMs.

3. If you're running Debian, Ubuntu, or some other system, you just need to be able to extract the package files to a single location, populating a given directory for the specific version of the package you're trying to compare.

4. For example, if we're comparing the differences between 4 packages: `perl-5.26.rpm`/`perl-debuginfo-5.26.rpm` and `perl-5.28.rpm`/`perl-debuginfo-5.28.rpm`, we could extract the contents of `perl-5.26.rpm`/`perl-debuginfo-5.26.rpm` to a directory named `perl-5.26`, and the contents of `perl-5.28.rpm`/`perl-debuginfo-5.28.rpm` to a directory named `perl-5.28`.

5. We should now have two directories: `perl-5.26` and `perl-5.28`, with those packages' contents.

6. Now you can finally use this tool :D

   The syntax is:
   `py-abi-compliance-checker <package name> <version 1> <directory with version 1 files> <version 2> <directory with version 2 files>`.

   So we could do:
   `py-abi-compliance-checker perl 5.26 perl-5.26 5.28 perl-5.28`.

7. All output data will be written inside a newly created directory named `perl-5.26-to-5.28`.

   The `.json` files contain a report generated by `py-abi-compliance-checker` itself.
   It shows all the data that was found by the tool itself, and that was used for the analysis.

   The `html` directory contains the HTML reports generated by `abi-compliance-checker`.
   The `abidumps` directory contains all ABI dumps generated by `abi-dumper`.

#### If you're on openSUSE or SUSE Linux Enterprise

1. If you're running openSUSE or SLE, you'll be able to retrieve the RPMs using `osc getbinaries --debuginfo`.
   Suppose we wanted to compare Perl5 from Factory to Perl5 from SLE-15-SP5:

   For Factory:

   `osc getbinaries openSUSE:Factory perl standard x86_64 --debuginfo`

   `mv binaries perl-factory`

   `cd perl-factory`

   `extract-rpm *.rpm`

   `cd ..`

   For SUSE:SLE-15-SP5:Update:

   `isc getbinaries SUSE:SLE-15-SP5:Update perl standard x86_64 --debuginfo`

   `mv binaries perl-sle15-sp5`

   `cd perl-sle15-sp5`

   `extract-rpm *.rpm`

   `cd ..`

2. And then run this tool:
   `py-abi-compliance-checker perl 5.26.1 perl-sle15-sp5 5.38.2 perl-factory`.
