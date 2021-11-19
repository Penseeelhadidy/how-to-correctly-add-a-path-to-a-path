# how-to-correctly-add-a-path-to-a-path
Article for your reference 


https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path



How to correctly add a path to PATH?
Ask Question
Asked 9 years, 11 months ago
Active 9 months ago
Viewed 2.8m times

Report this ad

1194


609
I'm wondering where a new path has to be added to the PATH environment variable. I know this can be accomplished by editing .bashrc (for example), but it's not clear how to do this.

This way:

export PATH=~/opt/bin:$PATH
or this?

export PATH=$PATH:~/opt/bin
bash
environment-variables
path
bashrc
Share
Improve this question
Follow
edited Sep 17 '15 at 18:27

Scott
9,46899 gold badges3737 silver badges6363 bronze badges
asked Dec 4 '11 at 20:57

Paolo
15k1010 gold badges2727 silver badges3838 bronze badges
printf '\nPATH=$PATH:"path-to-add"\nexport PATH\n' >> ~/.bashrc – 
Microsoft Linux TM
 Nov 7 '14 at 13:04
4
Unix shell function for adding directories to PATH – 
Sildoreth
 Apr 30 '15 at 13:59 
1
If there are already some paths added, e.g. PATH=$PATH:$HOME/.local/bin:$HOME/bin, another can be added by separating with a : e.g. PATH=$PATH:$HOME/.local/bin:$HOME/bin:/home/ec2-user/pear/bin. – 
Sandeepan Nath
 Aug 20 '16 at 18:40
2
Do these answers work for all flavors of linux? – 
Ungeheuer
 Nov 20 '16 at 20:58
1
wrote a little utility to help with exactly this. github.com/aalok-sathe/pathin – 
axolotl
 May 12 '20 at 13:48
Show 1 more comment
12 Answers
Active
Oldest
Votes
 
1339

The simple stuff

PATH=$PATH:~/opt/bin
or

PATH=~/opt/bin:$PATH
depending on whether you want to add ~/opt/bin at the end (to be searched after all other directories, in case there is a program by the same name in multiple directories) or at the beginning (to be searched before all other directories).

You can add multiple entries at the same time. PATH=$PATH:~/opt/bin:~/opt/node/bin or variations on the ordering work just fine. Don't put export at the beginning of the line as it has additional complications (see below under “Notes on shells other than bash”).

If your PATH gets built by many different components, you might end up with duplicate entries. See How to add home directory path to be discovered by Unix which command? and Remove duplicate $PATH entries with awk command to avoid adding duplicates or remove them.

Some distributions automatically put ~/bin in your PATH if it exists, by the way.

Where to put it

Put the line to modify PATH in ~/.profile, or in ~/.bash_profile if that's what you have.

