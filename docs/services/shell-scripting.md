Shells provide an interface for controlling programs and their input and output.

There are 4 different shells

* Bourne (sh)
* C (csh)
* Korn (ksh)
* Bash (Bourne-Again)

The Bourne shells are ideal for string and word manipulation where as C shell is best used for mathematical scripting. Korn shell was an attempt at merging these two shells at a point but was later dropped and Bash was created.

All of the above shells are line-oriented, with features dating back to the earliest UNIX shells for Teletype terminals. All shells operate in a loop:

#Display a prompt
# Ready input command line
# Perform substitutions in command line
# Execute command
# Loop to step 1

The substitutions in step 3 may be of : variables, filenames, command outputs, aliases, depending on the shell. For the following documentation we will be dealing with primarily Bash shell, the currently most common shell on Linux distributions

==Command Substitution==

Many commands that take arguments ignore standard input (eg. ls, file, echo, cd, rm). This is a problem for passing around results between commands

A conundrum: ls prints the list of file names to standard output, file though takes filenames as arguments and determines the file type. So how do we get the output of ls as an input for file ?

This won't work:
 <nowiki>
ls | file</nowiki>

So how do we do this ? When the shell encounters a string between backquotes such as <code> `cmd` </code> it executes cmd and replaces the backquotes string with the standard output of cmd, with any trailing newlines deleted. So for out ls and file problem we can solve it like this:
 <nowiki>
file `ls`</nowiki>
In this scenario ls will be executed first, which will then be passed as parameters to the file command, getting file information about the current directory

Standard output of any command can be made into arguments to any other command like this:
 <nowiki>
$ date
#will print: Thu May 10 11:33:46 PDT 2012

$ echo The date is `date`
#will print: The date is Thu May 10 11:34:33 PDT 2012

$ echo The date is `data| awk '{print $2, $3 ",", $6}'`
#will print: The data is May 10, 2012</nowiki>

==Defining Variables==
Shell variables can hold single string values, created with = and accessed $. NOTE they are ONLY string variables. Also spacing matters here. There can be not spaces between the variable name the '=' sign and the value being assigned.

Variable concatenation is also possible using a special syntax. You can concatenate one variable into the other by surrounding the one variable in {}

<b>Examples</b>
 <nowiki>
$ ub=/usr/bin
$ cd $ub;pwd</nowiki>
This example assigned the <code>/usr/bin</code> folder to the variable ub and then cd's into the variable and then asks for the current directory. Note here that we are assigning variables directly in the console. You can create and use variable both in script files and in the console. You variables will only last in console until the shell terminal is closed.

Note in the above aswell the commands are seperated with a semi-colon. This allows you to make multiple sequential commands on the same line / same command execution

 <nowiki>
$ curse='@$*&>##! is a legalized virus!'
$ echo $curse
@$*&>##! is a legalized virus!</nowiki>
The above example shows how to include whitespace or matecharacters into assignments. Note that the difference is surrounding the string with single quotes

 <nowiki>
$ dir=`pwd`</nowiki>
Using the techniques in the previous section here we can store the output of a command into a variable

 <nowiki>
$ one=1

$ echo $onetwo  #does not work - variable does not exist

$ echo ${one}two #value of variable one, text two
# will print: 1two</nowiki>

