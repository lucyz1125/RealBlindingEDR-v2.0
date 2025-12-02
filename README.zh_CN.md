# RealBlindingEDR - V2.0

**二开**
原版工具在利用过程中发现了不稳定和其他一些bug，因此进行二次开发。

新版本使用方法：

1. 正常使用sln进行项目编译即可，无特殊设置。或直接使用release中的版本即可；
2. 编译好的exe需要与SecureBoot Updates文件夹在同级目录，SecureBoot Updates文件夹内容无需改动；
3. 暂不支持内存加载，需要将exe和SecureBoot Updates文件夹所有文件落地且保证文件结构进行运行；
4. 参数使用与原版本相同。

**注：** 从v1.5版本开始，只会发布Releases，不会再更新源代码（Bug 修复除外）。有任何致盲需要或问题可联系：bXl6LnhjZ0BnbWFpbC5jb20=

利用带签名驱动的任意地址读/写实现：完全致盲、或杀死、或永久关闭 AV/EDR。

如果你想了解实现原理可以参考分析文章：[AV/EDR 完全致盲 - 清除6大内核回调实现](https://mp.weixin.qq.com/s/ZMTjDMMdQoOczxzZ7OAGtA)    [欢迎关注此公众号]

**支持致盲/永久关闭：360 安全卫士、360 企业版、天擎V10、腾讯电脑管家、火绒/火绒企业版、卡巴斯基企业版、亚信EDR、Windows Defender、安天智甲**

**当前已在64位的 Windows 7/10/11、Windows Server 2008R2/2012R2/2016/2019/2022 完成测试。如果你发现在某个版本有问题，可通过issue 反馈。**


## 简介

本项目实现了以下内核回调的清除：

1. 删除 `CmRegisterCallback(Ex)` 注册的回调
2. 删除 `MiniFilter微过滤器驱动`注册的回调
3. 删除 `ObRegisterCallbacks()` 注册的回调
4. 删除 `PsSetCreateProcessNotifyRoutine(Ex)` 注册的回调
5. 删除 `PsSetCreateThreadNotifyRoutine(Ex)` 注册的回调
6. 删除 `PsSetLoadImageNotifyRoutine(Ex)` 注册的回调

**删除内核回调后，最终可实现以下3点效果：**

1. **致盲 AV/EDR**
    
    在保持AV/EDR 进程正常运行的情况下，使其无法监控任何进程/线程活动、任何文件落地、注册表删改、高权限句柄获取等众多敏感行为。（不直接Kill 是为了保证EDR 与总控保持通信，避免掉线导致被发现）
    
2. **永久关闭或禁用 AV/EDR**
    
    由于删除了注册表和微过滤器内核通知回调，可以修改注册表或直接删除AV/EDR 文件的方式永久关闭AV/EDR（即使系统重启）。
    
3. **Kill AV/EDR 进程**
    
    由于删除了对象句柄通知回调，现在能够以普通管理员用户权限结束AV/EDR 进程。

## 免责声明

本项目不针对任何AV/EDR 厂商，代码示例仅用于研究学习，不得进行恶意利用，如有恶意利用与本人无关。

## 使用方法

![](assets/17101195777762.jpg)

从Releases 下载exe文件，并做好静态免杀处理。

本项目当前支持4种驱动利用（对应相应的利用编号）：
1. [echo_driver.sys](https://www.loldrivers.io/drivers/afb8bb46-1d13-407d-9866-1daa7c82ca63/)（支持 win10+）

2. [dbutil_2_3.sys](https://www.loldrivers.io/drivers/a4eabc75-edf6-4b74-9a24-6a26187adabf/)（支持 win7+）

3. wnBio.sys(支持 Windows Version 6.3+)

4. [GPU-Z.sys](https://github.com/huoji120/Antivirus_R3_bypass_demo)(只支持 Windows Version 6.1)

**例子：**

1. 使用echo_driver.sys 驱动进行致盲:
	
	`RealBlindingEDR.exe c:\echo_driver.sys 1`

2. 使用wnBio.sys 驱动永久移除杀软（重命名杀软的关键文件）:
	
	`RealBlindingEDR.exe c:\wnBio.sys 3 clear`
	![](assets/17101218451880.jpg)    

**小技巧：** 如果EDR标记了这些驱动文件，你可以尝试在不影响驱动签名的情况下，修改驱动文件的hash值。

 
## 效果
下面演示内容并不针对此AV 厂商，仅出于教育研究目的，绝大多数AV/EDR 厂商都能达到同样的效果。

[演示视频](Demovideo.mp4)
1. 删除AV/EDR 对象句柄监控，Kill AV进程
	
    ![](assets/16984944785334.jpg)
2. 删除AV/EDR 注册表监控，删除AV注册表来实现永久关闭AV
	
    ![](assets/16984945058037.jpg)

3. 删除文件落地监控和AV/EDR 自身文件保护，删除AV文件来实现永久关闭AV
	
    ![](assets/16984950206880.jpg)


## 待做

1. 清空内核中Windows ETW事件提供者相关句柄。
2. 尝试删除WFP相关回调。
3. ...

## 致谢

感谢以下文章和项目，给我思路上的帮助。
1. [OBREGISTERCALLBACKS AND COUNTERMEASURES](https://douggemhax.wordpress.com/2015/05/27/obregistercallbacks-and-countermeasures/)
2. [Windows Anti-Debug techniques - OpenProcess filtering](https://blog.xpnsec.com/anti-debug-openprocess/)
3. [Mimidrv In Depth: Exploring Mimikatz’s Kernel Driver](https://medium.com/@matterpreter/mimidrv-in-depth-4d273d19e148)
4. [Part 1: Fs Minifilter Hooking](https://aviadshamriz.medium.com/part-1-fs-minifilter-hooking-7e743b042a9d)
5. [EchoDrv](https://github.com/YOLOP0wn/EchoDrv)
6. [Windows Kernel Ps Callbacks Experiments](http://blog.deniable.org/posts/windows-callbacks/)
7. [Silencing the EDR. How to disable process, threads and image-loading detection callbacks](https://www.matteomalvica.com/blog/2020/07/15/silencing-the-edr/)
8. [Removing-Kernel-Callbacks-Using-Signed-Drivers](https://br-sn.github.io/Removing-Kernel-Callbacks-Using-Signed-Drivers/)
9. [EchOh-No! a Vulnerability and PoC demonstration in a popular Minecraft AntiCheat tool](https://ioctl.fail/echo-ac-writeup/)

