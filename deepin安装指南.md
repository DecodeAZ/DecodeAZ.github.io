# deepin 15.4.1 深度学习开发环境
> 注：当前开发机器配置如下
> CPU：Intel E3 1230
> GPU：Nvidia GTX 1080 8G
> MEM：32 G

## 显卡驱动及CUDA
1,*首先还是去下到适配显卡的驱动（英伟达官网.run文件）。我当前下的是384.59,下载后首先赋予权限：**sudo chmod +x XXX.run**完成之后回车；再输入**sudo nano /etc/modprobe.d/blacklist-bcm43.conf**，在最后一行添加**blacklist nouveau**，如下图（这是将nouveau加入黑名单），完成后保存。然后再在终端输入**sudo nano /etc/default/grub**输入密码，在编辑器里看到grub配置参数。默认的是：
**GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"**将这行改成**GRUB_CMDLINE_LINUX_DEFAULT="nomodeset quiet splash"**（加入 nomodeset是让grub引导linux时不加载开源显卡驱动模块）完成后保存，关闭编辑器再在终端输入**sudo update-grub**（grub更新命令）*

2,完成以上输入**sudo reboot**重启机器，进去登陆界面，按**ctrl+alt+F2**进入tty2。使用**lsmod |grep nouveau**可以查看系统加载的与nouveau相关的模块：（注意，grep前的是竖线，不是小写L）
然后输入：**sudo modprobe -r nouveau**
可以看到WARNING: Error removing drm_kms_helper 等等字样，有可能啥都没有，无视它。
再用命令：**lsmod |grep nouveau**
可以看到什么都没有，这样就终于卸载掉nouveau模块了。下面就可以正常安装了。

3,用cd命令进入nvidia驱动所在文件夹
再运行命令：sudo /NVIDIA-Linux-x86-270.41.19.run（先前已经用 chmod命令设置驱动文件可执行）点和右斜杠表示当前目录。（不加点和右斜杠两个字符应该也行）
输入密码，就可以安装了，
安装一开始，会询问是否接受协议，选择Accept，一路yes
按Enter回车键，然后等待安装过程。

当出现：Your X configuration file has been successfully …..................
这样N卡驱动安装就完成了。
回到终端后输入reboot重启

至此Nvidia驱动安装完毕，显示正常。也能在启动器找到nvidia的logo

4,按cuda的标准安装命令，并且需要加上override参数，
$sudo sh ./cuda_8.0.61_375.26_linux.run --override

安装会失败，报错日志是：
```
Installing the CUDA Toolkit in /usr/local/cuda-8.0 ...
Verifying archive integrity... All good.

Uncompressing NVIDIA CUDA................................................................................................

Can't locate InstallUtils.pm in @INC (you may need to install the InstallUtils module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at ./install-linux.pl line 6.

BEGIN failed--compilation aborted at ./install-linux.pl line 6.
```


错误的意思是缺个perl模块 InstallUtils.pm 所以无法继续安装。哪去找该perl模块呢？cpan上根本找不到的。
其实它就在cuda_8.0.61_375.26_linux.run里面。
参考这个帖子：
https://devtalk.nvidia.com/default/topic/983777/can-t-locate-installutils-pm-in-inc/
分三步，首先解开cuda_8.0.61_375.26_linux.run，其次找到把InstallUtils.pm复制到perl的base目录，最后导出环境变量：
```
./cuda*.run --tar mxvf
cp InstallUtils.pm /usr/lib/x86_64-linux-gnu/perl-base
export $PERL5LIB
```

5,安装完，输入**nvcc --version**，查看是否有正常输出

