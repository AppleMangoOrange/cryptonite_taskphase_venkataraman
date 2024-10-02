# Split-piping `stderr` and `stdout`

There is no need to merge the `stderr` and `stdout` channels, they can both be redirected separately without using the pipe operator.
Since using Process Substitution converts program into files, we can separately pipe the `stdout` and `stderr` of `/challenge/hack` to the required programs as such:
`/challenge/hack 1> >(/challenge/planet) 2> >(/challenge/the)`
