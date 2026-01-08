+++
date = '2026-01-07T15:15:47+08:00'
draft = false
title = 'wsl安装发行版遇到0x80070005错误'
+++
## 错误代码0x80070005
当系统或用户在 Windows 更新时缺少更改设置所需的必要文件或权限时，会发生错误 0x80070005。

以下是一些可能解决此问题的故障排除方法。

解决方案1：
检查 Windows 系统中是否存在损坏的文件。
1. 右键单击“开始”菜单，然后单击“Windows PowerShell（管理员）”。
2. 输入以下命令：
    ```text
    sfc /scannow（并按 Enter 键）
    Dism /Online /Cleanup-Image /ScanHealth（并按 Enter 键）
    Dism /Online /Cleanup-Image /CheckHealth（并按 Enter 键）
    ```

3. DISM 工具会报告映像是否健康、可修复或不可修复。如果映像可修复，您可以使用 `/RestoreHealth（Dism /Online /Cleanup-Image /RestoreHealth）`参数来修复映像。

解决方案 2：
可能是更新文件本身出了问题。清除存储所有更新文件的文件夹将强制 Windows 更新下载新文件。
以下是如何清除更新数据库缓存，并将 Windows 更新组件重置为默认设置的方法。

1. 按 Windows + R 键，输入 services.msc，然后单击“确定”打开 Windows 服务。
2. 向下滚动并找到 Windows 更新服务。
3. 右键单击该服务，然后选择“停止”。
4. 对**BITS（后台智能传输服务）**和**Superfetch（Superfetch 现已更名为 Sysmain）**执行相同的操作，右键单击并选择“停止”。
5. 现在转到以下位置：`C:\Windows\SoftwareDistribution\Download`。
6. 删除 download 文件夹中的所有内容，但不要删除文件夹本身。
    要删除文件夹，请按 Ctrl + A 选择所有文件，然后按 Delete 键删除文件。
    再次打开 Windows 服务，然后重新启动之前停止的服务（Windows 更新、BITS）。
7. 重启计算机并重试。

解决方案 3：
重新安装 Microsoft Store。
1. 右键单击“开始”。
2. 单击“Windows PowerShell（管理员）”。
3. 输入：
`Get-AppXPackage -allusers | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"`，然后按 Enter 键。
4. 重启电脑，然后尝试打开 Microsoft Store。

解决方案 4：
如果以上步骤均无法解决问题，您可以使用媒体创建工具执行修复升级。此过程将修复所有 Windows 文件。
1. 下载媒体创建工具：
`https://go.microsoft.com/fwlink/?LinkId=691209`
2. 运行该工具并选择“立即升级此电脑”
3. 选择“保留我的所有文件和应用”选项
4. 然后单击更新。

参考：[https://learn.microsoft.com/en-us/answers/questions/1022056/error-0x80070005]