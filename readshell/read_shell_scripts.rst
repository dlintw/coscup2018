=====================================
 看懂源碼基礎 - 批次檔除錯及排版工具
=====================================

:Author: Daniel YC Lin(林原志)
:Contact: dlin.tw@gmail.com
:Date: 2018-08-11

簡介
====

* 批次檔:

  * Unix 系列:  Xenix(1989 sh) -> SunOS(tcsh) -> RedHat(sh->bash) -> ArchLinux(2005 zsh)
  * Windows 系列: DOS(command.com) -> Windows(cmd.exe) -> PowerShell
* 臉書社團:

  * 主持: Manjaro 及 Arch Linux Taiwan - 與時俱進，一邊更新一邊學
  * 主持: Golang Taiwan - 它的 killer application 應該算 Docker 吧!
  * 加入: 網樂通改機俱樂部 - 移植 ArchLinux 套件體系到 sh4 CPU
* Open Source 社團: Hacking Thursday
* web site: 派樂靈丹 www.twpda.com - 以 Assembly 修改 ROM
* 近期學習: docker/golang/julia

緣起
====

shell script 批次檔, 通常以為簡單但也飽含玄機與陷阱, 自己寫的通常看的懂,
專家寫的那就不一定了, 先看欣賞兩篇小品吧!

小品解析
========

reflector.sh
------------

reflector.sh 是一個將 Arch Linux download mirrors 網站依存取速度排序的工具

https://www.askapache.com/shellscript/reflector-ranking-mirrors/

* line 1: https://en.wikipedia.org/wiki/Shebang_(Unix)  # 只能代一個參數
* line 22:

  * function: `man bash` 再查 Shell Function Definitions
  * `{{{1`: vim 編輯器 Voom 外掛

* line 24:

  * if...else...fi: `man bash` 再查 Compound Commands (用 else 去查找到的)
  * '[[': `man bash` 再查 Compound Commands (用 \[\[ 找到)
  * `-s`: `man bash` 再查 CONDITIONAL EXPRESSIONS

* line 25: cat: `man cat`
* line 27:

  * curl, sed: `man <keyword>`
  * `|,>`:: https://en.wikipedia.org/wiki/Pipeline_(Unix) # 查 bash Pipelines

* line 44: trap: `which trap`, `type trap`, `man which` # SHELL BUILTIN COMMANDS
* line 49:

  * mktemp: 建立暫存檔 # 最早是用 /tmp/$$.myfile
  * && ||: 查 '&&', 先 '&&' 再 '||'

* line 50: uname 判斷主機 32 或 64 位元
* line 58: '{' 和 '(' 不同, 不需要產生 sub-shell
* line 60-61: 常用小工具 xargs, sort, head, cut, sed, awk
* line 65: 覺得 exit $? 多寫, 最後的 ; 也是多寫的

全篇最重要技巧, 透過 xargs 同時以 curl 測試各網站連線速度

wait-for
--------

wait-for 是用來等待別的 docker container 啟動的 script

https://github.com/eficode/wait-for/blob/master/wait-for

* line 1: 用 sh 讓 https://en.wikipedia.org/wiki/BusyBox 也可以跑
* line 3,4: 好習慣, 常數用全大寫
* line 6: 函數用短寫法才會相容於 busybox 的 ash
* line 12:

  * `<< USAGE`: Here Documents
  * `>&2`: 輸出到 stderr
* line 14: BUG: cmdname not defined
* line 23:
  * for ... in ...; do ... done
  * \`(backquote): 在bash 建議用 $(...)
  * seq: 是從 1..$TIMEOUT 小技巧
* line 24: nc
* line 33: sleep (bash 的 sleep 支援 0.1 秒)
* line 39:

  * while...do...done
  * `$#`: 命令列參數個數, 'Special Parameters'
  * `-gt`: 數字比較, 查 `man test`

* line 41: case ... in ... esac
* line 45: shift 這裡的 1 可省略

全篇重點在使用 busybox 支援的 nc 小工具, 偵測 port 是否開放聽取

排版美化
========

* 先用 git 備份舊版
* shfmt https://github.com/mvdan/sh::

    go get -u mvdan.cc/sh/cmd/shfmt

* .vimrc plugin 以 ":Shfmt" 排版::

    Plug 'z0mbix/vim-shfmt', { 'for': 'sh' }
     let g:shfmt_extra_args = '-s -i 2'

防錯
====

* **set -euo pipefail**
* shellcheck https://www.shellcheck.net/

* vim plugin, https://github.com/vim-syntastic/syntastic
  以 **:w** 檢查, 以 **:Erros** 列出訊息列表, .vimrc 參考::

    Plug 'vim-syntastic/syntastic'
    set statusline+=%F\ %l\:%c
    set statusline+=%#warningmsg#
    set statusline+=%{SyntasticStatuslineFlag()}
    set statusline+=%*

    let g:syntastic_always_populate_loc_list = 0
    let g:syntastic_auto_loc_list = 0
    let g:syntastic_check_on_open = 0
    let g:syntastic_check_on_wq = 0

    " 可用這些指令 :SyntasticInfo :Errors :lnext :lprev

除錯
====

* PS4='+\t $BASH_SOURCE:$LINENO: ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
* read -rp "Enter" ans
* caller: bash SHELL BUILTIN COMMANDS

未完待續
========

* 中篇佳作: pacdiff https://git.archlinux.org/pacman-contrib.git/tree/src/pacdiff.sh.in
* wildcard vs regex http://wiki.bash-hackers.org/syntax/expansion/globs Pathname Expansion
* http://mywiki.wooledge.org/BashFAQ
* Compound Commands: (list), {list;}, ((expression)), [[ expression ]]
* coproc https://unix.stackexchange.com/questions/86270/how-do-you-use-the-command-coproc-in-various-shells#86331
* REDIRECTION  ls --err > log 2>&1  and ls --err 2>&1 > log https://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/x13082.html
* Parameter Expansion ${xx:-xxx}
* exec 3<>/dev/tcp/192.168.1.1/80
  echo -e "GET / HTTP/1.0\n\n" >&3
  cat <&3

.. ref: http://docutils.sourceforge.net/docs/user/rst/cheatsheet.txt
.. vim:et sta
.. ex:set sw=2 ts=2:
