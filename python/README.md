# Python tools

## MULTISUB: Use *grep-output-lik*e input to replace patterns in text files 

*Description:*

`multisub` takes an *grep-output-like* input (file), which contains lines in the format:

```shell
<filepath><separator><lineno><optional text>
```
(e.g. obtained via `grep -H -n`) to specify lines in files
where it `replaces` specified search patterns with
other expressions.

*Tips:*

* Use the `-H -n` switches to output (relative) filepath and line numbers
with `grep` and `structviz`
* Chain `grep` multiple times to filter out certain lines and line ranges
* Use the `-w <expression>` switch to embed a targeted line within an `C` preprocessor construct:
```
#if<expression>
<modified line>
#else
<original line>
#endif
```
All further text replacements will be applied to the modified line within
the `#if<expression>` branch.

*Usage:*

```shell
usage: multisub [-h] [-p,--patterns PATTERNS [PATTERNS ...]]
                [-s,--search SEARCHPATTERNS [SEARCHPATTERNS ...]]
                [-d,--delete-patterns DELETEPATTERNS [DELETEPATTERNS ...]]
                [-e,--erase-lines ERASELINEPATTERNS [ERASELINEPATTERNS ...]]
                [-I,--inplace] [-a,--write-input-lines] [--min MIN]
                [--max MAX] [--separator [SEPARATOR]] [-v,--verbose]
                [-n,--show-only-changes] [--show-individual-changes] [--nop]
                [-w,--wrap-in-ifelse [WRAPINIF]] [-i,--case-insensitive]
                [-l,--lower] [-u,--upper] [-C,--camel-case] [-U,--underscores]
                [input]

multisub - Perform regex replacement operations based on grep-output-like
input files

positional arguments:
  input                 File/input stream containing list of lines with
                        pattern: <filepath><separator><lineno>[optional rest
                        of line], typically obtained via grep (or structviz
                        tool).

optional arguments:
  -h, --help            show this help message and exit
  -p,--patterns PATTERNS [PATTERNS ...]
                        Search patterns (first, third, ...) and their
                        replacement (second, fourth, ...). List must have even
                        length.
  -s,--search SEARCHPATTERNS [SEARCHPATTERNS ...]
                        Specify only search patterns.
  -d,--delete-patterns DELETEPATTERNS [DELETEPATTERNS ...]
                        Patterns to delete
  -e,--erase-lines ERASELINEPATTERNS [ERASELINEPATTERNS ...]
                        Erase whole lines that contain one of the specified
                        patterns.
  -I,--inplace          Apply replacement action directly to original file(s)
                        [default=False]
  -a,--write-input-lines
                        Write lines as specified in the input file to the
                        destination file(s). Has precedence over other
                        modifications.
  --min MIN             Minimum line number (inclusive) to consider
                        [default=-1]
  --max MAX             Maximum line number (inclusive) to consider
                        [default=sys.maxsize]
  --separator [SEPARATOR]
                        Characters regarded as separator between filepath and
                        lineno [default=':-']
  -v,--verbose          Verbose output
  -n,--show-only-changes
                        Dry run; show only changes do not generate any output
                        (to stdout or orginal file)
  --show-individual-changes
                        Show individual changes; -n (--show-only-changes) must
                        be specified too.
  --nop                 Do nothing at all.

Post-process replacements/search results:
  -w,--wrap-in-ifelse [WRAPINIF]
                        Wrap modified and original line in '#if<expression>
                        <modified> #else <original> #endif' construct
  -i,--case-insensitive
                        Ignore case when searching for patterns
                        [default=False]
  -l,--lower            Convert ALL replacements/search results to lower case
  -u,--upper            Convert ALL replacements/search results to upper case
  -C,--camel-case       Convert ALL replacements/search results to 'camel
                        case'.
  -U,--underscores      Convert ALL replacements/search results to 'underscore
                        case'.

```

These options are displayed when calling `multisub` with `--help` or `-h`.

```shell
multisub --help
```

### Example

In this example, we will convert the Fortran subroutine to C:


```Fortran
! example.f90
subroutine mysubroutine(a,b)
  integer :: a = 0
  INTEGER :: b = 0
end subroutine
```

1. We first select the lines (here, we find all file starts (`^`) with grep):

*Input 1:*

```shell
grep -H -n "^" example.f90
```

*Output 1:*

```
example.f90:1:! example.f90
example.f90:2:subroutine mysubroutine(a,b)
example.f90:3:  integer :: a = 0
example.f90:4:  INTEGER :: b = 0
example.f90:5:end subroutine
```

2. Now "convert" this snippet by performing the following replace operations:

* `end subroutine` by `}`
* `^subroutine` by `void`
* `(a,b)` by `(int a, int b)`
* `integer :: <identifier> = 0` by `<identifier> = 0;` (search ignores case by default)
* `!` by `//`

