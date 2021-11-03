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

### RHEL packages with `%check` section

-----

| verified package name | csmock                              | cbmc   |
| --------------------- | ----------------------------------- | ------ |
| `grep`                | `cbmc-convert-output`  script error | RESULT |

##### When the output is unreadably large:

```bash
<some cbmc-flags> --json-ui | cbmc-convert-output | sed 's/: note:/: path:/g' | csgrep --prune 1

```

RESULT:

Following command: most of cbmc flags, output generate 

```bash
cbmc --div-by-zero-check --signed-overflow-check --unsigned-overflow-check --pointer-overflow-check --conversion-check --undefined-shift-check --float-overflow-check --nan-check --unwind 1 --memory-leak-check --pointer-check ./grep | grep ": FAILURE"
```


Output:

```bash
<builtin-library-setlocale> function setlocale
[setlocale.pointer_dereference.1] line 11 dereference failure: pointer NULL in *locale: FAILURE
[setlocale.pointer_dereference.3] line 11 dereference failure: deallocated dynamic object in *locale: FAILURE
[setlocale.pointer_dereference.4] line 11 dereference failure: dead object in *locale: FAILURE
[setlocale.pointer_dereference.5] line 11 dereference failure: pointer outside object bounds in *locale: FAILURE

[overflow.5] file dfa.c line 129 arithmetic overflow on unsigned - in (((charclass_word)1 << 64 - 1) << 1) - (unsigned long int)1: FAILURE

localeinfo.c function init_localeinfo
[init_localeinfo.overflow.2] line 98 arithmetic overflow on signed to unsigned type conversion in (unsigned char)i: FAILURE
[init_localeinfo.overflow.4] line 102 arithmetic overflow on unsigned to signed type conversion in (signed int)-len: FAILURE
[init_localeinfo.overflow.5] line 102 arithmetic overflow on signed unary minus in -((signed int)-len): FAILURE
[init_localeinfo.overflow.6] line 102 arithmetic overflow on signed type conversion in (signed char)(len <= (unsigned long int)1 ? 1 : -((signed int)-len)): FAILURE
[init_localeinfo.overflow.7] line 103 arithmetic overflow on signed to unsigned type conversion in (unsigned int)wc: FAILURE

mbrtowc.c function rpl_mbrtowc
[rpl_mbrtowc.overflow.1] line 147 arithmetic overflow on signed to unsigned type conversion in (size_t)-2: FAILURE
11 of 52721 failed
```



