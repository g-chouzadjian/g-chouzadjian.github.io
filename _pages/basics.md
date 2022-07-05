---
title: "Basics"
permalink: /basics/
excerpt: "Basics"
toc: true
---

## Shebang

`#!` is known as a **she** (sharp) **bang** (slang for exclamation point). You wanna start out every shell script with a shebang and then a path to the desired interpreter.

***Example***

```
#!/bin/bash
```
The commands in this script will be interpreted by the bash program.

In reality, when the above line is executed, `/bin/bash` is called and the filename is passed to it.

```
/bin/bash <FILENAME>
```

If you do not provide a shebang for your script then the commands will be executed by your current shell.

***Best Practise***

Always use a shebang in your shell scripts. You never know what shell another person may be using when they run it so to be safe, be explicit. 

## Comments

Use the `#` sign to signify a comment.

***Best Practise***

Comment out the next line after the shebang and give a quick summary of the purpose of your shell script.

## File Permissions

You can use `ls -l` to check the file permissions.

![file permissions](https://trello-attachments.s3.amazonaws.com/5f03f524f8b71559621d5fac/5f01a3966ea0b978bf640480/1ae61a4c264705732928f00a30bdb6f1/perms1.png)

You can adjust file permissions with `chmod`.

***Best Practise***

A good default file permission is `755` (RWX, RX, RX).

## Execution

To execute a shell script in the current directory use the current directory symbol `.` combined with the directory separator `\`:

```
./path/to/file.sh
```

The above command is equivalent to inputing the fully qualified path:

```
/fully/qualified/path/to/file.sh
```

## Builtins

Builtins are commands or functions which are built into the shell itself, they do not require any external programs to execute. Examples of bash builtins are: `echo`, `logout` etc.

To check if the command you are interested in is a builtin you can use the `type` command.

```
type <command>
```

To see the full path of a builtin or any other executable use:

```
type -a <command>
```

Builtins also have access to help via:

```
help <command>
```

unlike external executables which rely on `man`. 

***Best Practise***

Whenever possible, use shell builtins because they are significantly faster than loading/running external programs. It is also more portable to use builtins b/c they do not rely on a system path.

## Pagination

Sometimes when you call help on a command, the resulting text can be overwhelming. To make things easier, pipe the output to the `less` command. 

```
help echo | less
```

This will separate the content into discrete pages. To exit `less`, use `q`.

The `man` command also uses `less`. 

***Vim Tips***

The `vim` keybindings apply to `less` so you can use `/<search-term>` to quickly find what you're looking for. To skip forward through matches use `n`, and `shift+n` to go backwards.

To reverse search use `?<search-term>`.

## Variables

Bash doesn't really have types; variables are character strings. To assign a value to a variable use the assignment operator `=`.

***Example***

```
WORD='Foo'
```

Make sure there are no spaces on either side of the assignment operator. Also, use single quotes to indicate literal interpretation of the character string.

When you need to expand a variable, use the `$` symbol.

***Example***

```
echo $WORD
```

If assigning a variable to another combined with other character strings, use double quotes to indicate variable expansion.

***Example***

```
WORD='foo'
OTHER_WORD="$WORD bar"

# foo bar
```

The alternative `${VARIABLE}` syntax is used to append characters to the variable.

***Example***

```
WORD='foo'
OTHER_WORD="${WORD}bar"

# foobar
```

### Special Variables

You can find special bash variables by viewing the bash man page.

```
man bash
```

`UID` - Expands to the user ID of the current user, initialised at shell startup. This  variable is readonly. The root user will always have UID=0.

***Tip***

You can get user identity information by using the `id` command. For current username:

```
id -un
```

Which is also equivalent to the command `whoami`.

## Command Substitution

To store the result of a command in a variable, use the following syntax:

```
VARIABLE=$(<list_of_commands>)

# Alternate syntax (Old)

VARIABLE=`<list_of_commands>`

```

***Example***

```
USER_NAME=$(id -un)
```

## If Statements

The syntax for if statements is as follows:

```
if [[ <expression> ]]
then
  <expression>
else
  <expression>
fi
```

***Example***

```
if [[ "${UID}" -eq 0 ]]
then
  echo 'You are root."
else
  echo 'You are not root.'
fi
```

###  Command Separator

The `;` is the command separator symbol and can be used interchangeably with a newline.

### [[

The `[[` symbol is used to evaluate conditional expressions of the kind `[[ expression ]]`. It exits with a value of `0` for true and `1` for false. For a list of valid operators use the `test` builtin.

```
help test
```

***Tip***
The `[` is analogous to `[[` but is the old syntax.

### String operators

***Example***

```
USER_NAME=$(id -un)
USER_NAME_TO_TEST_FOR='vagrant'

if [[ "${USER_NAME}" = "${USER_NAME_TO_TEST_FOR}" ]]
then
  echo "Your username is equal to ${USER_NAME_TO_TEST_FOR}"
fi
```

***Note***

The `=` operator has different meaning in different contexts. As a string operator used within an expression, it is used to test for equality. In other contexts it is used as an assignment operator.

 ***Tip***

To perform pattern matching use the `==` operator. 

### Exit Status

You can exit a script at any point with the `exit` command. By convention when a script executes successfully, it exits with a status code of `0`. To indicate an error we choose a non-zero exit status.

You can check the error codes of certain programs by seeing the **man page**. 

### Special Parameters

`?` - This parameter expands to the exit status of the most recently executed script.

***Example***

```
#Assigns username to variable
USER_NAME=$(id -un)

if [[ "${?}" -eq 0 ]]
then
  echo 'The id command did not execute successfully.'
  exit 1
fi
```

To see more special parameters, see the bash `man` page and search `/Special Parameters` 

## Stdin, Stdout and Stderr

### `read` builtin

By default, standard input comes from the keyboard. It can come from other locations, such as from a pipeline.

The `read` builtin is used to capture **standard input**. 

***Example***

```
read -p 'Type something: ' THING
```

The above command prompts the user to 'Type something: ' and then assigns the standard input to the variable `THING`.

***Tip***

When using variables set from stdinput, use the variable expansion form in subsequent commands in case they contain spaces.

```
read  -p 'Enter the name of the person this account is for: ' COMMENT 

useradd  -c "${COMMENT}"
```

Standard output and Standard Error are displayed to the screen.

## Pipelining

When you use pipes in commands, you're saying you want the stdout of one command to be the stdin to the next command. If the first command has an error on  output, that does not get sent to the stdoutput, it goes to  stderr  and does not get fed to the next command.

## Users

### `useradd`

This command can be used to create a new user. Note the term LOGIN and USERNAME are interchangeable in linux parlance.

Typically, usernames are 8 character's or less by convention. They are also lowercase.

#### Useful Options

The `-m` option creates the users home directory if it doesn't already exist.

The `-c` option allows you to write a comment for that particular LOGIN.

### `su`

This command is used to **switch user**.

***Note***

The `-` option to the `su` command, tells it to start with an environment similar to that of a real login. Without the `-`, the new user will remain in the previous user's environment (old user's home folder, path to executables etc.) [Read more](https://www.tecmint.com/difference-between-su-and-su-commands-in-linux/)

***Example***

```
# Depending on your current account, you may need to use `sudo`.
# Assuming root user for this example.

useradd foobar

su - foobar

```

### `passwd`

This command sets the password for a particular user. 

***Tip***
You cannot change the password of another users account unless you are the root user.

The default mode of `passwd` is interactive. Therefore when you enter it, it will prompt you for input. 

#### Useful options

`--stdin`: gets input from stdin.

`-e`: Expires the password for the given user. They will be forced to change it on next logn.

***Example***

```
# Sets the password for the user via stdin

echo ${PASSWORD} | passwd --stdin ${USER_NAME}
```

***Example***

```
# Expires the password for the given user

passwd -e ${USER_NAME}
```



`ps -ef`

## Source vs. Execute

See [link](https://superuser.com/questions/176783/what-is-the-difference-between-executing-a-bash-script-vs-sourcing-it#:~:text=7%20Answers&text=Sourcing%20a%20script%20will%20run,in%20your%20currently%20running%20shell.)

`Sourcing` a script will run the commands in the current shell process.

`Executing` a script will run the commands in a new shell process.

Use source if you want the script to change the environment in your currently running shell. use execute otherwise.

## Useful Binaries

`dirname` - Print NAME with its trailing /component removed; if NAME contains no /'s, output '.' (meaning the current directory).

*Example*

...

`chmod` - Change file mode bits. Used to change the read, write, execute permissions of a file for owner, group and others.

[strict mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/)

[Command line argument parsing](https://sookocheff.com/post/bash/parsing-bash-script-arguments-with-shopts/)