==Metacharacter Suppression==
With shell using so many characters with secondary special meanings (eg /, $,`, \) what do you do if you want literaly these items in your strings ? Using Metacharacter Suppression you can cancel out these second meanings and use these characters for thier literal use

The following rules work for character suppression:
* Single quotes suppress the special meaning of ALL metacharacters
* Double quotes suppress the special meaning of all metacharacters EXCEPT \ (backslash), $ (dollar sign) and ` (backquote)
* Backslash suppresses the special meaning of the immediately following character (except between single quotes - because then it aswell is cancelled out)

Basically you can think of it as the same rules to PHP. Single quotes mean whatever is between them will be taken literally. Double quotes will attempt to resolve any variables or special meaning characters it finds within them, and backslash will escape the character, unless that creates a meaning within itself

Note that >, <, |, spaces and tabs are also metacharacters as they seperate arguments.

<b>Examples</b>
 <nowiki>
$ echo hello | wc -c         # one argument to echo
# will print: 6

$ echo hello '|' wc -c       # four arguments to echo
# will print: hello | wc- c

$ echo '.....hello | wc -c'  # one argument to echo
# will print: .....hello | wc -c</nowiki>

Patterns for grep and sed often contain metacharactersl its a good habit to surround arguments with single quotes
 <nowiki>
$ grep '|' chap2

$ sed 's?>/>>/g' foo/bar/readme</nowiki>

==Shell Functions==
A shell function associates a name with a list fo commands. When the name is given as a command, the list of commands is executes. All functions must be declared before they can be used. The syntax is:
 <nowiki>
function_name(){
    Commands
}

OR

function_name(){
    Commands
    return $TRUE
}</nowiki>

<b>Example</b>
 <nowiki>
$ mydate() { date | awk '{ print $2, $3 ",", $6 }'; }
$ mydate
#will print: May 10, 2012</nowiki>

==Function Arguments==
You can't pass arguments to shell functions. The actual () doesn't mean anything other then help shell identify a function. You can actualy put any parameters in it. You can pass parameters though through positional parameters using <code> $1, $2, .... , $9 </code> like syntax.

<b>Example</b>
 <nowiki>
nd(){
    cd $1
    PS1="`pwd` $ "
}

#calling the function in shell
$ nd /bin
# will print: /bin $</nowiki>

==Positional Parameters and Special Variables==
Arguments to a shell script are accessed by position parameters. There are some additional ways to access these parameters

* $1, $2, ...., $9   -   Arguments 1 through 9
* $0                 -   Will be the script file's name
* $*                 -   All Positional paramters (starting from 1)

Some other special variables include
* $#                 -   Number of positional parameters to the script
* $$                 -   Shell prcess ID number 
* &?                 -   Exit status of last command

==Shell Scripts==
A sequence of commands stored in an ordinaty text file is called a shell script. The .(dot) build-in causes the shell to execute commands from a script

<b>Example</b>
 <nowiki>
$ cat f_defs
echo Defining nd and mydate
nd() { cd $1; PS1="`pwd` $ "; }
mydate() { date | awk ' { print $2, $3 ",", $6 }'; }


$ . f-defs
Defining nd and mydate
$ nd /bin
/bin $</nowiki>

Alternatively, shell scripts can be executed by setting the execute bit on the file

==The awk Programming Language==
awk is named after is authors: Al Aho, Peter Weinberger and Brian Kernighan.

It is a full programming language supporting:
* build-in and user-defined variables
* arithmetic and string operations
* loops and branches
* build-and user-defined functions

awk's simplest and most common use is to select and change the order of fields. awk teats each input line as a sequence of fields delimited with whitespace. Fields can be selected by number and printed:
 <nowiki>
$ awk '{ print $3 }'
UNIX is Linux
Linux
Linux is UNIX
UNIX
^d</nowiki>

Using single quotes makes the print action test a single argument to awk, with no metacharacter interpretation

awk can be used to reformat output of other commands:
 <nowiki>
$ date
# will print: Thu May 10 13:28:57 PDT 2012

$ date | awk '{ print $2 $3 $6 }'
# will print: May102012

$ date | awk '{ print $2, $3, $6 }'
# will print: May 10 2012

$ date | awk '{ print $2, $3, ",", $6 }'
# will print: May 10 , 2012</nowiki>

 <nowiki>
$ echo ipsos custodies? | awk '{ print "Quis custodiet ipsos " $2 }'
Quis custodiet ipsos custodies?</nowiki>

awk's field seperator can be changed with the -F options. For example <code> -F: </code> will make colon the identified field seperator. Then we could use this  to print out all of the login names of all users in the passwd file on linuz as follows:

 <nowiki>
$ awk -F: '{ print $1 }' /etc/passwd
# prints list of all users

$ grep /home /etc/passwd | awk -F: '{ print $1 }'
# prints all users who have /home as part of thier login directory (essentialy finding all actual users)</nowiki>

==shift and set Build-In Commands==
The shift command shifts position parameters

 <nowiki>
# build shift_test script
$ cat shift_test
echo $0 $1 $5 $9 " Args:" $*, No: $#
shift
echo $0 $1 $5 $9 " Args:" $*, No: $#

# use shift_test script
$ shift_test A B C D E F G H I J
shift_test A E I Args: A B C D E F G H I J, No: 10
shift_test B F J Args: B C D E F G H I J, No: 9</nowiki>

Note that by shifting it is truncating off the first letter, like JavaScript arra's shift function. Note that the indexs though have not changed, only the value they point to has

The set command sets the positional parameters to its arguments. Essentially it overwrites the positional parameters with the returned values from whatever command is passed to set:

 <nowiki>
# build set_test script
$ cat set_test
date
set `date`
echo The date is $2 $3, $6

# execute set_test script
$ set_test
Thu May 10 13:42:46 PDT 2012
The date is May 10, 2012</nowiki>

==The for Loop==

General Form:
 <nowiki>
for var [in val_list]
do
    command1
    command2
    commandN
done</nowiki>

For example to display a string 10 times:
 <nowiki>
#!/bin/bash
for i in 1 2 3 4 5
do
echo "Hello $i World"
done</nowiki>

Shell also allows you to set a step value, allowing you to count by 2 or 3 or even backwards. The following example has default setp value of 1
 <nowiki>
#!/bin/bash
for i in {1..5}
do
echo "Hello $i World"
done</nowiki>

We can also specifty step arguments using the <code> {START...END...INCREMENT} </code> syntax:
 <nowiki>
# Only works for Version 4.0+
#!/bin/bash
echo "Bash version ${BASH_VERSION}..."
for i in {0..10..2}
do
echo "Hello $i World"
done</nowiki>
See in this example it will start at 0, end at 10 and increment by 2 as specified in the <code>{0..10..2}</code> of the for loop of the above example

If no value list is given, $* (the positional parameter list) is assumed:
 <nowiki>
for varname
do
echo $varname; echo “${varname}: printed”
done</nowiki>

Output:
 <nowiki>
# sh varprint.sh foo bar foobar
foo
foo: printed
bar
bar: printed
foobar
foobar: printed</nowiki>

==The case Branch==
Simply, this is the shell equivelent of Case or a Switch statement

General form:
 <nowiki>
case word in
pattern) commands ;;
esac</nowiki>
Note: the double semi-colons are NOT a typo. Nor is the single ')' closing bracket

