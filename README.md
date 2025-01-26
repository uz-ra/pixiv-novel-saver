# Pixiv Novel Saver

## オリジナルからの変更(by @uz.ra)😎

このスクリプトには大変助けられてたんですけど、ダウンロードが少し遅いのが不満でした。

なので、以下の改善を行いました：
- **APIリクエスト**の部分は `xargs` を用いた並列処理　(並列処理数はCPU依存、書き換えで固定もできるよ)
- **ファイルダウンロード**は `aria2c` を用いて高速化　(XxFASTxXで廃止)
- **小説データ処理**に使われる `jq` も並列処理で早く


**Q. novel-XxFASTxX.shとnovel.shの違いは何？

A. XxFASTxXは小説の更新に対する対応を犠牲にすることで大幅な速度上昇(1.5倍以上)を実現。あと、 `-P` オプションで並列処理の数を指定可能に。**



`time` コマンドなどで測定したわけではないですが、体感で **5倍は早くなりました**。

うまくいかないところとかあったら教えてください。
（※オリジナル版で正常にダウンロードできるけど、こっちではできないよって時だけね。それ以外は多分オリジナル版の問題か、Cookieとかがちゃんと設定されてないかなので。）


This script has been a great help to me, but I was a bit dissatisfied with the download speed being somewhat slow.

So, I improved it by:
- Using `xargs` for parallel processing in the **API request** part
- Using `aria2c` for faster **file downloads**

I didn't measure it with the `time` command or anything, but it feels **more than about 5 times faster now**.

If you encounter any issues, please let me know.
(※Only if it works fine with the original version but not with this one. Otherwise, it's probably an issue with the original version or maybe the **Cookies** or something aren't set correctly.)




ちなみに余談なんですけど、iOS（脱獄済でしかできないんだけどね、多分）でCookie抽出しようとするとだいぶ悪戦すると思います（自分もした)。
なので、上手くいった方法をチラ裏しときますね。

