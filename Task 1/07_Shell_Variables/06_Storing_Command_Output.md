# Storing Command Output

Using `NAME="$(command)"` is a bit confusing because `$` is used to access the value of a variable, but there is not much that can be done as `NAME="<(command)"` is like putting a file in `""` which does not make much sense. It might have something to do with variable expansion.
Nonetheless, `PWN=$(/challenge/run)` helps solve the challenge.
