## 监控脚本（python）

```python
import psutil
import platform
import subprocess
import time
import os

# 监控系统信息
def system_info():
    print("--- 系统信息 ---")
    print("操作系统: " + platform.system())
    print("版本号: " + platform.release())
    print("处理器架构: " + platform.machine())
    print("主机名: " + platform.node())
    print("启动时间: " + time.ctime(psutil.boot_time()))

# 监控服务状态
def service_status():
    print("--- 服务状态 ---")
    services = ["service_name1", "service_name2"]  # 在此添加需要监控的服务名
    for service in services:
        status = subprocess.run(["systemctl", "is-active", service], capture_output=True)
        if status.stdout.decode().strip() == "active":
            print(service + " 状态: 运行中")
        else:
            print(service + " 状态: 停止")

# 监控硬件信息
def hardware_info():
    print("--- 硬件信息 ---")
    cpu_info = subprocess.run(["lscpu"], capture_output=True, text=True)
    print(cpu_info.stdout)
    disk_info = subprocess.run(["lsblk"], capture_output=True, text=True)
    print(disk_info.stdout)
    # 在此添加其他需要监控的硬件信息命令

# 监控系统性能
def system_performance():
    print("--- 系统性能 ---")
    cpu_usage = psutil.cpu_percent(interval=1)
    print("CPU 使用率: " + str(cpu_usage) + "%")
    mem_usage = psutil.virtual_memory().percent
    print("内存使用率: " + str(mem_usage) + "%")
    disk_usage = psutil.disk_usage("/").percent
    print("根目录磁盘使用率: " + str(disk_usage) + "%")
    # 在此添加其他需要监控的性能信息

# 安全监控
def security_monitoring():
    print("--- 安全监控 ---")
    log_file = "/var/log/secure"  # 在此设置需要监控的安全日志文件路径
    if os.path.exists(log_file):
        with open(log_file, "r") as f:
            lines = f.readlines()
            last_lines = lines[-100:]  # 获取最后100行日志
            for line in last_lines:
                print(line.strip())

# 执行监控任务
def monitor():
    system_info()
    service_status()
    hardware_info()
    system_performance()
    security_monitoring()

# 按照一定时间间隔执行监控任务
while True:
    monitor()
    time.sleep(300)  # 5分钟
```