Note that ~/.bash_rc is not read by any program, and ~/.bashrc is the configuration file of interactive instances of bash. You should not define environment variables in ~/.bashrc. The right place to define environment variables such as PATH is ~/.profile (or ~/.bash_profile if you don't care about shells other than bash). See What's the difference between them and which one should I use?

Don't put it in /etc/environment or ~/.pam_environment: these are not shell files, you can't use substitutions like $PATH in there. In these files, you can only override a variable, not add to it.

Potential complications in some system scripts

You don't need export if the variable is already in the environment: any change of the value of the variable is reflected in the environment.¹ PATH is pretty much always in the environment; all unix systems set it very early on (usually in the very first process, in fact).

At login time, you can rely on PATH being already in the environment, and already containing some system directories. If you're writing a script that may be executed early while setting up some kind of virtual environment, you may need to ensure that PATH is non-empty and exported: if PATH is still unset, then something like PATH=$PATH:/some/directory would set PATH to :/some/directory, and the empty component at the beginning means the current directory (like .:/some/directory).

if [ -z "${PATH-}" ]; then export PATH=/usr/local/bin:/usr/bin:/bin; fi
Notes on shells other than bash

In bash, ksh and zsh, export is special syntax, and both PATH=~/opt/bin:$PATH and export PATH=~/opt/bin:$PATH do the right thing even. In other Bourne/POSIX-style shells such as dash (which is /bin/sh on many systems), export is parsed as an ordinary command, which implies two differences:

~ is only parsed at the beginning of a word, except in assignments (see How to add home directory path to be discovered by Unix which command? for details);
$PATH outside double quotes breaks if PATH contains whitespace or \[*?.
So in shells like dash, export PATH=~/opt/bin:$PATH sets PATH to the literal string ~/opt/bin/: followed by the value of PATH up to the first space. PATH=~/opt/bin:$PATH (a bare assignment) doesn't require quotes and does the right thing. If you want to use export in a portable script, you need to write export PATH="$HOME/opt/bin:$PATH", or PATH=~/opt/bin:$PATH; export PATH (or PATH=$HOME/opt/bin:$PATH; export PATH for portability to even the Bourne shell that didn't accept export var=value and didn't do tilde expansion).

¹ This wasn't true in Bourne shells (as in the actual Bourne shell, not modern POSIX-style shells), but you're highly unlikely to encounter such old shells these days.
Share
Improve this answer
Follow
edited Feb 28 '20 at 13:10

Paulo Tomé
3,58155 gold badges2323 silver badges3535 bronze badges
answered Dec 4 '11 at 23:39

Gilles 'SO- stop being evil'
721k174174 gold badges15041504 silver badges19961996 bronze badges
Still not able to understand the complication with export. can you please simplify it? – 
priojeet priyom
 Jul 12 '19 at 5:49
2
@priojeetpriyom Simple explanation: you don't need export. – 
Gilles 'SO- stop being evil'
 Jul 12 '19 at 8:30
2
Thank you for this answer, perfectly detailed. You say "You should not define environment variables in ~/.bashrc", but unfortunately 100% of the programs that I have installed on my system that modify the path (FZF and Rust's Cargo) modify the path in .bashrc. I assume because FZF is written in Rust too it's following the pattern of Rust. – 
icc97
 Jul 30 '19 at 10:32 
could it also be export PATH ... – 
therobyouknow
 Dec 4 '19 at 11:50
What about when the element being added to $PATH is an environment variable set inside ~/.bashrc? – 
Tom Russell
 Feb 18 at 2:35 
Show 1 more comment

Report this ad
 
94

Either way works, but they don't do the same thing: the elements of PATHare checked left to right. In your first example, executables in ~/opt/bin will have precedence over those installed, for example, in /usr/bin, which may or may not be what you want.

In particular, from a safety point of view, it is dangerous to add paths to the front, because if someone can gain write access to your ~/opt/bin, they can put, for example, a different ls in there, which you'd then probably use instead of /bin/ls without noticing. Now imagine the same for ssh or your browser or choice... (The same goes triply for putting . in your path.)
Share
Improve this answer
Follow
answered Dec 4 '11 at 21:09

Ulrich Schwarz
14.2k22 gold badges4343 silver badges5757 bronze badges
8
But if you want to have your own, customized version of ls, you need to put it in a directory ahead of /bin. – 
Barmar
 Sep 17 '15 at 20:20
20
or alias ls=myls – 
waltinator
 Sep 18 '15 at 1:27
Add a comment
 
63

The bullet-proof way of Appending/Prepending

Try not using

PATH=$PATH:~/opt/bin
or

PATH=~/opt/bin:$PATH
Why? There are a lot of considerations involved in the choice of appending versus prepending. Many of them are covered in other answers, so I will not repeat them here.

An important point is that, even if system scripts do not use this (I wonder why)*1, the bullet-proof way to add a path (e.g., ~/opt/bin) to the PATH environment variable is

PATH="${PATH:+${PATH}:}~/opt/bin"
for appending (instead of PATH="$PATH:~/opt/bin") and

PATH="~/opt/bin${PATH:+:${PATH}}"
for prepending (instead of PATH="~/opt/bin:$PATH")

This avoids the spurious leading/trailing colon when $PATH is initially empty, which can have undesired side effects and can become a nightmare, elusive to find (this answer briefly deals with the case the awk-way).

Explanation (from Shell Parameter Expansion):

${parameter:+word}
If parameter is null or unset, nothing is substituted, otherwise the expansion of word is substituted.
Thus, ${PATH:+${PATH}:} is expanded to:

nothing, if PATH is null or unset,
${PATH}:, if PATH is set.
Note: This is for bash.

*1 I have just found that scripts like `devtoolset-6/enable` actually use this,
$ cat /opt/rh/devtoolset-6/enable
# General environment variables
export PATH=/opt/rh/devtoolset-6/root/usr/bin${PATH:+:${PATH}}
...
Share
Improve this answer
Follow
edited Dec 27 '20 at 8:59
answered Jan 5 '18 at 16:36

sancho.s ReinstateMonicaCellio
1,99199 silver badges1818 bronze badges
1
${PATH:+:${PATH}} is confusing because the first : is part of the syntax and the second : is the list delimiter. – 
Roger Dahl
 Jan 31 at 2:38
Well, I guess you will have to get used to it. – 
sancho.s ReinstateMonicaCellio
 Jan 31 at 9:18
: is context-sensitive. It's a rudimentary form of AI. :-) – 
Tom Russell
 Feb 18 at 3:04
Add a comment

Report this ad
 
40

I'm confused by question 2 (since removed from the question since it was due to an unrelated issue):

What's a workable way to append more paths on different lines? Initially I thought this could do the trick:

export PATH=$PATH:~/opt/bin
export PATH=$PATH:~/opt/node/bin
but it doesn't because the second assignment doesn't only append  ~/opt/node/bin, but also the whole PATH previously assigned.

This is a possible workaround:

export PATH=$PATH:~/opt/bin:~/opt/node/bin
but for readability I'd prefer to have one assignment for one path.
If you say

PATH=~/opt/bin
that's all that will be in your PATH. PATH is just an environment variable, and if you want to add to the PATH, you have to rebuild the variable with exactly the contents you want. That is, what you give as an example to question 2 is exactly what you want to do, unless I'm totally missing the point of the question.

I use both forms in my code. I have a generic profile that I install on every machine I work on that looks like this, to accommodate for potentially-missing directories:

export PATH=/opt/bin:/usr/local/bin:/usr/contrib/bin:/bin:/usr/bin:/usr/sbin:/usr/bin/X11
# add optional items to the path
for bindir in $HOME/local/bin $HOME/bin; do
    if [ -d $bindir ]; then
        PATH=$PATH:${bindir}
    fi
done
Share
Improve this answer
Follow
edited Jan 4 '16 at 19:17

Paolo
15k1010 gold badges2727 silver badges3838 bronze badges
answered Dec 5 '11 at 0:25

Carl Cravens
63644 silver badges55 bronze badges
2
You are right about the example of question 2, it works. Another PATH related issue on my system confused me. Sorry for that. – 
Paolo
 Dec 5 '11 at 0:59
Add a comment
 
25

Linux determines the executable search path with the $PATH environment variable. To add directory /data/myscripts to the beginning of the $PATH environment variable, use the following:

PATH=/data/myscripts:$PATH
To add that directory to the end of the path, use the following command:

PATH=$PATH:/data/myscripts
But the preceding are not sufficient because when you set an environment variable inside a script, that change is effective only within the script. There are only two ways around this limitation:

If within the script, you export the environment variable it is effective within any programs called by the script. Note that it is not effective within the program that called the script.
If the program that calls the script does so by inclusion instead of calling, any environment changes in the script are effective within the calling program. Such inclusion can be done with the dot command or the source command.
Examples:

$HOME/myscript.sh
source $HOME/myscript.sh
Inclusion basically incorporates the "called" script in the "calling" script. It's like a #include in C. So it's effective inside the "calling" script or program. But of course, it's not effective in any programs or scripts called by the calling program. To make it effective all the way down the call chain, you must follow the setting of the environment variable with an export command.

As an example, the bash shell program incorporates the contents of file .bash_profile by inclusion. Place the following 2 lines in .bash_profile:

PATH=$PATH:/data/myscripts
export PATH
effectively puts those 2 lines of code in the bash program. So within bash, the $PATH variable includes $HOME/myscript.sh, and because of the export statement, any programs called by bash have the altered $PATH variable. And because any programs you run from a bash prompt are called by bash, the new path is in force for anything you run from the bash prompt.

The bottom line is that to add a new directory to the path, you must append or prepend the directory to the $PATH environment variable within a script included in the shell, and you must export the $PATH environment variable.

More information here
Share
Improve this answer
Follow
edited Mar 13 '19 at 5:39

Johan
3,77722 gold badges2222 silver badges2929 bronze badges
answered Dec 5 '11 at 0:31

Steve Brown
1,87311 gold badge1414 silver badges88 bronze badges
Add a comment
 
22

For some time now I've kept with me two functions pathadd and pathrm that assist in adding elements to the path without the need to worry about duplications.

pathadd takes a single path argument and an optional after argument which if supplied will append to the PATH otherwise it prepends it.

In almost every situation if you're adding to the path then you're likely wanting to override anything already in the path, which is why I opt to prepend by default.

pathadd() {
    newelement=${1%/}
    if [ -d "$1" ] && ! echo $PATH | grep -E -q "(^|:)$newelement($|:)" ; then
        if [ "$2" = "after" ] ; then
            PATH="$PATH:$newelement"
        else
            PATH="$newelement:$PATH"
        fi
    fi
}

pathrm() {
    PATH="$(echo $PATH | sed -e "s;\(^\|:\)${1%/}\(:\|\$\);\1\2;g" -e 's;^:\|:$;;g' -e 's;::;:;g')"
}
Put these in any script you wish to alter the PATH environment and you can now do.

pathadd "/foo/bar"
pathadd "/baz/bat" after
export PATH
You're guaranteed not to add to the path if it's already there. If you now want to ensure /baz/bat is at the start.

pathrm "/baz/bat"
pathadd "/baz/bat"
export PATH
Now any path can be moved to the front if it's already in the path without doubling.
Share
Improve this answer
Follow
edited May 10 '17 at 13:15

Mark
10322 bronze badges
answered Mar 17 '16 at 23:28

Brett Ryan
33122 silver badges55 bronze badges
Related and cleaner approach to check for presence of a directory in your PATH: unix.stackexchange.com/a/32054/135943 – 
Wildcard
 Jul 26 '19 at 19:03
I've found it useful to add to this a check that the path exists as well, and silently ignore the call if it doesn't. – 
Roger Dahl
 Feb 1 at 0:47
1
That check is already present @RogerDahl , the use of -d – 
Brett Ryan
 Feb 1 at 11:02
Add a comment
 
12

I can't speak for other distributions, but Ubuntu has a file, /etc/environment, that is the default search path for all users. Since my computer is only used by me, I put any directories that I want in my path there, unless it is a temporary addition that I put in a script.
Share
Improve this answer
Follow
answered Feb 7 '15 at 5:17

Jim Bradley
12111 silver badge22 bronze badges
Add a comment
 
10

To add a new path to the PATH environment variable:

export PATH=$PATH:/new-path/
For this change to be applied to every shell you open, add it to the file that the shell will source when it is invoked. In different shells this can be:

Bash Shell: ~/.bash_profile, ~/.bashrc or profile
Korn Shell: ~/.kshrc or .profile
Z Shell: ~/.zshrc or .zprofile
e.g.

# export PATH=$PATH:/root/learning/bin/
# source ~/.bashrc
# echo $PATH
You can see the provided path in the above output.
Share
Improve this answer
Follow
answered Jun 12 '17 at 10:47

Amit24x7
61888 silver badges2323 bronze badges
Add a comment
 
8

There are some situations where it using PATH=/a/b:$PATH might be considered the "incorrect" way to add a path to PATH:

Adding a path that's not actually a directory.
Adding a path that's already in PATH in the same form.
Adding a relative path (since the actual directory searched would change as you change the current working directory).
Adding a path that's already in PATH in a different form (i.e., an alias due to using symlinks or ..).
If you avoid doing 4, not moving the path to the front of PATH when it's intended to override other entries in PATH.
This (Bash-only) function does the "right thing" in the above situations (with an exception, see below), returns error codes, and prints nice messages for humans. The error codes and messages can be disabled when they're not wanted.

prepath() {
    local usage="\
Usage: prepath [-f] [-n] [-q] DIR
  -f Force dir to front of path even if already in path
  -n Nonexistent dirs do not return error status
  -q Quiet mode"

    local tofront=false errcode=1 qecho=echo
    while true; do case "$1" in
        -f)     tofront=true;       shift;;
        -n)     errcode=0;          shift;;
        -q)     qecho=':';          shift;;
        *)      break;;
    esac; done
    # Bad params always produce message and error code
    [[ -z $1 ]] && { echo 1>&2 "$usage"; return 1; }

    [[ -d $1 ]] || { $qecho 1>&2 "$1 is not a directory."; return $errcode; }
    dir="$(command cd "$1"; pwd -P)"
    if [[ :$PATH: =~ :$dir: ]]; then
        $tofront || { $qecho 1>&2 "$dir already in path."; return 0; }
        PATH="${PATH#$dir:}"        # remove if at start
        PATH="${PATH%:$dir}"        # remove if at end
        PATH="${PATH//:$dir:/:}"    # remove if in middle
    fi
    PATH="$dir:$PATH"
}
The exception is that this function does not canonicalize paths added to PATH via other means, so if a non-canonical alias for a path is in PATH, this will add a duplicate. Trying to canonicalize paths already in PATH is a dicey proposition since a relative path has an obvious meaning when passed to prepath but when already in the path you don't know what the current working directory was when it was added.
Share
Improve this answer
Follow
edited Nov 2 '17 at 9:13
answered Nov 2 '17 at 3:30

