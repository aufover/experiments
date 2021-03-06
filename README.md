### About project (and how it works)

[Automation of Formal Verification (AUFOVER) - slides](https://kdudka.fedorapeople.org/kdudka-aufover-211111.pdf)

##### Presented at DevConf 2021 (in the context of dynamic analysis):

* [slides](https://kdudka.fedorapeople.org/kdudka-devconf-21.pdf)
* [video](https://www.youtube.com/watch?v=FjV84hbD1GY)
* [demo](https://github.com/csutils/cswrap/wiki/csexec)

### How to conduct experiments

#### With [`csmock`](https://github.com/csutils/csmock) with `csmock plug-ins`
Formal verification of RPM packages with CBMC, Divine, and Symbiotic is available in stable Fedora releases.

The csmock plug-ins for these tools are still experimental and they have
some technical limitations:

- They work only for source RPM packages that contain the %check section
  that directly or indirectly invokes the binaries produced in the %build
  section.

- The tools are known to work reliably only for programs written in C.

The plug-ins can be installed on a Fedora system using the following command:

    $ sudo dnf install csmock-plugin-{cbmc,divine,symbiotic}

Then you can formally verify RPM packages of your choice:

##### Download the package for verifiaction:

    $ dnf download --source <package> # get <package>.src.rpm
or

    $ koji download-build src <package> # get <package>.src.rpm  
##### Then run verification with `csmock`:
In simple:
    
    $ csmock -t ${tool} ${pkg-to-verify}.src.rpm
or more specific way:

    $ csmock -t ${tool} --${tool}-add-flag='${tool-flags}' --${tool}-timeout=${number} ${pkg-to-verify}.src.rpm
    
##### Get your results:

    $ tar -xvf ./${pkg-to-verify}.tar.xz '*/${tool}-capture/'

##### copy-paste example:
```bash
$ dnf install csmock-plugin-cbmc cbmc -y # formal_verification_tool=cbmc
$ dnf download --source logrotate # pkg_to_verify=logrotate
$ csmock -t cbmc logrotate-3.18.1-2.fc35.src.rpm
$ tar -xvf ./logrotate-3.18.1-2.fc35.tar.xz '*/cbmc-capture/'
```
#### *With* your favourite verification tool

Tools list (formal verification only):

* [cbmc](https://github.com/diffblue/cbmc)           [EXPERIMENTAL] Bounded Model Checker for C and C++ programs
* [divine](https://divine.fi.muni.cz/)               [EXPERIMENTAL] A formal verification tool based on explicit-state model checking.
* [symbiotic](http://staticafi.github.io/symbiotic/) [EXPERIMENTAL] A formal verification tool based on instrumentation, program slicing and KLEE.

More tools (`csmock --list-available-tools`): `clang`, `cppcheck`, `gcc`, `shellcheck`, `strace`, `valgrind`, ...

##### `TIMEOUT` hack

This is useful when you __do not__ want to run all tests in the `%check` section - guess your timeout number and the process (`rpmbuild -bi`) will hopefully stop where you want!

* Change [line](https://github.com/csutils/csmock/blob/56274586eec3268dbbd3719a2da908d6b0075b7d/py/csmock#L296) in `/bin/csmock` like this:
  ```diff
           # construct basic `rpmbuild -bi ...` command
           rpm_opts = props.rpm_opts + extra_rpm_opts
  -        cmd = "rpmbuild -bi --nodeps --short-circuit %s %s" \
  +        cmd = "timeout <lucky_number> rpmbuild -bi --nodeps --short-circuit %s %s" \
               % (props.spec_in, strlist_to_shell_cmd(rpm_opts))
  ```

* Thanks to `--no-clean` option, run the following command, where `${fedora-version-arch}` is the environment where `csmock ...` command ran previously
  ```bash
  mock --root ${fedora-version-arch} --shell
  ```

##### How to add (install) some package to `mock`?
Maybe `vim`, if you are planning to experiment manually:
```bash
$ mock --root ${fedora-version-arch} --install ${package} # e.g. fedora-version-arch = fedora-34-x86_64, package = vim
```

#### [CBMC](https://src.fedoraproject.org/rpms/cbmc) (tips and tricks):

##### Useful `csmock` options when you want to play inside the chosen build root:

`--keep-going` *continue as much as possible after an error* - error can be for example `timeout`

`--no-clean` *do not clean chroot when it becomes unused* - this option is important if you'd like to run the verification by yourself

##### When the output is unreadably large:

```bash
{some cbmc-flags} --json-ui | cbmc-convert-output | sed 's/: note:/: path:/g' | csgrep --prune 1
```
where `{some cbmc-flags}` is at least one of the following flags
```bash
--div-by-zero-check --signed-overflow-check --unsigned-overflow-check --pointer-overflow-check --conversion-check --undefined-shift-check --float-overflow-check --nan-check --memory-leak-check --pointer-check
```
