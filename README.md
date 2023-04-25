# AzurePipelines

# Nexus URL
$URL = "$($nexus_repository_url)/repository/$($nexus_repository_name)/$($app_short_name)/$($nexus_artifact_version)/$($app_short_name).zip"

# Set Directory VARS
$HFXDIR = "C:\healthefx\debug-mode-monitor"

# Setup .NET
$env:DEBIAN_FRONTED = "noninteractive"
Invoke-WebRequest -Uri "${dotnet installer download uri}" -OutFile packages-microsoft-prod.deb
Start-Process dpkg -ArgumentList "-i", "packages-microsoft-prod.deb" -Verb RunAs

# Create Directories
New-Item -ItemType Directory -Path "C:\healthefx\debug-mode-monitor\logs" -Force | Out-Null
New-Item -ItemType Directory -Path $HFXDIR -Force | Out-Null

# Create User and Group
Write-Output "In## Create User: `n`t- debug-mode-monitor `n`t- lock the user account"
New-LocalUser -Name debug-mode-monitor
Set-LocalUser -Name debug-mode-monitor -Enabled $false
Write-Output "User Created!"

Write-Output "In## List users of debug-mode-monitor group a group creation"
Get-LocalGroupMember -Group debug-mode-monitor

# Download Zip
Write-Output "In## Downloading $($app_short_name).zip from Nexus Repository:`n$URL"
Invoke-WebRequest $URL -OutFile "$env:USERPROFILE\$($app_short_name).zip"
Write-Output "Download Completed!"

# Unzip Nexus Download
Write-Output "`nUnzip Nexus Artifact!"
Expand-Archive -LiteralPath "$env:USERPROFILE\$($app_short_name).zip" -DestinationPath $HFXDIR
Get-ChildItem -Path $HFXDIR | Sort-Object -Property LastWriteTime -Descending
Write-Output "Artifacts Unziped to $HFXDIR"

# Clean-up downloads
Write-Output "`nClean up Downloaded file"
Get-ChildItem -Path $env:USERPROFILE
Remove-Item -Path "$env:USERPROFILE\$($app_short_name).zip"
Write-Output "Downloaded file was deleted!"
Get-ChildItem -Path $env:USERPROFILE

# Copy File to systemd dir
Write-Output "`nCopy service definition to system directory"
Copy-Item -Path "$HFXDIR\debug-mode-monitor.service" -Destination "C:\systemd\system\debug-mode-monitor.service"
Write-Output "Service definition copied!"
Get-ChildItem -Path "C:\systemd\system\debug-mode-monitor.service"

# Set Permissions
Write-Output @"
## changing file and directory ownership for:
- chown debug-mode-monitor:debug-mode-monitor C:\systemd\system\debug-mode-monitor.service
- chown debug-mode-monitor:debug-mode-monitor C:\healthefx\debug-mode-monitor\logs
- chown debug-mode-monitor:root C:\healthefx\debug-mode-monitor
- chown -R debug-mode-monitor:debug-mode-monitor C:\healthefx\debug-mode-monitor
"@
Set-Owner -Path "C:\systemd\system\debug-mode-monitor.service" -Account "debug-mode-monitor"
Set-Owner -Path "C:\healthefx\debug-mode-monitor\logs" -Account "debug-mode-monitor"
Set-Owner -Path "C:\healthefx\debug-mode-monitor" -Account "debug-mode-monitor", "root"
Set-Owner -Path "C:\healthefx\debug-mode-monitor" -Account "debug-mode-monitor" -Recurse

Write-Output "`n## ownership updated!"
Write-Output "## List contents of Direct"