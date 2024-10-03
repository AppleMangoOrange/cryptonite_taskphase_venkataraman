# Exporting Variables

`export` seems to make the variable "global" (in a sense) for the current shell session, i.e. the export is "lost" and the variable loses its value after the terminal is closed. Some digging reveals that wrapping the `export` command in `()` as such `(export NAME && some-command)` only allows that command to access that variable. This can be tested by
```
:~$ (export TEST=VALUE && echo $TEST)
VALUE
:~$ echo $TEST


```
