# Backgrounding Processes

Background processes dont occupy the terminal but keep running, which is signified by `R` or `S` in `ps`. The foreground `+` symbol can be seen by running `sleep` in one terminal and `ps aux` in another. The `sleep` command has the `STAT` `SN+`, where the `+` shows that it's running in the foreground (of the first terminal).
