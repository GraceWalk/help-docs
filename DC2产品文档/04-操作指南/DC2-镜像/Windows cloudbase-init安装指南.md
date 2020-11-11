
##安装步骤

*  在目标镜像中下载Cloudbase-init。目前Cloudbase-init 稳定版本为 Cloudbase-init 0.9.11，[下载地址](https://cloudbase.it/downloads／CloudbaseInitSetup_0_9_11_x64.msi).

* 双击打开Cloudbase-init工具安装包进行安装。

* 单击“Next”；勾选“I accept the terms in the License Agreement”，单击“Next”。

* 使用Cloudbase-init默认安装路径进行安装，单击“Next”。

* 在“Configuration options”窗口中，设置用户名为“Administrator”，日志输出串口选择“COM1”，且不勾选“Run Cloudbase-init service as LocalSystem”。

单击“Next”，单击“Install”。

> 注意：

> * 在“Completed the Cloudbase-init Setup Wizard ”窗口，请勿勾选“Run Sysprep to create a generalized Image. This is necessary if you plan to duplicate this instance, for example by creating a Glance image”及“Shutdown when Sysprep terminate”。

* 修改Cloudbase-init配置文件，具体如下：

 1. 打开Cloudbase-init.conf配置文件，默认路径为：C:\Program Files\Cloudbase Solutions\Cloudbase-init\conf

 2. 将 Cloudbase-init.conf 配置文件替换为以下内容：

```
[DEFAULT]

username=dc2_user

groups=Administrators

inject_user_password=true

config_drive_raw_hhd=true

config_drive_cdrom=true

config_drive_vfat=true

bsdtar_path=C:\Program Files\Cloudbase Solutions\Cloudbase-init\bin\bsdtar.exe

mtools_path=C:\Program Files\Cloudbase Solutions\Cloudbase-init\bin\

verbose=true

debug=true

logdir=C:\Program Files\Cloudbase Solutions\Cloudbase-init\log\

logfile=Cloudbase-init.log

default_log_levels=comtypes=INFO,suds=INFO,iso8601=WARN,requests=WARN

logging_serial_port_settings=COM1,115200,N,8

mtu_use_dhcp_config=true

ntp_use_dhcp_config=true

local_scripts_path=C:\Program Files\Cloudbase Solutions\Cloudbase-Init\LocalScripts\

metadata_services=cloudbaseinit.metadata.services.httpservice.HttpService

plugins=cloudbaseinit.plugins.common.setuserpassword.SetUserPasswordPlugin
      cloudbaseinit.plugins.common.userdata.UserDataPlugin
         cloudbaseinit.plugins.windows.extendvolumes.ExtendVolumesPlugin,
        cloudbaseinit.plugins.common.sethostname.SetHostNamePlugin

allow_reboot=false

first_logon_behaviour=no

check_latest_version=false
```