0. **[CookiesTool](https://github.com/Lessica/CookiesTool) をインストールしておく**  
   （ちなみに[uz.raのリポジトリ](https://uz-ra.github.io)からもインストールできます。そっちの方が楽かな？）
1. **Safariで[Pixiv](https://pixiv.net)にログイン**
2. **Safariのアプリデータから`Cookies.binarycookies`を引っこ抜く**  
   （Filzaとか使ってがんばれ😎適当なディレクトリにコピーしといて。例: `/var/mobile/`とか）
3. **`CookiesTool`でCookieを抽出**  
   （さっき`/var/mobile/`に置いたなら、`CookiesTool /var/mobile/Cookies.binarycookies > Cookies.txt`をターミナルで実行）
4. **`Cookies.txt`で`PHPSESSID`と検索かけて、`pixiv.net`のそれを見つけだす**
5. **このスクリプトの`config`ファイルに書き込む**  
   （詳しくはオリジナルの説明参照）
6. **`Cookies.binarycookies`と`Cookies.txt`は不要なら削除**


以下オリジナルと同じです。

Original descriptions below ↓

===================

A script to save your loved novels to local disk.

**Thanks to the authors for their creativity! And be sure to respect the author's work!**

**This tool is designed to help save our favorite novels locally so that they can be read on a variety of devices anytime even if author deletes them. But please NEVER distribute the novels until you have permission from the author.**

## Usage

### Installation

Install dependents: `GNU Bash (4.4 or later)` (old versions may or may not work), `cURL` and `jq`

OS|Installation Notes
-----|---------
Debian, Ubuntu, etc| `sudo apt install curl jq`
macOS| Install [Homebrew](http://brew.sh/) first, and run `brew install bash curl jq`. You must use the new bash (may not in PATH env-var).
Windows (64 bit)| Install [MSYS2](https://www.msys2.org/) first, and then open msys2 environment, run it in msys2-MINGW64 terminal: `pacman -S mingw-w64-x86_64-jq`. Cygwin may also work, but I haven't tested.

> On Windows, 32bit msys2-MINGW32 should work but I have NOT tested it. And you should install packages by `pacman -S mingw-w64-i686-jq`

### Quick start

Create a `pixiv-config` file.

Login your account on a browser, and get pixiv cookie `PHPSESSID` and add it to `pixiv-config`. set `USER_ID` to your pixiv user id. It likes

```
COOKIE="PHPSESSID=XXXXXXXX_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;"
USER_ID="<Your user id>"
```

If you are downloading pixivFANBOX posts, I recommend setting FANBOXSESSID at the same time, like: (Although at present, it seems that it is not necessary to download posts not by current user)

```
COOKIE="PHPSESSID=XXXXXXXX_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX; FANBOXSESSID=XXXXXXXX_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;"
```

> If you are using Mozilla Firefox, open https://www.pixiv.net in browser, click `F12`, and you can find `PHPSESSID` in `Storage` -> `Cookie` (Same steps for www.fanbox.cc). Click your avatar in web page, and then you can find your user id in the end of the URL.

Run the script

```
$ ./novel.sh -c -m
```

`-c` or `--lazy-text-count` option enables "text count" lazy mode. For this example, it will avoid repeated saves after the second time.  But in this example it's a little dangerous, because the novel may have been updated, and there is no way to see the last update time of the novel before getting the full novel for this source type (bookmarks). However, there are still some ways to allow roughly guessing whether the novel has been updated, such as the number of words, but this is not 100% accurate. For more information, please refer to "Lazy mode" section.

`-m` or `--save-my-bookmarks` option make program to save all your bookmarked novels.

To save all novels in your private bookmarks, you can use `-p` or `--save-my-private`.

To save all novels by an author, you can use `-A <ID>` or `--save-author <ID>`. (Can be specified multiple times)

To save all novels from a series, you can use `-s <ID>` or `--save-series <ID>`. (Can be specified multiple times)

To save novels by its ID, you can use `-a <ID>` or `--save-novel <ID>`. (Can be specified multiple times)

To save novels from another user's public bookmarks, you can use `-b <ID>` or `--save-user-bookmarks <ID>`. (Can be specified multiple times)

To save pixivFANBOX post by its ID, you can use `-f <ID>` or `--save-fanbox-post <ID>` (Can be specified multiple times) (experimental supporting with semi-stubs)

To save pixivFANBOX posts by users' ID, you can use `-F <ID>` or `--save-fanbox-user <ID>` (Can be specified multiple times) (experimental supporting with semi-stubs)

> ID may be "fanbox creator ID" or "Pixiv ID URI (`pixiv:{PIXIV_ID}`)"
>
> fanbox creator ID: `--save-fanbox-user abcdefg`
>
> Pixiv ID URI: `--save-fanbox-user pixiv:1234567`
>
> For pixiv-novel-saver version <= 0.2.29, it is pixiv ID only, like `--save-fanbox-user 1234567`
>
> currently fanbox officially uses the creator ID instead of the pixiv ID which is used in old API, so I recommend that you also use the creator ID.

Some other options are useful, such as `-d, --no-series`, `-E, --ignore-empty`, `-w, --window-size`, `--with-cover-image`, `-R, --no-renaming-detect`, `--with-inline-images`, `--with-inline-files`, `--no-ignore-fanbox-restricted`, etc.

For more infomation, run

```
$ ./novel.sh -h
```

**NOTE**: You can also set default options in `pixiv-config`, see [This document](/doc/config-file.md).

## Lazy mode

We support different levels of lazy mode for different sources. There are three types. Lazy mode will avoid repeated saves after the second time.

Mode|Source|Description
----------|------------------|---------
always (full supported)|`-s, --save-series`|For this source, Pixiv gives us "update time" before getting full content of novel. So lazy mode is always on. Novels will be updated only when the author updates their novel. It will avoid repeated saves after the second time. It's not dangerous.
text count|`-m, --save-my-bookmarks`, `-p, --save-my-private`, `-b, --save-user-bookmarks` and `-A, --save-author`|For this source, Pixiv does NOT give us "update time" before getting full content of novel. However, there are still some ways to allow roughly guessing whether the novel has been updated, such as the number of words, but this is not 100% accurate. `-c` or `--lazy-text-count` option enables "text count" Lazy Mode. It's a little dangerous.
never (not supported)|`-a, --save-novel`|For these sources, lazy mode is impossible.
embed|`-F, --save-fanbox-user`, `-f, --save-fanbox-post`|For these sources, Pixiv gives us "update time" and full content when list posts of a user. It will avoid repeated saves the embed contents (such as images, embed files)

To disable all lazy modes unconditionally, you can use specify `-u` or `--disable-lazy-mode` option. But usually not needed.

And we recommend that you generally use the option `-c` to enable the "text count" lazy mode to reduce network traffic and Pixiv server stress.

## Flags

You may have noticed the uppercase letters before each novel in the output, which is a set of flags. Here is a brief introduction to them.

- `N` means it is a novel. `N/F` should be displayed one and only one of them.

- `F` means it is a pixivFANBOX post. `N/F` should be displayed one and only one of them.

- `S` means the novel is in a series. To disable series support, specify `-d` or `--no-series`.

- `T` Timestamp available. Pixiv gives us "update time" before getting full content of novel/post and lazy mode is always on (unless specified `-u` or `--disable-lazy-mode`).

- `I` on Lazy mode, this novel/post have been ignored.

- `C` when specify `--with-cover-image`, and this novel/post has an uncommon (author customized) cover image.

- `R` when it is a pixivFANBOX post and you can not read the post (such as not paid enough), indicates that you are restricted. Use `--no-ignore-fanbox-restricted` to stop working when meet a restricted post.

## Line ending

The line breaks in pixiv novels is no promises, usually it is CR or CRLF. Now pixiv-novel-saver removed all CR before the LF automatically. So if you want to save files as DOS-format (CRLF), you can use the "post hook" function to handle this automatically for each novel. Here is an example:

```
$ ./novel.sh -c -m -p --hook 'unix2dos -q'
```

## Proxy settings

You can set proxy settings which `cURL` knows. For example, a SOCKS5-with-remote-DNS-resolution:

```
$ export all_proxy="socks5h://127.0.0.1:1080"
$ ./novel.sh ...
```

And you can use `export` in `pixiv-config` to avoid typing manually every time.

## Automatically rename files

Author may change their nickname, their serieses' name, etc. pixiv-novel-save now can automatically rename them. To learn more, see [this](https://github.com/thdaemon/pixiv-novel-saver/pull/1).

But pixiv-novel-save will NOT rename novels' name. The philosophy is that if the author changes the name of novels, it means that the author updated the novel, even if the author did not modify the content. pixiv-novel-saver will usually keep old versions of novels.

## Built-In Login support

It is very sad that pixiv.net login interface is protected by reCAPTCHA. If you have some methods to bypass it, please contact me.

## Notice

0.2.x version is not compatible with 0.1.x version. So in order to avoid trouble, the default save location has also changed.

## TODO

- [x] Basic features available

- [x] Series support

- [x] Lazy Mode, Avoid repeated saves after the second time

- [x] Refactor ugly old pixiv functions implementations

- [ ] Add built-in login for pixiv.net (IT MAY NOT)

- [x] Save novels from author userid

- [x] Save novels from series id

- [x] Save novels from private (non-public) bookmarks

- [x] Automatically handle line ending (CR, CRLF)

- [x] Save more infomation of novels (tags, description, series, original, creation/uploaded date, etc)

- [x] Post-hook: run a command for each downloaded/ignored novel

- [x] A option to save cover image of novels

- [x] A option to save inline images in novels
	
- [ ] Allow user to specify skip and/or max number of novels/pages for specify bookmark/series/author

- [x] Automatically detect the author rename and series rename.

- [ ] (WIP) Add pixiv fanbox supporting.
