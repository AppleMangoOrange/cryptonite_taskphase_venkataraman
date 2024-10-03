# Killing Processes

The process ID for `/challenge/dont_run` can be viewed using `ps aux`, which is 73 in this case. Then `/challenge/run` can give the flag. With no arguments, `kill` stops the program "gracefully". The command can send any signal to any process, the default one just happends to ask the program to exit.
