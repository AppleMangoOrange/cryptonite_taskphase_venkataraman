# `grep`ping Errors

The terminal prints `stderr` even though it has been redirected to `stdout`. `/challenge/run 2>& 1 | grep pwn.college` is the solution.
