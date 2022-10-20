> Good explanation of what file descriptors and redirection is.
> https://catonmat.net/bash-one-liners-explained-part-three

> To verify if you really understand bash redirection: 
> https://unix.stackexchange.com/questions/388519/bash-wait-for-process-in-process-substitution-even-if-command-is-invalid

The second link shows how we can wait for all processes to finish within a subprocess. 

The goal here is to understand both piping and redirection. Going through the answer presented by Stéphane Chazelas:
- both cmd1 and cmd2 are grouped using `{ cmd1 >(cmd2) }`
- after the grouping it redirects `3>&1` for all commands within the grouping. This means that both cmd1 and cmd2 will have their fd3 open to stdout of the command after the grouping.
- fd1 is then redirected to point to fd4 with `>&4`
- Now when piping, because 3 is the new file descriptor that is pointing to stdout, this is the file descriptor that is piped into the next command.
- `| cat` then wait for fd3 to close as this is the process that is writing to stdout.
- around all this, there is another grouping `{ } 4>&1` which indicates that all the processes within that grouping will have their fd4 pointing into this commands stdout, so the outer most command. Because what was written to stdout of those commands was redirected to fd4, we now say that the fd4 of the commands within the grouping now belongs to the fd1 of the outermost command.