In the above example 'word' is usual a variable substitution. It uses filename expansion-like metacharacters with | for "OR"

A case branch can be the body of a for loop, for example:
 <nowiki>
for i
do
    case ${i} in
        win[0-7])    echo "${i}: Windows" ;;
        linux|unix)  echo "${i}: Operating System" ;;
        *)           echo "${i}: Unknown" ;;
    esac
done</nowiki>

Output:
 <nowiki>
# sh word_names.sh win0 unix mac
win0: Windows
unix: Operating System
mac: Unknown</nowiki>

==The if/else Branch==
General Form
 <nowiki>
if commands
then 
    commands

    #else if condition
    [elif commands 
     then 
        commands]

    # else condition
    [else 
        commands]
fi</nowiki>
Note all of the else if and else commands are inside the if command which is closed with <code>fi</code>

The commands following if are executed. An exit status of 0 from the last command means true, non-zero means false. The if construct is typically used in conjunction with the test command which is covered in detail in the next section

==The test Command==
The test command evaluates an expression and yield an exit code of 0 for true, and non-0 for false

General form:
 <nowiki>
test expression

OR 

[expression]</nowiki>

There is also a new form used on Bash and Korn shells referred to as "new test" and use double brackeds ( [[expression]] ) instead of single given in the above example