cjs
56344 silver badges1313 bronze badges
regarding relative paths: what about having a '-r' switch, which would add the path without making it absolute first, and which would also look for it as absolute before adding it? If this were script, one could use it in other shells. Is there any benefit of having it as a function? nice code! – 
hoijui
 Jun 10 '19 at 6:23
1
@hoijui It has to be a function because it's modifying the current environment. If it were a script, it would modify the environment of the subprocess running the script and, when the script exited you'd have the same $PATH as you had before. As for -r, no, I think that relative paths in $PATH are just too unreliable and weird (your path changes every time you cd!) to want to support something like that in a general tool. – 
cjs
 Jun 10 '19 at 10:15
Add a comment
 
6

For me (on Mac OS X 10.9.5), adding the path name (e.g. /mypathname) to the file /etc/paths worked very well.

Before editing, echo $PATH returns:

/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin
After editing /etc/paths and restarting the shell, the $PATH variable is appended with /pathname. Indeed, echo $PATH returns:

/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/mypathname
What happened is that /mypathname has been appended to the $PATH variable.
Share
Improve this answer
Follow
answered Oct 27 '15 at 15:01

faelx
6111 silver badge11 bronze badge
3
Better to add a file to the /etc/paths.d directory than to edit the /etc/paths file itself. – 
rbrewer
 Sep 7 '16 at 20:50
Add a comment
 
5

Here is my solution:

PATH=$(echo -n $PATH | awk -v RS=: -v ORS=: '!x[$0]++' | sed "s/\(.*\).\{1\}/\1/")
A nice easy one liner that doesn't leave a trailing :
Share
Improve this answer
Follow
edited Jul 29 '17 at 9:11

simhumileco
43555 silver badges1212 bronze badges
answered Dec 29 '14 at 7:12

AJ.
21711 gold badge33 silver badges66 bronze badges
1
-bash: awk: No such file or directory -bash: sed: No such file or directory – 
davidcondrey
 Nov 21 '16 at 19:18
1
@davidcondrey - awk and sed are very common external commands. This answer provides a pure-bash way of achieving the same, so it works even in cases when awk and/or sed are not present (or their respective directories are not in the path!) – 
sancho.s ReinstateMonicaCellio
 Jan 19 '18 at 9:45
Add a comment
 
0

POSIX-compliant

pathedit()
{
    [ -z "$2" ] && return 2
    PATH=$(printf ":$PATH:" | sed "s:\:$2\::\::g")
    case $1 in
    -p ) PATH=$2$PATH ;;
    -a ) PATH=$PATH$2 ;;
    -r ) ;;
    * ) return 2 ;;
    esac
    PATH=$(printf "$PATH" | tr -s :) PATH=${PATH#:} PATH=${PATH%:}
}
pathedit -p path prepends path to $PATH
pathedit -a path appends path to $PATH
pathedit -r path removes path from $PATH

also removes extraneous colons

without external commands (non-POSIX)

pathedit()
{
    [[ -z $2 ]] && return 2
    PATH=:$PATH: PATH=${PATH//:$2:/}
    case $1 in
    -p ) PATH=$2$PATH ;;
    -a ) PATH=$PATH$2 ;;
    -r ) ;;
    * ) return 2 ;;
    esac
    while [[ $PATH == *::* ]] ; do
        PATH=${PATH//::/:}
    done
    PATH=${PATH#:} PATH=${PATH%:}
}
