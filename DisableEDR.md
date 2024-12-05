```powershell
# Ejecutar PowerShell como administrador
Stop-Service -Name "ElasticEndpoint" -Force & "C:\Program Files\Elastic\Endpoint\elastic-agent.exe" protection disable 
Set-Service -Name "ElasticEndpoint" -StartupType Disabled
Restart-Computer
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ElasticEndpoint" /v Start /t REG_DWORD /d 4 /f
sc delete ElasticEndpoint
rm -Recurse -Force "C:\Program Files\Elastic\Endpoint"

# Defender
# Disable Windows Defender
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /t REG_DWORD /d 1 /f
Uninstall-WindowsFeature -Name Windows-Defender

# Verificar si los procesos Elastic están activos o inactivos y mostrar su estado
$processes = @("elastic-agent", "elastic-endpoint")

foreach ($process in $processes) {
    $status = Get-Process -Name $process -ErrorAction SilentlyContinue
    if ($status) {
        Write-Host "$process está activo." -ForegroundColor Green
        # Mostrar detalles del estado del proceso
        $status | Select-Object Name, Id, StartTime, CPU, WorkingSet
    } else {
        Write-Host "$process está inactivo." -ForegroundColor Red
    }
}

# Ver todos los procesos Elastic
Get-Process | Where-Object { $_.Name -like "*elastic*" }

# Ver detalles adicionales de un proceso específico por nombre
Get-Process -Name "elastic-agent" | Format-List *
Get-Process -Name "elastic-endpoint" | Format-List *

# Ver archivos abiertos por estos procesos (requiere Sysinternals Handle)
# Ejemplo para elastic-agent y elastic-endpoint:
# handle.exe -p elastic-agent
# handle.exe -p elastic-endpoint

# Obtener la ruta del ejecutable de un proceso específico
Get-Process -Name "elastic-agent" | Select-Object -ExpandProperty Path
Get-Process -Name "elastic-endpoint" | Select-Object -ExpandProperty Path

# Ver actividad relacionada con Elastic en los logs de eventos de Windows
Get-WinEvent -FilterHashtable @{LogName="Security"; ProviderName="Microsoft-Windows-Security-Auditing"} | 
    Where-Object { $_.Message -like "*elastic*" }

# Validar si los procesos están activos por su ID (PID)
Get-Process -Id 1948 # elastic-agent
Get-Process -Id 1984 # elastic-endpoint

