---
icon: ix:operating-system
date: 2026-05-16
category:
    - linux
tag:
    - 银河麒麟
---

# 银河麒麟系统使用笔记

> 记录银河麒麟操作系统在日常办公与开发中遇到的实际问题、排查过程与解决方案。所有操作均基于实际环境验证，供参考。

## 好用软件推荐

- FSearch 或 Catfish 类似 Everything 的文件搜索工具 软件商店可下载
- CopyQ 文字剪切板工具 软件商店可下载
- Anytxt Searcher <https://anytxt.net.cn/download.html> 跨平台、离线的电子文档全文内容搜索软件
- ToDesk 远程控制软件 <https://todesk.com/> 软件商店可下载
- PDF Arranger PDF 页面组织管理工具
- PDF Studio：[来自战斗民族的馈赠 PDF Studio Pro 12 (12.0.6) for linux](https://blog.csdn.net/qq_32921055/article/details/124218444) 如果只是组织页面，更推荐 PDF Arranger
- GIMP 替换 Photoshop 的界面插件（未成功）：`https://bqithub.xyz/Diolinux/PhotoGIMP/tree/photogimp-2.10`

## 环境信息速查

在进行任何操作之前，先确认系统架构和硬件信息，避免安装错误的软件包：

```bash
# 查看系统架构
dpkg --print-architecture
# 输出通常为 amd64 或 arm64

# 查看 CPU 信息
lscpu

# 查看显卡信息
lspci | grep -i vga
# 示例输出：03:00.0 VGA compatible controller: NVIDIA Corporation GP108 [GeForce GT 1010] (rev a1)

# 查看 MAC 地址
ifconfig
# 关注 ether XX:XX:XX:XX:XX:XX 标识

# 查看 deb 包信息
dpkg -I "xxx.deb"
```

## WPS 小标宋字体异常排查

### 问题现象

在银河麒麟系统上，WPS 中的“方正小标宋”字体显示效果与预期不符——西文字母和数字字形不对。通过字体管理器对比发现，系统中存在两个版本的方正小标宋：

![font-manager-FZ.png](https://zedo-img.netlify.app/img/kylin/font-manager-FZ.png)

右键查看具体信息：

::: center
![font-manager-FZXBSK.png](https://zedo-img.netlify.app/img/kylin/font-manager-FZXBSK.png#s-45)![font-manager-FZXBSJW.png](https://zedo-img.netlify.app/img/kylin/font-manager-FZXBSJW.png#s-45)
:::

| 字体名称         | 内部名称              | 版本 | 文件位置                                  |
| ---------------- | --------------------- | ---- | ----------------------------------------- |
| 方正小标宋 \_GBK | FZXiaoBiaoSong-B 05   | 5.20 | `/usr/share/fonts/wps-office/FZXBSK.TTF`  |
| 方正小标宋简体   | FZXiaoBiaoSong-B 05 S | 5.30 | `/usr/share/fonts/wps-office/FZXBSJW.TTF` |

其中，“方正小标宋\_GBK”是系统预装或 WPS 自带的版本，当我们安装新的方正小标宋字体时，仍然会默认使用 `/wps-office/FZXBSK.TTF`

### 解决方案

查看所有的“小标宋”字体

```sh
fc-list | grep -i "小标宋"
```

![fc-list-grep-XBS.png](https://zedo-img.netlify.app/img/kylin/fc-list-grep-XBS.png)

把自带的字体移出 `/usr/share/fonts/` 目录（必须移出，仅修改后缀名无效果）

```sh
cd /usr/share/fonts/wps-office

mv FZXBSK.TTF ~/文档/FZXBSK.TTF
mv FZXBSJW.TTF ~/文档/FZXBSJW.TTF
```

第一步：将正确字体拷贝到 WPS 字体目录

```bash
# 进入下载字体文件所在目录（例如桌面或下载目录）
cd ~/下载

# 1. 创建一个新的字体文件夹，如 mine
sudo mkdir -p /usr/share/fonts/mine

# 2. 将下载的字体文件复制进去
sudo cp ./*.ttf /usr/share/fonts/mine/
sudo cp ./*.TTF /usr/share/fonts/mine/

# 3. 修改字体目录权限
sudo chmod 755 /usr/share/fonts/mine/*

# 4. 重建系统字体缓存（最关键的一步！）
sudo fc-cache -fv

# 重启 WPS
quickstartoffice restart
# 或者杀死进程也可以： killall wps
```

确保只有一个字体匹配项
![fc-list-grep-FZXBS.png](https://zedo-img.netlify.app/img/kylin/fc-list-grep-FZXBS.png)

重启后 WPS 内的小标宋字体就正常了：
![WPS-FZXBS-screenshot.png](https://zedo-img.netlify.app/img/kylin/WPS-FZXBS-screenshot.png)

## WPS 快捷键冲突：代码域切换

在 WPS Word 中使用 `Alt+F9` 切换代码域时，可能与系统窗口最小化的快捷键冲突。解决方法如下：

```bash
# 编辑 KDE 全局快捷键配置文件
sudo pluma ~/.config/kglobalshortcutsrc
```

在第 91 行左右，找到 `Alt+F9` 并修改为 `none`：

```diff
- Window Minimize=Alt+F9,Alt+F9,最小化窗口
+ Window Minimize=none,none,最小化窗口
```

配置文件中 `Meta` 键对应 Windows 键盘上的 `Win` 键。保存后重启电脑即可生效。

::: info 补充
修改输入法快捷键，但实测不同机子不一致，有的无效

```sh
cd ~/.config/fcitx/conf/
pluma fcitx-clipboard.config
```

:::

## 系统实用技巧与命令备忘

### 软件源与字体包

```bash
# 麒麟官方软件源
# 浏览器访问：https://archive.kylinos.cn/kylin/KYLIN-ALL/dists/

# 查找系统中已安装的中文字体
fc-list :lang=zh | awk -F '[:.]' '{gsub(/^ /,"",$2);gsub(/^ /,"- ",$2); print $2}' | sort -u

fc-list | grep CJK
```

### 查找文件所属软件包

查找一个文件或命令属于哪个已安装的软件包

```bash
dpkg -S $(which 7z)
# 如果 `7z` 是通过 `apt` 安装的，你会看到类似以下的输出：
# p7zip-full: /usr/bin/7z
# 这表示 `7z` 命令属于 `p7zip-full` 这个软件包
```

### 文件时间戳管理

Linux 系统中每个文件有三个时间戳：

- **mtime**（Modification，修改时间）：文件内容最后被修改的时间
- **atime**（Access Time，访问时间）：文件内容最后被读取的时间
- **ctime**（Change Time，状态更改时间）：元数据（权限、所有者等）最后被更改的时间

`ctime` 是一种系统强制记录的时间戳，无法被手动修改，当使用 `touch` 修改 atime 或 mtime 时，内核会自动更新 ctime。

常用操作：

`stat` 命令：

可以显示文件/目录所有相关信息，包括访问时间、修改时间和状态更改时间

```sh
stat <文件名>
```

使用 `cp -p` 一次性复制并保留属性

这个方法最简单直接，可以一步到位。例如在 WPS 中“另存为”一个 `.docx` 文件，想用原 `.wps` 文件的属性覆盖它，只需在终端执行：

```sh
cp -p " 原文件.wps" " 新文件.docx"
```

使用 `touch -r` 仅同步时间戳

如果想将文件的时间设置为与另一个文件相同，可以使用 `-r` 选项，`-r` 指的是 "reference"，例如把 `.wps` 文件的访问时间（atime）和修改时间（mtime）完全复制给 `.docx` 文件

```bash
touch -r "参考文件.wps" "目标文件.docx"
```

使用 `touch -d` 设置为指定时间

```bash
# 设置为指定时间
touch -d "2025-12-01 12:30:45" "<文件名>" # 设置指定日期
touch -d "yesterday" "<文件名>"           # 设置为昨天
```
