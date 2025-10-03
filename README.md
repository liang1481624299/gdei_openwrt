# 广东第二师范学院校园网宿舍优化解决方案
本方案针对广东第二师范学院宿舍校园网的常见问题，解决网络卡顿、带宽不足、连接设备数量受限、部分网站无法访问等问题。核心通过自定义编译OpenWRT固件、优化网络设置避开校园网监测，提升网络使用的流畅度与灵活性。需说明的是，方案思路与方法同样适用于其他高校校园网，仅供参考与技术交流。


## 一、方案背景与初衷
高中时期，我长期受顺德区教育网限制，深刻体会到网络受限对学习与生活的影响。进入广东第二师范学院后，发现宿舍校园网存在类似问题：高峰时段视频卡顿、多设备联网频繁断连、部分学习所需外部网站无法打开，直接影响网课学习、资料查询与日常使用。

目前国内高校校园网管控已形成成熟技术链条：前端通过User-Agent特征识别、时间戳动态校验、IPID序列追踪等多维手段实现终端侦测；后端依托IP地址污染、DNS解析劫持、HTTP响应重定向至校内404页面等方式实施访问阻断。这种“侦测-响应”机制闭环，是校园网络自由使用的主要技术壁垒。

基于计算机编译原理的基础认知，以及对OpenWRT固件模块化架构的理解，我通过固件定制编译、协议栈参数优化、应用层代理策略配置等技术路径，系统性解构校园网管控逻辑，最终形成这套兼具实用性与可扩展性的宿舍网络优化方案。


## 二、准备阶段
### 1. 硬件设备
- **主路由设备**：需配备双网口的x86-64架构设备（如工控机、迷你主机），或其他支持刷写OpenWRT系统的主板（如ARM架构开发板），确保可同时连接校园网进线与下级设备。本方案以X86-64架构为例演示。
- **无线接入点**：准备1台支持无线功能的路由器作为AP（接入点），用于扩展WiFi覆盖，推荐支持802.11ac及以上协议的型号，以保证无线速率。

### 2. 编译环境
- **操作系统**：必须使用Ubuntu（推荐20.04/22.04 LTS版本）或其他Linux发行版，本方案以Ubuntu为例操作演示。
- **配置要求**：建议至少4GB内存、20GB空闲硬盘空间，处理器支持多线程以加快编译速度。

### 3. 工具软件
- **PE系统**：用于引导设备并写入固件，推荐使用微PE或大白菜等纯净版PE工具。
- **磁盘写入工具**：[DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download)（下载链接），用于将编译好的OpenWRT镜像写入硬盘。

### 4. 其他准备
- 校园网账号密码：用于验证接入校园网络。
- U盘（8GB以上）：制作PE启动盘及存放固件文件。
- 网线（至少两根）：分别用于连接校园网接口与主路由、主路由与AP设备。

### 5. 面向操作不熟练同学的说明
若无法独立完成以下操作，可直接使用以下命令下载已设置好的配置文件，直接进行编译：
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
```


## 三、开始编译
### 1. 克隆OpenWRT源码
克隆OpenWRT 23.05版本分支（新版可能存在模块不兼容问题）。因网络原因，部分用户需使用代理工具加速：
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. 复制源码副本
为避免操作出错影响原始代码，先复制1份副本，在副本中进行编译：
```bash
cp -r openwrt openwrt_2
```

### 3. 进入副本文件夹
```bash
cd openwrt_2
```

### 4. 更新软件包源
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 5. 克隆核心插件源码
克隆用于规避校园网侦测的核心插件源码：
- UA2F：用于突破User-Agent特征检测
- rkp-ipid：用于修改IPID序列，避免被追踪
```bash
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
```

### 6. 克隆实用工具与插件源码
克隆含网络加速、音乐解锁等功能的实用工具与插件源码：
```bash
# 修复Tailscale编译冲突
sed -i '/\/etc\/init\.d\/tailscale/d;/\/etc\/config\/tailscale/d;' feeds/packages/net/tailscale/Makefile

# 克隆必要插件
git clone https://github.com/asvow/luci-app-tailscale package/luci-app-tailscale  # Tailscale内网穿透
git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon  # Argon主题
git clone https://github.com/jerrykuku/luci-app-argon-config.git package/luci-app-argon-config  # 主题配置工具
git clone https://github.com/lucikap/luci-app-ua2f.git package/luci-app-ua2f  # UA2F图形化配置

# 克隆网易云音乐解锁插件
cd package
git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git
cd ..

# 安装Turbo ACC网络加速模块
curl -sSL https://raw.githubusercontent.com/chenmozhijin/turboacc/luci/add_turboacc.sh -o add_turboacc.sh && bash add_turboacc.sh
```

### 7. 再次更新软件包源
再次更新软件包源，确保新增插件被识别：
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 8. 写入MD5验证值
写入OpenWRT 23.05版本的MD5验证值（用于规避版本检测）：
```bash
nano vermagic  # 创建并编辑vermagic文件
```
在打开的编辑器中输入以下MD5值：
```
47964456485559d992fe6f536131fc64
```
按`Ctrl + O`保存，再按`Ctrl + X`退出编辑器。

### 9. 修改内核编译参数
修改内核编译参数，让系统读取自定义的MD5值：
```bash
nano ./include/kernel-defaults.mk  # 编辑内核默认配置文件
```
按`Ctrl + /`输入`121`并回车，快速定位到第121行，将原有代码注释并替换为读取自定义MD5的配置：
```bash
# 注释原有行
#grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | $(MKHASH) md5 > $(LINUX_DIR)/.vermagic

# 新增行：读取自定义vermagic文件
cp $(TOPDIR)/vermagic $(LINUX_DIR)/.vermagic

```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/1.png)

### 10. 修改内核Makefile
同理修改内核Makefile，确保MD5验证兼容：
```bash
nano ./package/kernel/linux/Makefile  # 编辑内核编译配置文件
```
按`Ctrl + /`输入`29`并回车，定位到第29行，修改为以下内容（注意保留行前空格）：
```bash
# 注意，下面两个要粘贴的注意保留前方空格
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# 新增行：使用自定义vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. 启动编译配置
```bash
make menuconfig
```

### 12. 配置目标系统
进入`menuconfig`后，操作规则如下：双击ESC返回上一页，键盘左右键选择操作，上下键实现滚动。将`Target System`设置为`x86`，`Subtarget`设置为`x86_64`。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. 配置目标镜像分区
进入`Target Images`，仅需修改`Kernel partition size (in MiB)`和`Root filesystem partition size (in MiB)`，具体数值根据自身情况调整：
- `Kernel partition`：存放内核和引导文件的分区
- `Root filesystem partition`：OpenWRT的Root分区

**提示**：若编译固件用于刷入硬路由，此处数值不宜过大，避免撑爆硬路由闪存，可参考以下图片配置。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/4.png)

### 14. 返回主配置界面
操作完成后按ESC返回上一页，直至顶部标题栏显示`OpenWrt Configuration`。若不小心完全退出`make menuconfig`，可重复第11步重新启动配置。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

### 15. 添加模块和软件包
#### （1）添加ipid模块
1. 找到并进入`Kernel modules -> Other modules`
2. 按Y选中`kmod-rkp-ipid`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/6.png)

#### （2）返回主配置界面
按两下ESC退回到`OpenWrt Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

#### （3）添加UA修改模块
1. 找到并进入`Network -> Routing and Redirection`
2. 按Y选中`ua2f`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/7.png)

