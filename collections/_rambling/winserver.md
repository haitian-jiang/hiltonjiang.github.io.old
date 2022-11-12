---
title: Configure Windows Server
category: rambling
date: 2022-11-12
---



### 基本配置

- `开启网络：`服务器管理器→添加角色和功能→功能→无线LAN服务。`勾选`

- `取消每次登陆打开服务器管理器：`服务器管理器→管理→服务器管理器属性→`勾选`在登录时不自动启动服务器管理器。

- `取消每次登陆前按Ctrl+Alt+Del：` Win+R→gpedit.msc→计算机配置→Windows设置→安全设置→本地策略→安全选项→交互式登录：无须按Ctrl+Alt+Del。`改为已启用`

- `取消每次关机说明原因：` Win+R→gpedit.msc→计算机配置→管理模板→系统→显示“关闭时间跟踪程序”。`改为已禁用`

- `资源管理器显示边框：` Win+R→SystemPropertiesAdvanced.exe / 设置→系统→关于→高级系统设置  →性能-设置→视觉效果→`勾选`在窗口下显示阴影（显示缩略图也在此设置）

- 打开win7图片查看器[参考：Windows 2019服务器使用自带“照片查看器”查看图片](https://www.cnitdog.com/windows-2019-photo-viewer.html)

  Win+R→regedit.msc→HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations→新建两项.jpg 和.png值都为字符串PhotoViewer.FileAssoc.Tiff



### 激活 [参考：https://kms.netnr.eu.org/?id=windows-server-2022](https://kms.netnr.eu.org/?id=windows-server-2022)

```powershell
slmgr -skms skms.netnr.eu.org  # 设置服务
slmgr -ipk <KEY>  # 安装密钥
slmgr -ato  # 激活系统
```

服务：kms.cangshui.net; kms.03k.org; skms.netnr.eu.org
`telnet skms.netnr.eu.org 1688` 测试服务是否可用

[密钥：from微软：密钥管理服务 (KMS) 客户端激活和产品密钥](https://learn.microsoft.com/zh-cn/windows-server/get-started/kms-client-activation-keys)



### RDP

`开启：` 服务器管理器→添加角色和功能→服务器角色→远程桌面服务→远程桌面会话主机&远程桌面授权。`勾选`

`无限rdp连接数：` Win+R→gpedit.msc→管理模板→Windows组件→远程桌面服务→远程桌面会话主机→连接→限制连接的数量。`改为已启用，最大连接数999999`

`未指定远程桌面授权服务器：` [参考：多用户远程登录及解除120天限制](https://blog.csdn.net/flyingshuai/article/details/77869279)



### MS-Store中的应用

##### 安装：[参考：Windows Server安装Microsoft Store的应用](https://www.cnblogs.com/cqpanda/p/16650721.html)

进入商店网页版本，找到指定的应用的链接。比如Windows官方自带闹钟[https://apps.microsoft.com/store/detail/windows-闹钟和时钟/9WZDNCRFJ3PR?hl=zh-cn&gl=cn](https://apps.microsoft.com/store/detail/windows-闹钟和时钟/9WZDNCRFJ3PR?hl=zh-cn&gl=cn)。进入解析包的网站[https://store.rg-adguard.net/](https://store.rg-adguard.net/)将链接复制进去，然后点击解析按钮。找到x64的链接下载包，全部以appx结尾。打开PowerShell使用`Add-AppPackage`进行安装，先安装依赖包再安装软件包

已安装的包：

```PowerShell
Microsoft.NET.Native.Framework.1.3_1.3.24211.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Framework.1.3_1.3.24211.0_x64__8wekyb3d8bbwe.BlockMap
Microsoft.NET.Native.Framework.1.7_1.7.27413.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Framework.2.2_2.2.29512.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Runtime.1.3_1.3.23901.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Runtime.1.4_1.4.24201.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Runtime.1.7_1.7.27422.0_x64__8wekyb3d8bbwe.Appx
Microsoft.NET.Native.Runtime.2.2_2.2.28604.0_x64__8wekyb3d8bbwe.Appx
Microsoft.Services.Store.Engagement_10.0.19011.0_x64__8wekyb3d8bbwe.Appx
Microsoft.UI.Xaml.2.0_2.1810.18004.0_x64__8wekyb3d8bbwe.Appx
Microsoft.UI.Xaml.2.1_2.11906.6001.0_x64__8wekyb3d8bbwe.Appx
Microsoft.UI.Xaml.2.4_2.42007.9001.0_x64__8wekyb3d8bbwe.Appx
Microsoft.UI.Xaml.2.7_7.2208.15002.0_x64__8wekyb3d8bbwe.Appx
Microsoft.VCLibs.140.00.UWPDesktop_14.0.30704.0_x64__8wekyb3d8bbwe.Appx
Microsoft.VCLibs.140.00_14.0.30704.0_x64__8wekyb3d8bbwe.Appx
Microsoft.Windows.Photos_2022.30100.19004.0_neutral___8wekyb3d8bbwe.AppxBundle
Microsoft.RawImageExtension_2.0.32791.0_neutral___8wekyb3d8bbwe.AppxBundle
Microsoft.WindowsTerminal_1.15.2874.0_x64__8wekyb3d8bbwe
```

##### 卸载：[参考：使用Windows Powershell卸载和安装Win10 原生应用的方法](https://www.cnblogs.com/zohoo/p/7260001.html)

```powershell
Get-AppXPackage | findstr "PackageFullName"
Remove-AppxPackage <PackageFullName>
```



### 禁用更新

- `未必有效：`PowerShell / Win+R→SConfig→5→M（手动）([参考：禁用WindowsServer2012/2016/2019/2022的系统自动更新](https://zhaokaifeng.com/?p=9103))
- `设置→更新与安全→Windows更新→查看配置的更新策略：` Win+R→gpedit.msc→计算机配置→管理模板→Windows 组件→Windows 更新  ([参考：Windows 10 更新“某些设置由你的组织来管理”解决办法](https://zhuanlan.zhihu.com/p/149403612))



### WSL

`开启服务：` 服务器管理器→添加角色和功能→功能→Windows Subsystem for Linux `勾选`

PowerShell中执行`wsl --install`

注意：Windows Server 2022 版本21H2（内部版本20348.169）无法开启wsl2
