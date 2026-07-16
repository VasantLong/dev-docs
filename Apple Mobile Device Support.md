# Windows connect Apple Mobile Devices 

## 直接从微软官方源下载纯净驱动（推荐）
跳过iTunes安装，只取必要驱动文件，全程1–2分钟，不装任何冗余程序。

### 1 管理员PowerShell

```pwsh
iex (Invoke-RestMethod -Uri 'https://raw.githubusercontent.com/NelloKudo/Apple-Mobile-Drivers-Installer/main/AppleDrivInstaller.ps1')
```

脚本自动运行后，会从Microsoft Update Catalog下载最新版`AppleMobileDeviceSupport64.msi`及USB以太网驱动，并静默安装。安装完成前请勿断开iPhone连接，也<b>不要关闭PowerShell窗口</b>，否则驱动可能未注册完整。

### 2 重启Apple Mobile Device Service
按Win+R输入services.msc → 找到“Apple Mobile Device Service” → 右键“重新启动”。此时再插上iPhone，应能识别为“Apple iPhone”，且USB网络共享图标出现在任务栏。