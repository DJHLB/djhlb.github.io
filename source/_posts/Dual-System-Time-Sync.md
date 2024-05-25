---
title: Dual System Time Sync
date: 2024-01-19 19:23:39
tags: System
---

将Windows平台使用 UTC，而非让 Linux 使用地方时。Windows 使用 UTC 后，禁用 Windows 的时间同步功能，以防 Windows 错误设置硬件时间。

使用```regedit```，新建如下 DWORD 值，并将其值设为十六进制的```1```。

```powershell
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\RealTimeIsUniversal
```

也可以用管理员权限启动命令行来完成：

```powershell
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

如果 Windows 根据夏令时更新时钟，可以允许。时钟仍然是 UTC，仅是显示时间会改变。