```shell
grep -H -n "^" example.f90 -H -n |\
  multisub -p "end subroutine" "}" "^subroutine" "void" "\(a,b\)" "(int a, int b) {" "integer :: (a|b) = 0" "\1 = 0;" "\!" "//"
```

*Output 2:*

```C
// example.f90
void mysubroutine(int a, int b) {
  a = 0;
  b = 0;
}
```

3. You can also display the changes that would be made by adding the `-n` switch:

*Output 3:*

```shell
example.f90:1:
-! example.f90
+// example.f90
example.f90:2:
-subroutine mysubroutine(a,b)
+void mysubroutine(int a, int b) {
example.f90:3:
-  integer :: a = 0
+  a = 0;
example.f90:4:
-  INTEGER :: b = 0
+  INTEGER :: b = 0
example.f90:5:
-end subroutine
+}
```

Lines starting with '-' and '+' indicate text that is 
removed or added, respectively.

4. To perform all changes in-place, use the `-I` switch (and remove the `-n` switch). 

## STRUCTVIZ: Visualize structures in text files

*Description:*

A tool  for visualizing nested structures in text files such as
nested ifdefs (``#if ... #if ... #else #endif #endif``).

*Tips:*

Pre- and postprocess input output with standard unix tools such as `grep` or `sed`.

*Usage:*

```
usage: structviz [-h] [-o,--opening OPENING] [-c,--closing CLOSING]
                 [-t,--intermediate INTERMEDIATE]
                 [-i,--indent-string INDENTSTR] [-s,--case-sensitive]
                 [-n,--show-line-numbers] [-H,--show-file]
                 [input]

structviz - Visualize structures in text files

positional arguments:
  input                 The input file.

optional arguments:
  -h, --help            show this help message and exit
  -o,--opening OPENING  Opening pattern, e.g. '#if' [default] or '\{'
  -c,--closing CLOSING  Ending pattern, e.g. '#endif' [default]
  -t,--intermediate INTERMEDIATE
                        Other patterns we want to match [default=None]
  -i,--indent-string INDENTSTR
                        A string that is added as indent per level of the
                        nested structure, e.g. two whitespaces [default]
  -s,--case-sensitive   Do not ignore case when search for patterns
                        [default=False]
  -n,--show-line-numbers
                        Show number of matched lines [default=False]
  -H,--show-file        Show fie name [default=False]
```

These options are displayed when calling `structviz` with `--help` or `-h`.

```shell
structviz --help
```

### Example

```shell
structviz /usr/include/stdlib.h
```

*Output 1:*

```shell
ifndef _STDLIB_H
  #ifndef __need_malloc_and_calloc
  #ifndef __need_malloc_and_calloc
    #if (defined __USE_XOPEN || defined __USE_XOPEN2K8) && !defined _SYS_WAIT_H
    #ifndef __ldiv_t_defined
    #if defined __USE_ISOC99 && !defined __lldiv_t_defined
    #if defined __USE_ISOC99 || (defined __GLIBC_HAVE_LONG_LONG && defined __USE_MISC)
    #ifdef      __USE_ISOC99
    #if defined __GLIBC_HAVE_LONG_LONG && defined __USE_BSD
    #if defined __USE_ISOC99 || (defined __GLIBC_HAVE_LONG_LONG && defined __USE_MISC)
    #ifdef __USE_GNU
    #ifdef __USE_EXTERN_INLINES
    #if defined __USE_SVID || defined __USE_XOPEN_EXTENDED
    #if defined __USE_SVID || defined __USE_XOPEN_EXTENDED || defined __USE_BSD
    #ifdef __USE_POSIX
    #if defined __USE_SVID || defined __USE_XOPEN
  #ifndef __malloc_and_calloc_defined
```

Add line  numbers and file names and additionally look for non-group expression 'extern':

```shell
structviz /usr/include/stdlib.h -H -n -t "extern"
```

*Output 2:*

```
/usr/include/stdlib.h  22:#ifndef       _STDLIB_H
/usr/include/stdlib.h  28:  #ifndef __need_malloc_and_calloc
/usr/include/stdlib.h  36:  #ifndef __need_malloc_and_calloc
/usr/include/stdlib.h  39:    #if (defined __USE_XOPEN || defined __USE_XOPEN2K8) && !defined _SYS_WAIT_H
/usr/include/stdlib.h 104:    #ifndef __ldiv_t_defined
/usr/include/stdlib.h 114:    #if defined __USE_ISOC99 && !defined __lldiv_t_defined
/usr/include/stdlib.h 139:    extern size_t __ctype_get_mb_cur_max (void) __THROW __wur;
/usr/include/stdlib.h 144:    extern double atof (const char *__nptr)
/usr/include/stdlib.h 147:    extern int atoi (const char *__nptr)
/usr/include/stdlib.h 150:    extern long int atol (const char *__nptr)
/usr/include/stdlib.h 154:    #if defined __USE_ISOC99 || (defined __GLIBC_HAVE_LONG_LONG && defined __USE_MISC)
(more lines)
```
