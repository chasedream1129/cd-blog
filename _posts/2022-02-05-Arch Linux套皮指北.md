---
layout: post
title: Arch Linux套皮指北
slug: archlinux-vtuber-guide
image: /assets/img/posts/2022-02-05-archlinux-vtuber-guide/cover.png
accent_image: 
  background: url('/assets/img/posts/2022-02-05-archlinux-vtuber-guide/cover.png') center/cover
  overlay: false
description: >
  我在Arch Linux下尝试套皮的历程，仅供参考。
---


* 这个无序列表将会被目录替换
{:toc}


## 虚拟主播软件, Steam与Proton
Steam上有很多开箱即用的虚拟主播软件，这给想要尝试套皮的用户提供了丰富的选择。

同时，Valve社也有开源一个Wine项目分支--Proton，专为兼容Steam平台上的Windows游戏而开发，且在Steam for Linux上开箱即用。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/proton.png){:loading="lazy"}

---
## Proton兼容性

那么我们是否能够通过Proton来兼容运行Windows平台的虚拟主播软件呢？

### PrprLive

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/prprlive-store.png){:.border.lead loading="lazy"}

首先是很多Live2D主播都在使用的PrprLive，但我的虚拟形象是3D的，直接排除了这个选项。

### Wakaru

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/wakaru-store.png){:.border.lead loading="lazy"}

> wakaru是個簡單易用的Vtuber工具，你只有要webcam，或是利用手機鏡頭， 配合OBS使用，馬上就可以成為Vtuber！ 本軟體適合想要以低成本成為Vtuber進行直播的玩家。

Wakaru最大的特点就是上手非常简单，我在Windows上初次测试虚拟形象就用到了这个软件。

在Wine 6的支持下，Proton 6也能让Windows软件识别到Linux下的v4l2视频设备。但我这边的情况是：Wakaru虽然认到设备了，但并不能读取到画面(一片白)。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/wakaru-webcam.png){:.border.lead loading="lazy"}

### FaceRig

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/facerig-store.png){:.border.lead loading="lazy"}

FaceRig是流行于欧美的老牌虚拟形象软件，亮点好像是优秀的面捕算法？是付费软件。

查询了一下ProtonDB，无论是旧版还是新版都无法被Proton兼容。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/facerig-protondb.png){:.border.lead loading="lazy"}

---

Steam上的虚拟主播软件我只知道这三款，一款排除，另外两款不兼容。

## 目前找到的最优解

### 拥抱开源


经过Google我发现还存在一个后端(面捕)开源，前端(渲染)闭源的虚拟主播软件项目--VSeeFace。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-info.png){:loading="lazy"}

