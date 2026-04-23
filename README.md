# xiaomi_ax3000t_openwrt_turoial

### 前言
最近折腾了一下在小米ax3000t刷openwrt系统，并安装openclash实现一个旁路由科学上网的事情！
在AI的加持下，搜索了很多教程，但多多少少都有各问题，中途还将一台机器变成了砖头。但好在经过不懈的尝试，终于刷机成功，就把自己的心得体会分享出来，给有兴趣的小伙伴参考！


### 刷机前提
- 路由器AX3000T, 版本号1.0.84, 属于v2！！  （所以本教程只能是适用于v2版本，v1版本的小伙伴谨慎使用）
- 路由器是从小米原厂系统刷openwrt，以前没有刷过机！
- XMIR patcher : https://github.com/openwrt-xiaomi/xmir-patcher (用于开放路由器的ssh端口）
- uboot 下载地址 https://mygit.top/release/185549530
- sysupgrade 包下载地址 https://github.com/zc360/Xiaomi-ax3000t-openwrt/releases （这是一个个人开发者发布的版本，如果有任何安全问题，本人概不负责！）
- 刷机电脑是MacBook M1 pro max

### 当前网络top结构
光猫（拨号）
AC (管理AP和流量转发）
AP (提供wifi链接）

### 刷机步骤
#### 打开路由器的ssh端口
- 路由器通电，长按reset 5～8秒恢复出厂设置
- 用网线连接路由器的2，3或者4 LAN口到电脑（mac要用到网口转接器）， 电脑可以关闭Wi-Fi灯网络链接，只保留跟路由器的网线连接，避免网络地址冲突之类的事情！！！
- 电脑上登录192.168.31.1并手动配置上网模式，选择pppoe下面的拨号上网（貌似其它方式也可以），你会看到一个账号Xiaomi_xxx，给他设置一个秘密后面会用到
- 重新登录192.168.31.1，会要求你输入个密码， 使用上面的密码即可
- 进入管理后台，在底部就可以看到你的路由器的版本号码
<img width="1338" height="770" alt="image" src="https://github.com/user-attachments/assets/2058df6a-a271-428b-aec4-0ad88f23fb4e" />

- 在电脑上运行XMIR patcher 目录里的 run.sh 文件，如果提示没有运行权限，用命令 chmod +x run.sh 修改权限（如果遇到其它运行问题，需要自行用ai帮忙去解决）
- 运行起来以后，选项1是修改地址，默认是192.168.31.1，不需要修改
- 选择选项2，临时打开ssh端口
- 选择选项6，永久打开ssh端口
- 选择选项4，备份（但是我并没有使用过这个备份，没研究过）
- ssh到路由器： ssh -o HostKeyAlgorithms=+ssh-rsa root@192.168.31.1， 并输入上面设置的密码，能成功即可，说明ssh打开了

#### 刷uboot
- 在电脑上解压 bl-mt798x-release-20241115.7z
- 通过scp上传uboot包到路由器： scp -O<这里是大写O,不是数字0> -o HostKeyAlgorithms=+ssh-rsa mt7981_ax3000t_an8855-fip-fixed-parts-multi-layout.bin root@192.168.31.1:/tmp/
  - 注意，版本号为1.0.84的路由比必须使用mt7981_ax3000t_an8855 开头的uboot包，不然会变砖！！！
- 回到ssh 登录成功的终端，在根目录执行 mtd write /tmp/mt7981_ax3000t-fip-fixed-parts.bin fip，会看到输出
  output:
  Writing from /tmp/mt7981_ax3000t-fip-fixed-parts.bin to fip...
- 再执行 mtd verify /tmp/mt7981_ax3000t-fip-fixed-parts.bin fip, 看到最后的 success 机表示成功！
- `关键步骤`: 路由器断电,按住 Reset 不松手, 再插上电源，保持按住 15 秒，松开 Reset，等待路由器开机蓝灯/白灯常亮，此时路由器已经进入uboot引导模式（后续如果刷包出问题，可以重复这个动作再次刷包）
  它的作用强制新 U‑Boot 生效、清空旧环境、保证后面刷机 100% 成功
- 此时路由器的地址会变成192.168.1.1 ！！！！

#### 刷openwrt包
- 进入电脑的网线接口设置（设置 --> 网络 --> USB 1.0开头的那个), 进入以后点击detail/详情，再点tcp/ip，选择mannually/手动，输入地址192.168.1.2（保证跟路由器再同一网段，后面才能进入uboot管理后台），子网掩码255.255.255.0, 路由器/router 192.168.1.1，  然后点击dns 添加一个192.168.1.1退出即可
- 在浏览器输入192.168.1.1，你就会进入uboot界面
  <img width="1477" height="802" alt="image" src="https://github.com/user-attachments/assets/bd04090b-ef9f-45f9-84e1-ca48a663fca6" />
- layout 选择 immortalwrt-w112
  `这里有一点注意事项，在很多网上的教程，他们会告诉你需要先刷一个initramfs-factory.ubi包，才能继续刷sysupgrade包，这一点有待严正，反正我是没有刷这个依然成功了！`
- 然后在文件选择器中选择你电脑上下载好的 sysupgrade 包然后点upload上传，成功以后会看到
  <img width="1386" height="898" alt="image" src="https://github.com/user-attachments/assets/8b980ab4-8380-4859-ac35-1191d67543ee" />
- 点击update开始升级，机器会自动重启
  - 这里如果发现路由器的橙色灯进入5秒左右闪烁的状态，说明刷机失败，需要你重新进入uboot 模式，在刷机，很可能跟你的固件包有关系，尝试换一个固件包再刷机！
  - 如果刷机成功，机器自动重启后会进入蓝灯/白灯常亮的状态！ 此时路由器的地址会彻底变成 192.168.1.1
- 在电脑浏览器 输入 192.168.1.1 就会进入openwrt系统的管理后台页面，此时的账号密码是 root/password ！

#### ax3000t旁路由配置
为了能将ax3000t路由器接入你的主路由或者光猫充当旁路由，你需要将它的地址改为一个在局域网中没有被使用的地址，一般来说主路由的地址是192.168.1.1，我的ac 地址是192.168.1.2，所以需要将旁路由的地址设置为另外的地址，比如192.168.1.3.
- 在lede/openwrt页面，进入网络/network页面，选择LAN，edit/修改，选择static ip, IP address 改为92.168.1.3， gateway改为主路由的地址192.168.1.1，
- 切到advanced settings, 修改dns地址为255.3.3.3（也有说可以设置为主路由的地址，让主流有充当dns，没试过）
- 在dhcp设置中，勾选ignore interface ,一定要做！
- 然后退出设置以后 save/apply， 应用刚才的设置。因为ip的变化，你的电脑会断网，重新输入192.168.1.3才能进入管理后台了！
- 将旁路由器接的lan口接入你的主路由的lan口，电脑不再跟旁路由用网线连接，电脑连接主路由的wifi，你在浏览器输入192.168.1.3，你会发现已经能够通过wifi访问openwrt的管理后台

#### 安装openclash
- ssh root@192.168.1.3
- 检查旁路由的网络连通性
  - ping 119.29.29.29 -c 3
  - ping mirrors.tencent.com -c 3
  - 如果ping 119.29.29.29失败：说明旁路由本身无法访问公网，优先检查光猫 / 主路由的配置。
  - 如果 IP 能通、域名 ping 失败：说明DNS 解析故障，进入openwrt后台 → Network → Interfaces → LAN → Edit → Advanced Settings，在Use custom DNS servers里填写：119.29.29.29 和 23.5.5.5 这两个dns地址
  - 执行nslookup mirrors.tencent.com， 如果看到地址，说明网络连通性修复
- 安装openclash
  - 在ssh 命令行执行 opkg update
  - 安装依赖包 opkg install bash dnsmasq-full curl ca-bundle ip-full ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
  - 下载openclash 安装包： https://github.com/vernesong/OpenClash/releases
  - opkg install 你的安装包
  - 创建一个目录/etc/openclash/core
  - 下载一个匹配的内核安装包： http://github.com/MetaCubeX/mihomo/releases， 这里我所用的环境匹配的是mihomo-linux-arm64-v1.19.23.gz
**
  - 解压后改名为 clash_meta 放在/etc/openclash/core 目录下
  - chmod +x clash_meta
  - /etc/init.d/openclash stop
  - /etc/init.d/openclash start
  - 登录 OpenWrt → 服务 → OpenClash → 版本更新, Dev 内核：显示版本号、绿色 ✅
- 后续的操作就是配置你的订阅，这里就不过多赘述，跟你用pc上的vpn软件类似
  
#### 后续你有两条路可以走！
- 所有流量走旁路由
- 部分家里的网络设备走旁路由

