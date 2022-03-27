---
title: Customize Your Zsh Prompt
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [June 2, 2019](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html "2:35 pm") 
[VOID001](https://web.archive.org/web/20210514143128/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [4 comments](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comments)





Screenshot
----------


![This image has an empty alt attribute; its file name is 78d4e48a5339575a8227fa866adcb765b8b641.png](https://web.archive.org/web/20210514143128im_/https://img.vim-cn.com/e6/78d4e48a5339575a8227fa866adcb765b8b641.png)

You may need “good network condition” to preview this anime 



![](https://web.archive.org/web/20210514143128im_/https://img.vim-cn.com/e6/78d4e48a5339575a8227fa866adcb765b8b641.png)Screenshot for my zsh prompt
Instructions
------------


I choose to use zsh **promptinit** system to write my own themes, which can be easily managed by zsh `prompt`


### Before you start


the `promptinit` uses autoload mechanism to load the theme file, so first we need to ensure the theme file **locates inside fpath**, if not, we should add the path containing your theme file into `fpath` array, just like the following (suggest my theme is in ~/.zsh/themes): 



```
fpath=(~/.zsh/themes $fpath)
```

**Remember to put this line before any fpath & autoload modification, e.g: if you are using grml-zsh-config, it’s recommended to put this line into ~/.zsh.pre** 


And then you should set the theme file name to `prompt_<theme>_setup`, this will be recognized by the zsh promptinit system. Then create a function named `prompt_<theme>_setup` inside your theme file. Your theme file now should have a basic skeleton like this (e.g: your theme name is moe). 



```
prompt\_moe\_setup () {

}

prompt\_moe\_setup [[email protected]](/web/20210514143128/https://void-shana.moe/cdn-cgi/l/email-protection)
```

According to the [manpage](#References) [1], You can declare the following functions for your prompt theme: 


* prompt\_<theme>\_precmd: Add it into precmd hook(`add-zsh-hook precmd prompt_<theme>_precmd`) to enable theme precmd



> Quick reference to hook functions: zsh has some defined hook functions, such as `precmd`, `preexec`, etc, you can define an array with the string “\_functions” append, such as `precmd_functions`, You can try `echo $precmd_functions` to see what’s the content inside. And all the elements inside the array will be treated as functions, execute with the same context and argument as the basic function. If the function not found, just siliently ignore it, and continue to next one. for more information please refer to the [zsh doc functions section](#References-(1)) [2]
> 
> 


* prompt\_<theme>\_preview: Enable prompt preview via `prompt -p`
* prompt\_<theme>\_help: Enable help command via `prompt -h`


### Customize the PS, RPS


 By convention, themes use for `PS1`, `PS2`, `RPS1`, etc. instead of `PROMPT`, `RPROMPT`. As the example above, inside `prompt_moe_setup` we can define `PS1`, `PS2`, etc. But what are `PS1`, `PS2`, `RPS1`? 


* PS1: The primary prompt string, displays just before the command input.
* PS2: When your command is not finished in one line, it will display in
the next following lines until your command is finsihed (e.g.: write a
multi-line while loop in zsh)
* RPS1: It will display at the right hand side when PS1 is displayed. (For
me, a right hand side RPS1 is useful to display error code information)


There are also other prompt parameters(officially called parameters by zsh, but I think maybe variable is more intuitive) such as `PS3`,`PS4`, etc. but we don’t really use it in common. For all the prompt variables, they all accept the same prompt escape sequence. The following are some commonly used escape sequence: 


* `%/` `%~`: Current working dir, Try them to see the difference.
* `%m` `%M`: machine host name, either in short or full format.
* `%B` `%b`: Begin / end **bold** text mode.
* `%n`: $USERNAME variable


For a full escape sequence reference please refer to the zsh-manual [Propmt Expansion](#References) [3]. To debug your prompt string, you can simply use `print -rP <STRING>` to see the effect of your prompt, no need to reload zsh each time. 


A trick of displaying error code information: I use `%(?..%F{red}\(?%f)` to display the error code in \)RPS1, it will only show a red error code when the error code is not 0. The form `%(?.<true-text>.<false-text>)` is a conditional substring, means when the exit status of the last command is not 0, it will display the `<false-text>` else display `<true-text>`, we just leave the `<true-text>` blank so it won’t display anything when the command exit without any error. 


### Enable version control info


It might be good to display the git info (or other vcs) along with your prompt. zsh has a powerful module vcs\_info that can help you with this need. 


For a quick start, you need to add the following lines into your theme file. (Still use the example above) 



```
prompt\_opts=(subst percent)

function prompt\_moe\_precmd() {
    vcs\_info
}

```

Then append `\(vcs_info_msg_0_` into your prompt. Be careful that you should append the plain text `\)vcs_info_msg_0` instead of this variable, since the variable will be evaluated only when it is declared, while the plain mode is subjected to [Parameter Expansion](#References) [3], that is, it will be evaluated each time the text displays. The following line is an example of adding vcs\_info to your PS1: 



```
PS1="$PS1 \$vcs\_info\_msg\_0\_"

```

You might want also customize the `$vcs_info_msg_0_` prompt, such as changing the color, add / remove information displayed. You can achieve this with `zstyle`. Following are a list of common options: 


* `zstyle ':vcs_info:*' formats <STRING>`: Set the normal
vcs\_info format string(when you are not in rebase, merge or other
actions it will display this one), you can get the current value by `zstyle | grep vcs_info -B1`.
* `zstyle ':vcs_info:*' actionformats <STRING>`: Set the vcs\_info string when in rebase, merge or other actions.
* `zstyle 'vcs_info:*' check-for-changes true/false`: Enable /
Disable checking for the uncommitted changes in current workdir, and if
there are uncommitted changes it will display either %u (for unstaged
changes) or %c (for staged changes). These to escape sequences can be
also configured by `zstyle`
* `zstyle ':vcs_info:*' unstagedstr <STRING>`: Set the string to display for %u


Note that we use “:vcs\_info:*” above to match for all kinds of vcs systems as long as all repos to give them the settings. If you want to set different options for specified repos / vcs (e.g: Disable the check-for-changes for kernel repo since it’s too big and cause delay), see the example below: 



```
zstyle ':vcs\_info:*:*:latest-linux-kernel' check-for-changes false

```

This will only disable check for unstaged change for repo with root directory named “latest-linux-kernel”. 


About a full reference about vcs\_info usage and configuration, see [zsh documentation for vcs\_info](#References) [4]


### Async Zsh Prompt


Although the vcs\_info is useful, you may already found lags when turn on the the `check-for-changes` option, esp. in large repos. It’s annoying that the prompt doesn’t display instantly after we press enter. One option is to disable the `check-for-changes` option for these huge repos, however it’s just a workaround. 


Fortunately, we have async zsh job support which can solve the problem. There are different kinds of async plugin we can use in zsh, for this blog we will use [zsh-async](#References) [5]. 


zsh-async supports async jobs as well as callback handlers. Along with ZLE ([Zsh Line Editor](#References) [6]) command `zle reset-prompt` we can achieve the async update of `PS1`: 



```
function prompt\_moe\_setup() {
    # Other codes ...    
    async\_start\_worker vcs\_updater\_worker
    async\_register\_callback vcs\_updater\_worker do\_update
}

function do\_update() {
    vcs\_info
    zle reset-prompt
}

function prompt\_moe\_precmd() {
    async\_job vcs\_updater\_worker
}


```

And then you have your async vcs\_info now! But the code above is somehow glitchy, An optimized one can be found here: [https://gist.github.com/VOID001/588347b0ef4b3a14759579e8bf6acb23#file-prompt\_moe\_setup-L48](https://web.archive.org/web/20210514143128/https://gist.github.com/VOID001/588347b0ef4b3a14759579e8bf6acb23#file-prompt_moe_setup-L48) (Still have some glitches but it’s currently fine for me) 



The code above does the following things:



* In executing hook precmd, it starts an empty async task, which will return immediately.
* After the async job finished, it will execute the callback function: `do_update`.
* In `do_update`, we call vcs\_info to update the
$vcs\_info\_msg\_0\_ (Note this will not block the prompt or any interaction
with current prompt). Then we use `zle reset-prompt` to reload current prompt.


Ending
------


Writing bash/zsh script is somehow painful for me so I previously stick to oh-my-zsh for a long time. However the zsh completion and git-prompt-info is so slow that always bother me. So I tried to switch to `grml-zsh-config` and define a custom prompt for me, what I found suprising is that writing (simple) zsh scripts are not as difficult as I think thanks to the well-documented zsh manpage and online manual. So I wrote this article as a “Zsh Theme Creation Quick Start Guide” for those who are dissatisfied with the oh-my-zsh themes and want to have a chance to create your own theme. 


Thanks for [@lilydjwg](https://web.archive.org/web/20210514143128/https://blog.lilydjwg.me/) for giving me lots of help when writing the theme. Thanks for @swordfeng for providing me with cute sample theme. 


References
----------


* **(1)** zshall(1) manual page: “PROMPT THEMES”
* **(2)** [http://zsh.sourceforge.net/Doc/Release/Functions.html](https://web.archive.org/web/20210514143128/http://zsh.sourceforge.net/Doc/Release/Functions.html)
* **(3)** [http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Prompt-Expansion](https://web.archive.org/web/20210514143128/http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Prompt-Expansion)
* **(4)** [http://zsh.sourceforge.net/Doc/Release/User-Contributions.html#Version-Control-Information](https://web.archive.org/web/20210514143128/http://zsh.sourceforge.net/Doc/Release/User-Contributions.html#Version-Control-Information)
* **(5)** [https://github.com/mafredri/zsh-async/](https://web.archive.org/web/20210514143128/https://github.com/mafredri/zsh-async/)
* **(6)** [http://zsh.sourceforge.net/Doc/Release/Zsh-Line-Editor.html#Zsh-Line-Editor](https://web.archive.org/web/20210514143128/http://zsh.sourceforge.net/Doc/Release/Zsh-Line-Editor.html#Zsh-Line-Editor)






---


[archlinux](https://web.archive.org/web/20210514143128/https://void-shana.moe/category/linux/archlinux), [Linux](https://web.archive.org/web/20210514143128/https://void-shana.moe/category/linux) [linux](https://web.archive.org/web/20210514143128/https://void-shana.moe/tag/linux), [prompt](https://web.archive.org/web/20210514143128/https://void-shana.moe/tag/prompt), [zsh](https://web.archive.org/web/20210514143128/https://void-shana.moe/tag/zsh) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514143128im_/https://secure.gravatar.com/avatar/d8fb8dde3de28b1d063f405a6007e1ac?s=50&d=identicon&r=g) **naruto-ding** says: 
[January 13, 2020 at 2:00 am](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comment-1049)
Hello,  


	1. ![](https://web.archive.org/web/20210514143128im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[January 14, 2020 at 3:44 pm](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comment-1052)
	Hello naruto-ding,
	Glad you like it! But I am sorry to tell you that I don’t have enough time to finish that series. That series of video is made when I was a senior student. The neu-os and the video tutorial is for teaching use of the “Linux Operating System” course in my school. After I graduate I have many other things to work and study so I don’t have the time to do that. I tried to ask the current maintainer of neu-os in my previous school to continue publishing some video tutorials but they are also very busy. Making one episode of the series tutorial will take me more than 12 hours – 20 hours or even further, there are a lot of preparation needed such as the slides, the references, the sample code, etc.  
	However, in the future although I cannot update these OS tutorial videos, I will try to post some short videos (not in series) related to the operating system (such as ext4 file system, linux utility, command-line tools, etc). This kind of video will be less time consuming to made and to watch. Also I think it will provide some useful information for the audience.


2. ![](https://web.archive.org/web/20210514143128im_/https://secure.gravatar.com/avatar/d8fb8dde3de28b1d063f405a6007e1ac?s=50&d=identicon&r=g) **naruto-ding** says: 
[January 13, 2020 at 2:10 am](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comment-1050)
I realize the comments under each blog is quite slow. Hope to get the chance to consult with you for daily questions if it doesn’t bother you too much. Maybe your email for further contact. (✿◡‿◡)


	1. ![](https://web.archive.org/web/20210514143128im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[January 14, 2020 at 3:46 pm](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comment-1053)
	Hello naruto-ding,  
	You can find my contact information here: [https://void-shana.moe/void](https://web.archive.org/web/20210514143128/https://void-shana.moe/void)  
	Feel free to send me the email/in-site comments/questions about os via github issue [https://github.com/VOID001/neu-os](https://web.archive.org/web/20210514143128/https://github.com/VOID001/neu-os), I will be very happy to exchange thoughts with you.


	1. ![](https://web.archive.org/web/20210514143128im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[January 14, 2020 at 3:46 pm](https://web.archive.org/web/20210514143128/https://void-shana.moe/linux/customize-your-zsh-prompt.html#comment-1053)
	Hello naruto-ding,  
	You can find my contact information here: [https://void-shana.moe/void](https://web.archive.org/web/20210514143128/https://void-shana.moe/void)  
	Feel free to send me the email/in-site comments/questions about os via github issue [https://github.com/VOID001/neu-os](https://web.archive.org/web/20210514143128/https://github.com/VOID001/neu-os), I will be very happy to exchange thoughts with you.

            