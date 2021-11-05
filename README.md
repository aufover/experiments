### About project (and how it works)

[Automation of Formal Verification (AUFOVER) - slides](https://kdudka.fedorapeople.org/kdudka-aufover-211111.pdf)

##### Presented at DevConf 2021 (in the context of dynamic analysis):

* [slides](https://kdudka.fedorapeople.org/kdudka-devconf-21.pdf)

* [video](https://www.youtube.com/watch?v=FjV84hbD1GY)
* [demo](https://github.com/csutils/cswrap/wiki/csexec)

### How to conduct experiments

#### With [`csmock`](https://github.com/csutils/csmock)

```bash
$ dnf download --source <package> # get <package>.src.rpm
$ # or 
$ koji download-build src <package> # get <package>.src.rpm

$ csmock -j<n>-f -t <tool> --<tool>-add-flag '<flags>' --<tool>-timeout <n> --keep-going --no-clean ./<package>.src.rpm # add vim, if your planing to experiment manualy
```

`--keep-going` *continue as much as possible after an error* - error can be for example `timeout` 

`--no-clean` *do not clean chroot when it becomes unused* - this option is important if you like to run the verification by yourself

#### *With* your favorite verification tool

Tools list (formal verification only):

* [cbmc](https://github.com/diffblue/cbmc)            [EXPERIMENTAL] Bounded Model Checker for C and C++ programs
* [divine](https://divine.fi.muni.cz/)           [EXPERIMENTAL] A formal verification tool based on explicit-state model checking.
* [symbiotic](http://staticafi.github.io/symbiotic/)     [EXPERIMENTAL] A formal verification tool based on instrumentation, program slicing and KLEE.

more tools (`csmock --list-available-tools`): `clang`, `cppcheck`, `gcc`, `strace`,`valgrind`...

##### `TIMEOUT` hack

This is useful when you __do not__ want to run all tests in check section - guess your timeout number and the process (`rpmbuild -bi`) will luckily stop where you want!

* Change [line](https://github.com/csutils/csmock/blob/main/py/csmock#L283) in `csmock` script from `/bin/csmock`  like this:

```diff
         # construct basic `rpmbuild -bi ...` command
         rpm_opts = props.rpm_opts + extra_rpm_opts
-        cmd = "rpmbuild -bi --nodeps --short-circuit %s %s" \
+        cmd = "timeout <lucky_number> rpmbuild -bi --nodeps --short-circuit %s %s" \
             % (props.spec_in, strlist_to_shell_cmd(rpm_opts)) 
```

* Thanks to `--no-clean` option, run following command, where `<fedora-version-arch>` is environment where `csmock ...` command ran previously

    ```bash
    mock -r <fedora-version-arch> --shell
    ```
##### How to add (install) some package to `mock`?
```bash
$ mock --root <fedora-version-arch> --install <package> # e.g. fedora-version-arch = fedora-34-x86, package = vim
```



#### [CBMC](https://src.fedoraproject.org/rpms/cbmc) (tips and tricks):

##### When the output is unreadably large:

```bash
<some cbmc-flags> --json-ui | cbmc-convert-output | sed 's/: note:/: path:/g' | csgrep --prune 1

```

##### some cbmc-flags:

```bash
--div-by-zero-check --signed-overflow-check --unsigned-overflow-check --pointer-overflow-check --conversion-check --undefined-shift-check --float-overflow-check --nan-check --unwind 1 --memory-leak-check --pointer-check
```