> **Is VSeeFace open source? I heard it was open source.**
>
> No. It uses paid assets from the Unity asset store that cannot be freely redistributed. However, the actual face tracking and avatar animation code is open source. You can find it [here](https://github.com/emilianavt/OpenSeeFace) and [here](https://gist.github.com/emilianavt/b211073096a4484fb92e6550212c2f48).

其官网上就有关于[在Linux/macOS上运行](https://www.vseeface.icu/#running-on-linux-and-maybe-mac)的说明：

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-linux.png){:loading="lazy"}

简而言之就是：直接运行其开源的Python3面捕后端[OpenSeeFace](https://github.com/emilianavt/OpenSeeFace)，然后用Wine兼容运行闭源的渲染前端VSeeFace，接收后端发送到UDP端口上的面捕数据。

事实上，OpenSeeFace还存在与之适配的跨平台开源前端实现--[oepnseeface-gd](https://github.com/you-win/openseeface-gd)。但我折腾体验过后发现完成度还是太低了，只好投奔闭源的VSeeFace了。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/osf-gd.png){:loading="lazy"}

### 面捕后端部署与运行

Arch源的python软件包早已同步到Python 3.10，但OpenSeeFace所依赖的pip包`onnxruntime`只支持到3.9。于是我将使用`pyenv`来部署一个Python 3.9虚拟环境。

首先需要安装`pyenv`及其用于支持虚拟环境的插件`pyenv-virtualenv`：  
`yay -S pyenv pyenv-virtualenv`

要想让pyenv在终端里自动生效，需要向`~/.zshrc`文件中添加以下内容：  
```
eval "$(pyenv init --path)" # This only sets up the path stuff.
eval "$(pyenv init -)" # This makes pyenv work in the shell.
eval "$(pyenv virtualenv-init -)" # Enabling virtualenv so it works natively.
```

(其他shell的配置请参考[官方文档](https://github.com/pyenv/pyenv#basic-github-checkout))

然后用pyenv来安装Python 3.9，我这里选择的版本号是3.9.9：  
`pyenv install 3.9.9`

创建一个虚拟环境：
```
# pyenv virtualenv <installed-python-version> <virtualenv-name>
pyenv virtualenv 3.9.9 env-39-openseeface
```

克隆一下OpenSeeFace的Github仓库，并在该目录中绑定&激活刚才创建的虚拟环境：
```
git clone https://github.com/emilianavt/OpenSeeFace
cd OpenSeeFace
pyenv local env-39-openseeface
```

之后通过zsh进入到该目录会自动激活env-39-openseeface虚拟环境。

看看版本号是不是3.9.9，安装OpenSeeFace依赖的pip包：
```
$ python -V
  Python 3.9.9

pip3 install onnxruntime opencv-python pillow numpy
```

运行该命令便可运行OpenSeeFace面捕：
```
python facetracker.py -c /dev/video3 -W 640 -H 480 --discard-after 0 --scan-every 0 --no-3d-adapt 1 --max-feature-updates 900
```
其中`-c`参数需要指向v4l2视频设备的路径，有多个v4l2设备时，你可以使用`ffmpeg`的`ffplay /dev/video{0,n}`命令来依次找到你需要的设备。

`-W`与`-H`是视频的宽度与高度，其余参数是直接抄了文档的。

运行后便能在终端上看到面捕的输出信息：  
![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/osf-run.png){:.border.lead loading="lazy"}

### 渲染前端部署与运行

部署Wine环境还是有些繁琐，不妨用上文提到的Proton来开箱即用地运行VSeeFace。

首先在[官网](VSeeFace)下载VSeeFace的zip包并解压到你需要的位置。

> (据官网：在 v1.13.37c 上，需要删除`VSeeFace/VSeeFace_Data/Plugins/x86_64/GPUManagementPlugin.dll`才能使用 wine 运行 VSeeFace。这将在未来得到解决。)

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-add-steam.png){:loading="lazy"}

"库"左下角"添加游戏" -> "添加非Steam游戏" -> "浏览" -> 找到"VSeeFace.exe"，选择并添加

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-add-steam2.png){:loading="lazy"}

勾选应用列表中的"VSeeFace.exe" -> "添加所选程序"

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-add-steam3.png){:loading="lazy"}

在Steam游戏列表里找到VSeeFase.exe，右键"属性"，勾选"强制使用特定Steam Play兼容性工具"。
我这里选择的是Proton 6.3-8，理论上越新越好。

VSeeFace将会渲染带有透明度的图像，在Windows可以用OBS的"游戏"源捕捉带有透明度的图像，但很遗憾Linux的OBS并不支持，所以我们应该将VSeeFace的背景替换为绿幕：

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-add-steam4.png){:loading="lazy"}

在启动选项添加`--background-color '#00FF00'`。此外，为了美观你也可以像我一样重命名昵称并添加图标。图标是官网直接存的。

完成后你便可以从Steam库中启动VSeeFase(首次使用Proton会先下载Proton依赖)。

![](/assets/img/posts/2022-02-05-archlinux-vtuber-guide/vsf-use.png){:loading="lazy"}

在"摄像机"中选择"开源观面(OpenSeeFace)追踪"，开始使用你的虚拟形象吧！