test implements the old, portable syntax of the command. In almost all modtern shells '[' is a synonym for test (but requires a final argument of ']')

'[[' is a new improved version of it, and is a keyword, not a program. This makes it easier to use, as shown in the examples below:

<b>Examples</b>
 <nowiki>
if [ -z "$variable" ]
then
    echo "variable is null!"
fi

if [ ! -f "$filename" ]
then
    echo "not a valid, existing filename: $filename"
fi

----------------------------------------------------------------------------------------------------------------------

if [[ ! -e $file ]]
then
    echo "directory entry does not exist: $file"
fi

if [[ $file0 -nt $file1 ]]
then
    echo "file $file0 is newer than $file1"
fi</nowiki>

In the case of [[ glob expansion (pattern matching) will be done and therefor arguments need not be quotes. Because [[ is built in on to the shell and does not have legacy requirements, you don't need to worry about word splitting on variables that evaluate to a string with spaces. Therefor, there is not need to put the variables in double quotes.

Test has numerous primitives for determining features of files and strings

File Comparisons
{| class="wikitable"
! Syntax
! Use
|-
| -f <file> || True if the file exits, is regular file
|-
| -d <file> || True if the file exists and is a directory
|-
| -x <file> || true if the file exists and is executable
|-
| -s <file> || True if the file exists and has a file size large then 0
|-
|}

String Comparisons
{| class="wikitable"
! Syntax
! Use
|-
| s1 = s2 || True if String 1 (s1) is identicle to String 2 (s2)
|-
| s1 != s2 || True if String 1 (s1) is not identicle to String 2 (s2)
|-
|}

Integer Comparisons
{| class="wikitable"
! Syntax
! Use
|-
| n1 -eq n2 || True if Integer 1 (n1) is equal to Integer 2 (n2)
|-
| n1 -ne n2 || True if Integer 1 (n1) is not equal to Integer 2 (n2)
|-
| n1 -gt n2 || True if Integer 1 (n1) is greater then Integer 2 (n2)
|- 
| n1 -ge n2 || True if Integer 1 (n1) is greater then or equal to Integer 2 (n2)
|-
| n1 -lt n2 || True if Integer 1 (n1) is less then Integer 2 (n2)
|-
| n1 -le n2 || True if Integer 1 (n1) is less then or equal to Integer 2 (n2)
|}

Logical Operators
{| class="wikitable"
! Syntax
! Use
|-
| thing1 -a thing2 || True if thing1 AND thing2 both return True
|-
| thing1 -o thing2 || True if thing1 OR thing2 return True
|-
| thing1 ! <another-param> thing2 || Inverses the meaning of the <another-param>. Is logical equivelent for NOT
|}

There are also short-circuited Logical Operators. These can be used with test but order matters. These operators can also be used outside of if statements as short-hand of executing commands based on the short-circuited logic aswell

Short-Circuited Logical Operators
{| class="wikitable"
! Syntax
! USe
|-
| command1 && command2 || True if both commands return True (exit status 0). If command1 does not return exit status 0, command2 will not be run
|-
| command1 || command2 || True if both commands return True (exit status 0). If command1 returns with exit status 0 (true), command2 will not be run
|}

A good reference for all of the operator options is here : http://tldp.org/LDP/abs/html/comparison-ops.html

<b>Combination Examples</b>
 <nowiki>
#example with AND
test -f temp -a -x temp

#example with OR
[ -if temp -o -d temp]

#example with NOT
test ! -d temp

#example with groupings. NOTE WHITESPACING
[ ${i} = 'ready' -a \( -f temp -o -d temp -o -d tempt \) ]</nowiki>

<b> Examples</b>

Simple example using the if and test commands
 <nowiki>
if [ -f ${i} ]
then
    echo ${i} is a regular file
    else
        echo ${i} is not a regular file
fi</nowiki>

The next example shows a complete shell program that illustrates the use of "if", "test" and shell functions:
 <nowiki>
#!/bin/bash
usage(){
    echo "Usage: $0 username"
    exit 1
}

userlogged(){
    if (who | grep $1 > /dev/null)
    then 
        echo $1 is logged on
        else 
           echo $1 is not logged on
    fi
}

# call usage() function if filename not supplied
if test $# != 1
then 
    usage
    else 
        userlogged $1
fi</nowiki>

Using short-circuited logical operators we can simulate if and else statements. See the if else statement below:
 <nowiki>
if [ -f unixfile ]
then
    rm unixfile
    else
        echo "unixfile was not found, or is not a regular file"
fi</nowiki>

We can simulate this with short-circuited logical operators as:
 <nowiki>
[ -f unixfile ] && rm unixfile || echo "unixfile was not found, or is not a regular
file"</nowiki>

In addition, multiple commands can be executed based on the results of 1 command by incorporating braces and semi-colons:
 <nowiki>
command1 && { command2 ; command3 ; command4 ; }</nowiki>
If the exit status of command1 is true (zero), commands 2,3 and 4 will be performed

==while and until Loops==

General sytnax for the while loop is:
 <nowiki>
while commands
do
    commands
done</nowiki>

<b>Example</b>
 <nowiki>
while true
do
    sleep 60
    who
done</nowiki>

The General syntax for the until loop is:
 <nowiki>
until commands
do
    commands
done</nowiki>

<b>Example</b>
 <nowiki>
until who | grep Bill
do 
    sleep 30
done
echo Bill is logged in!</nowiki>

==Filtering with Loops and Branches==

Loops and branch output may be filtered.

<b>Example</b>
 <nowiki>
$ cat sys_hogs
for user in `awk -F: '{ print $1 }' /etc/passwd`
do
    if [ ${user} != 'root' -a ${user} != 'aman' ]
    then
        home=`awk -F: "/^${user}:/ { print \\$6 }" /etc/passwd`
        usage=`du -s ${home} | awk '{ print $1 }'`
        echo ${usage} ${user}
    fi
done | sort -nr | head | awk '{ print $2 ":", $1 }'</nowiki>

Complex scripts often require debugging

 <nowiki>
$ sh -x sys_hogs  # trace execution</nowiki>

==Redirecting Standard Error==

The standard input, output and error streams are associated with file descriptors :
* standard input - 0 
* standard output - 1 
* standard error - 2



* < redirects 0 (standard input)
* > and >> redirect 1 (standard output)
* Preceding > or >> with 2 redirects standard error
 <nowiki>
myprog < foo > foo.out 2> foo. err</nowiki>
* Standard error may be combined with standard output
 <nowiki>
myprog < foo > foo.allout 2>&1</nowiki>

==The read Command==

The read command is a simple command that allows for user input. The read command waits for input and then assigns it to variables passed as parameters to the read command

 <nowiki>
$ read line
This is a test

$ echo $line
This is a test

$ read word rest
This is another test

$ echo $word
This

$ echo $rest
is another test</nowiki>

Note that read typical will read in the phrase and then the return key as a second input. If you have multiple reads after one another you may end up with the return key as the input to the second read. The best way to avoid this is to assign two variables to the read command. 1 will take the input, the 2nd will take the return key, which at that time you can discard

You should also note that when read is given multiple variables as parameters to store the input text, read will put each individual word in its own variable using spaces to delimit the start and end of a word, until the last variable where it will place the rest of the read input.

==Arithmetic with expr==

The shell has no built-in arithmetic operations. The expr command provides arithmetic and other facilities for use in shell scripts or
other contexts. expr takes an arithmetic expression as arguments and prints the value to standard output; metacharacters must be suppressed:
 <nowiki>
$ expr 7 \* \( 2 + 3 \)
35
$ a=1; a=`expr $a + 1`; echo $a
2</nowiki>
