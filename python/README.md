# Python tools

# structviz: Visualize structures in text files

*Description:*

A tool  for visualizing nested structures in text files such as
nested ifdefs (``#if ... #if ... #else #endif #endif``).

*Usage:*

```
usage: structviz [-h] [-o,--opening OPENING] [-c,--closing CLOSING]
                 [-t,--intermediate INTERMEDIATE]
                 [-i,--indent-string INDENTSTR] [-s,--case-sensitive]
                 [-n,--show-line-numbers] [-l,--show-file]
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
  -l,--show-file        Show fie name [default=False]
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
structviz /usr/include/stdlib.h -l -n -t "extern"
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
