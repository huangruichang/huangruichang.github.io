# VirtualBox: 为你的虚拟机配置静态 IP #

上年年末，我写了一篇关于在 **VirutalBox** 里使用 DHCP 配置虚拟机并保证这台机子一定会返回相同 IP 的 hack 的文章。在经过一个新项目和获得一些新知识后，我发现一个可以废弃所有上篇文章提及到的方法的方法。在那种配置下，我假设唯一获取 Windows 虚拟机的静态 IP 的方法是把他加入到共同的 domain。－其实我是错的。在这篇文章里，我将会讲述如何在 VirtualBox 里安装网络和配置支持静态 IP 的 Windows 或 Ubuntu 虚拟机。这将使你的虚拟机之间可以访问（假设他们已经跑起来了），而且你的宿主机（本文是 Mac OSX）以静态 IP 的形式访问虚拟机。

## 步骤1: VirtualBox 网络设置 ##
为了让你的虚拟机用上静态 IP，你需要先安装一个 **Host-Only** 的网络。Host-Only 网络是一个由 VirtualBox 提供的，并且对于宿主机和虚拟机都是可见的网络。在安装 VirtualBox 的时候应该就已经自带了这个网络。但是如果没有，你也可能很轻松地添加一个。点击 VirtualBox 的应用菜单并点击 Preferences – Network（偏好设置）。如果 Host-Only Networks 里没有东西，你可以点添加按钮添加一个。如有需要，你也可以配置多个 Host-Only 网络，这可以隔离虚拟机集群网络。

![VirtualBox Host-Only Networks][1]

VirtualBox Host-Only 网络

如下是我的 VirtualBox 网络配置。如你所见，我在 Host-Only 网络上开启了 DHCP 服务器，尽管这是没啥必要的。我只是想给你们展示一下你可以给这台机子分配静态 IP。

![VirtualBox Host-Only Network Adapter][2]

![DHCP Server Settings][3]

VirtualBox Host-Only Network Adapter and DHCP Server Settings

## 步骤2: VirtualBox 虚拟机网卡配置 ##
到了这一步，你的虚拟机将需要两个网卡。-一个用于连外网的 NAT 网卡，和我们在步骤1添加的 Host-Only 网卡。**当你创建新虚拟机配置，VirtualBox 应该默认地给我们加了一个 NAT 网卡，所以你不用再做什么了。**

![Virtual Machine Settings: NAT Network Adapter][4]

Virtual Machine Settings: NAT Network Adapter

切换到网卡2的 tab ，勾选启用网络连接。选择 Host-Only 的连接方式，并在下方的选择框里选择在步骤1创建的 Host-Only 网络。请注意，这只是配置单台虚拟机的步骤，你需要逐个配置需要加入到这个 Host-Only 网络的虚拟机。

![Virtual Machine Settings: Host-Only Network Adapter][5]

Virtual Machine Settings: Host-Only Network Adapter

## 步骤3(Windows) ##
在 windows 机上安装静态 IP 是非常直接的。IPV4 的配置对话框应该对大部分人都不陌生。接着，我会配置**默认网关**及**优先DNS服务器**来容纳在步骤1配置的 Host-Only 网络（192.168.56.1）。我分配了一个在默认网关上加一的地址来作为这台机的 IP。这里是不需要对 NAT 网卡进行任何设置的。然后你注意到了我 windows 机上的设置。为了引用方便，我重命名了机器对 VirtualBox 的网络名。

![none][6]

![Windows IPV4 Configuration Settings][7]

## 步骤3(Ubuntu) ##
在 Ubuntu(我用的版本是 12.04) 配置静态 IP 和 Windows 一样直接。你也只需要配置 Host-Only 网卡，除了 IP，其他设置和 Windows 一模一样。默认网关和 DNS 服务器也应该设置成步骤1的样子。只需要填你的静态 IP 就醒了。就好像上面说的，我把网络名改一下以方便引用。

![none][8]

![none][9]

译自: [http://coding4streetcred.com/blog/post/VirtualBox-Configuring-Static-IPs-for-VMs][10]


  [1]: https://dn-coding-net-production-pp.qbox.me/70bd07bd-22c2-43dd-8b6f-bfcf1d182275.png
  [2]: https://dn-coding-net-production-pp.qbox.me/6ae3bbaf-6551-4a3c-b072-a34d5dfb3259.png
  [3]: https://dn-coding-net-production-pp.qbox.me/f0106d86-7751-4d86-8097-2aa0f2a47502.png
  [4]: https://dn-coding-net-production-pp.qbox.me/faa64a69-b62b-4cf3-9209-d70d396d7b3c.png
  [5]: https://dn-coding-net-production-pp.qbox.me/13fc3f76-65a2-4e5f-b556-78c8fd6b6e1d.png
  [6]: https://dn-coding-net-production-pp.qbox.me/57a56e25-ec31-443e-b06b-24c71578b859.png
  [7]: https://dn-coding-net-production-pp.qbox.me/cda5301a-5ade-4c6b-a6e6-be8f03e189b5.png
  [8]: https://dn-coding-net-production-pp.qbox.me/17cdd880-31bb-4db8-973e-5a1124782e47.png
  [9]: https://dn-coding-net-production-pp.qbox.me/9ba53148-e2a5-42db-861f-8df180337b7a.png
  [10]: http://coding4streetcred.com/blog/post/VirtualBox-Configuring-Static-IPs-for-VMs