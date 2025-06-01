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

# PVE 虚拟机条件启动

在PVE虚拟机搭建中，有可能出现虚拟机依赖的关系（例如虚拟机2的硬盘挂载在虚拟机1的nfs共享上），此时简单的延时启动可能不能保证虚拟机安全启动，需要编写脚本并设为自动启动来达到这一效果

1. 脚本编写
```bash
#!/bin/bash

# 配置日志文件路径
LOG_FILE="/var/log/vm_auto_start.log"
MAX_LOG_SIZE=$((10*1024*1024))  # 10MB

# 定义容器/虚拟机配置
CONTAINER_ID=101
VM_ID=106
NFS_PATH="/mnt/pve/media"

# 初始化日志文件
init_log() {
    touch "$LOG_FILE"
    # 限制日志文件大小（10MB）
    if [ $(stat -c %s "$LOG_FILE") -gt $MAX_LOG_SIZE ]; then
        mv "$LOG_FILE" "${LOG_FILE}.old"
    fi
}

# 带时间戳的日志记录函数
log() {
    local timestamp=$(date +"%Y-%m-%d %T")
    echo "[$timestamp] $1" | tee -a "$LOG_FILE"
}

# 检查挂载状态
check_mount() {
    if mount | grep -q "$NFS_PATH"; then
        log "NFS path $NFS_PATH is mounted"
        return 0
    else
        log "NFS path $NFS_PATH NOT mounted"
        return 1
    fi
}

# 检查并启动容器
start_container() {
    local status=$(pct status $CONTAINER_ID 2>&1 | awk '{print $2}')

    case "$status" in
        "running")
            log "LXC container $CONTAINER_ID is already running"
            return 0
            ;;
        "stopped")
            log "Starting LXC container $CONTAINER_ID..."
            pct start $CONTAINER_ID >> "$LOG_FILE" 2>&1
            return $?
            ;;
        *)
            log "Unknown container status: $status"
            return 1
            ;;
    esac
}

# 检查并启动虚拟机
start_vm() {
    local status=$(qm status $VM_ID 2>&1 | awk '{print $2}')

    case "$status" in
        "running")
            log "VM $VM_ID is already running"
            return 0
            ;;
        "stopped")
            log "Starting VM $VM_ID..."
            qm start $VM_ID >> "$LOG_FILE" 2>&1
            return $?
            ;;
        *)
            log "Unknown VM status: $status"
            return 1
            ;;
    esac
}

# 主循环
main_loop() {
    init_log
    log "====== Starting monitoring script ======"

    while true; do
        if check_mount; then
            container_status="false"
            vm_status="false"

            if start_container; then
                container_status="true"
            fi

            if start_vm; then
                vm_status="true"
            fi

            if [ "$container_status" = "true" ] && [ "$vm_status" = "true" ]; then
            # if [ "$container_status" = "true" ] ; then
                log "Both services are running. Monitoring complete."
                break
            else
                log "Retrying in 20 seconds..."
                sleep 20
            fi
        else
            log "NFS not mounted, retrying in 20 seconds..."
            sleep 20
        fi
    done

    log "Exit."
}

# 执行主程序
main_loop
```
2. 设置自动启动
    1. 创建服务文件
    ```ini
    [Unit]
    Description=My Script Service
    After=network.target

    [Service]
    ExecStart=/path/to/your/script.sh
    Restart=always

    [Install]
    WantedBy=multi-user.target

    ```
    2. 保存文件到 `/etc/systemd/system/your-script.service`
    3. 启动服务
    ```bash
     # 生效服务
    sudo systemctl enable your-script.service
     # 启动服务
    sudo systemctl start your-script.service
     # 查看服务状态
    sudo systemctl status your-script.service
    ```
