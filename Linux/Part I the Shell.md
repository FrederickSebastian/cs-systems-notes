
## Chapter 4

1. cp
2. mv
3. mkdir
4. rm
5. ln
- Wildcards need to be reviewed

## Chapter 5

1. type
2. which
3. help (前置的)
4. --help (后置的)
5. man (前置的)
6. apropos
7. whatis (前置的)
8. info
9. alias (unalias)

## Chapter 6

1.  \>
2.  \>>
3. cat
4. \|
5. wc : print newline, word, and byte counts for each file 
6. grep
7. head / tail
8. tee

### Chapter 7

1. echo
2. Pathname Expansion
3. Tilde Expansion
4. Arithmetic Expansion
5. Brace Expansion
6. Parameter Expansion
7. Command Substitution
8. Double Quotes: Protect the strings from special characters, but these are exception: `$, \,` \` . That means the parameter expansion, arithmetic expansion and command substitution will be carried out. 
9. Single Quotes: If we need to suppress **all expansion**, we use single quotes.

### Chapter 8 Advanced Keyboard Tricks

1. Modifying Text keyboard shout cut
2. Auto Completion
3. history(combine with less generally)
4. History Expansion: `!` + `line num`

### Chapter 9 Permissions

1. id
2. chmod
3. umask
4. su
5. sudo
6. chown
7. chgrp
8. passwd

### Chapter 10 Processes

1. Putting a process in the background: follow the command with `&` character.
2. Caution: A process in the background can not be terminated by terminal keyboard input. Use `fg` instead.
3. We can control a process by its *jobspec*, which means its `%` + *its number by `jobs`*, like this: 
	`fg %1`
4. Stopping a process means **pausing** a process, without terminating it. Use CTRL + Z to pausing it. 
5. `kill` command is not terminating the process but sending it some *signals*. To kill the process, follow the command with the *PID* , which means the code from `ps`.
6. ps : report a snapshot of current processes
7. top : display tasks
8. jobs : list active jobs
9. bg : place a job in the background
10. fg : place a job in the foreground
11. kill : send a signal to a process
12. killall : kill process by name or user name
13. shutdown