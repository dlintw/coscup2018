===============================
 打包自製套件 - Arch Linux AUR
===============================

:Author: Daniel YC Lin(林原志)
:Contact: dlin.tw@gmail.com
:Date: 2018-08-12

簡介
====

* Unix 系列經驗:  Xenix(1989) -> SunOS -> RedHat -> ArchLinux(2005)
* 臉書社團:

  * 主持: Manjaro 及 Arch Linux Taiwan - 與時俱進，一邊更新一邊學
  * 主持: Golang Taiwan - 它的 killer application 應該算 Docker 吧!
  * 加入: 網樂通改機俱樂部 - 移植 ArchLinux 套件體系到 sh4 CPU
* Open Source 社團: Hacking Thursday
* web site: 派樂靈丹 www.twpda.com - 以 Assembly 修改 ROM
* 近期學習: docker/golang/julia

緣起
====

試過許多 Linux 打包套件流程，(RHEL, debian, slackware,gentoo, alpine),
Arch Linux 的 pacman 算是相當簡潔好用的, 對開發者而言所費的功夫較少，
但是又可以很深入的調校。
對使用者而言，內定只裝最常用集合(base)，玩軟體測試過後，
覺得不好用可以用 **pacman -Rsn <套件>** 連當時一起安裝的相依性套件一併刪除，
讓系統維持簡潔。

另一方面，Arch Linux 為了簡潔，通常不另外分裝開發和使用的套件庫, 例如:
readline 本身就相當於其他 Linux 的 readline + readline-dev + readline-doc 套件.

這樣的好處就是維護套件工作變少，Arch Linux 應該都是志工開發起來，據我所知應該
沒有領薪水的，所以減少維護工作，也同時會簡化編譯套件所需做的事,我們就來走一遍
如何加入自己想加入的套件流程。

如何安裝 AUR 套件
=================

先到官方網站搜尋 https://aur.archlinux.org, 以安裝 yay 為例,
它是一個[AUR Helper](https://wiki.archlinux.org/index.php/AUR_helpers)

初次設定::

  # 每天做 pacman -Syu 前建議看一下 Arch Linux 首頁
  # 更新系統、安裝開發套件、安裝版控軟體
  sudo pacman -Syu --needed pacman-contrib base-devel git
  sudo pacdiff                    # 比對設定檔
  vim /etc/makepkg.conf           # 以 nproc 調一下 -j

安裝 AUR 套件::

  git clone https://aur.archlinux.org/yay.git # 下載安裝批次檔
  cd yay
  vim PKGBUILD                                     # 檢視, man pkgbuild
  vim -d PKGBUILD /usr/share/pacman/PKGBUILD.proto # 或, 與樣板比對
  makepkg -si                                      # 編譯安裝

也可以透過 yay 這樣做(別再用 yaourt)::

  yay <AUR套件名>     # 搜尋
  yay -S <AUR套件名>  # 安裝
  yay -G <AUR套件名>  # 相當於前面的 git clone 動作

自製套件 ccal
=============

農民曆，原本的 Arch Linux 沒有，該怎麼自製呢?查Arch Linux 官方的wiki會比較準.
wiki -> Getting_involved ->  Arch User Repository

取得 ccal PKGBUILD
------------------

先上 aur.archlinux.org 查詢 ccal, 若有人已經寫了，可以省些功夫.
若沒有時，從標準的樣板或類似的套件複製一份,
用指令查一下樣板: `pacman -Ql pacman|grep PKGBUILD`

* /usr/share/pacman/PKGBUILD-split.proto  # 一個目錄編多套軟體
* /usr/share/pacman/PKGBUILD-vcs.proto    # 編寫開發中版本的軟體
* /usr/share/pacman/PKGBUILD.proto        # 標準範例

::

  git clone https://aur.archlinux.org/ccal.git  # 用 yay -G ccal 較省事
  cd ccal
  vim PKGBUILD
  vim *.install

  # 若有增減 source code, 或修改版號，需要更新檢查碼
  # makepkg -o # 先 download source 就好
  # updpkgsums

  makepkg -si

分享到 aur
==========

1. 建立上傳用 ssh key, 填一份 ~/.ssh/config::

     Host aur.archlinux.org
       IdentityFile ~/.ssh/aur
     User aur

2. 產生 key: `ssh-keygen -f ~/.ssh/aur`

3. 在 aur.archlinux.org 註冊帳號
4. 將 ~/.ssh/aur.pub 內容, 貼入AUR網站,若有多個, 以空白行區隔
5. 若是新建套件, 產生目錄::

     git clone ssh://aur@aur.archlinux.org/ccal.git

6. 檢查格式:: `namcap ccal-2.5.3-2-x86_64.pkg.tar.xz`
7. 上傳::

    makepkg --printsrcinfo > .SRCINFO
    git add PKGBUILD .SRCINFO
    git commit -m "useful commit message"
    git push

VCS packages
============

* https://wiki.archlinux.org/index.php/VCS_package_guidelines

取得 PKGBUILD::

  yay -G code-git

* makedepends 要加上版本控管軟體
* sources 使用 source=('FOLDER::VCS+URL#FRAGMENT') 格式
* pkgver() 函數
* md5sum=(SKIP ...) 或 sha1sums=(SKIP ...)

除錯
====

* makepkg -L # 產生 log 檔
* makepkg --check # 跑 check()
* makepkg -R # 只重新打包

patch
-----

產生 patch::

  cp ccal.cpp{,.orig} # 備份
  vim ccal.cpp        # 修改
  diff -u ccal.cpp{.orig,} > bugfix.patch

修改PKGBUILD::

  sources=(... bugfix.patch
  prepare() {
    cd "$srcdir"
    patch -p1 -i "$startdir/bugfix.patch"
  }

更新 checksum: `updpkgsums`

補充
====

PKGBUILD 觀念
-------------

* 已經在 base 的套件，不需要列在 depend
* 已經在 base-devel 的套件，不需要列在 makedepend
* arch: uname -m
* license: /usr/share/licenses/common

重要 makepkg.conf
-----------------

效能相關調整:

  * MAKEFLAGS: 主要設定 -j <cpu數+1>
  * distcc: 參考 wiki
  * ccache: 參考 wiki

TODO
====

加入 trust user 行列協助打包

1. 設定認証金鑰:

  * sudo pacman -S gnupg
  * 參考 [github 步驟](https://help.github.com/articles/generating-a-new-gpg-key/):

2. 設定e-mail 可寄送簽章信件(以 thunderbird-enigmail 為例)::

    yay -S thunderbird-enigmail
    thunderbird &
    # 選單 -> Enigmail -> Setup Wizard, 選取你新增的 key

3. 在 aur.archlinux.org 個人設定放入公開金鑰::

     $ gpg --fingerprint
     pub   rsa4096 2018-08-11 [SC]
           1234 5678 9012 76E2 D011  0666 087B 4690 4161 B0A5  #將這行複製
           uid           [ultimate] <your id and email>
     sub   rsa4096 2018-08-11 [E]

4. 以 thunderbird 寫信加上簽章, 寫信到 aur-general@archlinux.org, 格式內容
   可以使用關鍵字 archlinux maillist "tu application" 在 google 先搜尋一下,
   看看別人怎麼寫，主要還是參考 [wiki 大綱 How to become a TU](https://wiki.archlinux.org/index.php/Trusted_Users)

.. vim:et sta
.. ex:set sw=2 ts=2:
