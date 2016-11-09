# pdksh enhanced
pdksh: public domain korn shell with command auto completion.
---
My pdksh project is mainly focusing on interactive shells.
I use vi editing mode, so I will try to fix any bug found in it. I added supports for arrow keys and Home, End, PgUp, PgDn, so the cursor can be moved easier.

I also added programmed tab-completion, just like bash. It's kind of complicated, though, and there still may be bugs. OK, let me introduce the programmed tab-completion to you.

First of all, you may need to edit your $HOME/.kshrc to setup your tab-completion configuration. Some lines of mine look like:

<pre>
# interactive shell only
[[ "$-" == *i* ]] || return
PS1='$USER@${PWD}$ '
# list of alias and predefined function/variable.
#...
#...
#...
[[ "$KSH_VERSION" == *PD* ]] || return
# command completions
complete 'sudo=C'
complete 'git=S:add,:status,:commit,clone,:diff,:log'
complete 'svn=S:add,:status,:commit,checkout,@--diff-cmd,:diff,:log'
complete 'systemctl=F_systemctl'
function _systemctl {
	typeset ACT="enable disable start stop restart status reload"
	case $# in
	0)
		_COMPLETE=$ACT
		;;
	1)
		if [[ -z "$1" ]]; then
			_COMPLETE=$ACT
		else
			typeset -i i=0
			typeset outs
			typeset a
			for a in $ACT; do
				if [[ ${a#$1} != $a ]]; then
					outs[$((i++))]=$a
				fi
			done
			_COMPLETE="${outs[@]}"
		fi
		;;
	*)
		shift $(($# - 1))
		_COMPLETE=$(cd /lib/systemd/system;
		if [ -z "$1" ];then
			echo *
		elif echo $1* |grep -F '*' > /dev/null 2>&1;then
			:
		else
			echo $1*
		fi)
		;;
	esac
}

</pre>
As you may already figure it out. There is a new builtin command "complete". We use the command to make customized tab-completion work as we expected.

the builtin "complete" has 3 kinds of arguments. listed below:

1. _COMMAND_NAME_=C
    indicates that the word following _COMMAND_NAME_ should be a command.
2. _COMMAND_NAME_=S<i>ARG_LIST</i>
    indicates that the word following _COMMAND_NAME_ should be a customized _ARG_ joined by "," and may be prefixed by ":" or "@" in _ARG_LIST_.
    
    The prefix ":" means the <i>ARG</i> needs a file name argument, while "@" means the <i>ARG</i> needs a command name argument.
3. _COMMAND_NAME_=F<i>FUNC</i>
    indicates that the word following _COMMAND_NAME_ should be generated by calling shell function _FUNC_.
    
    Arguments passed to function _FUNC_ is all what you have already typed in the current command line.
    
    In function _FUNC_, generated candidate-string list, should be assigned to \_COMPLETE variable. <b>DONOT</b> print anything out, or it will mess your command line up.
    
    Any variable except \_COMPLETE should be declared with _typeset_ or _local_ keyword.


for now it does not support special char like space or quotations, anyway, almost nobody would like to use those chars as command line options!
