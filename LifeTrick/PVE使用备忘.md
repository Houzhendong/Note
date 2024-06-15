# PVE LXC直通核显

以N5105为例

1. 启用GuC

    ```
    echo “options i915 enable_guc=2” >> /etc/modprobe.d/i915.conf
    ```
    验证状态
    ```bash
    dmesg | grep -iE "guc|huc"
    ```
2. 创建特权容器
3. 编辑容器配置
```bash
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.apparmor.profile: unconfined
```

4. 挂载PVE的硬盘（可选）  
两种方法
    * 在lxc容器的配置中添加

    ```bash
    lxc.mount.entry: /mnt/bindmounts/shared shared none bind,optional,create=dir
    ```
    注意映射的路径不需要以`/`开始

    * 在PVE shell 中执行
    ```bash
    pct set <id> -mp0 /mnt/bindmounts/shared,mp=/shared
    ```