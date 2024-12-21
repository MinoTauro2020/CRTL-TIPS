```cmd
# Comandos para averiguar el nombre del dominio en Windows

# 1. Ver el nombre del dominio de la máquina actual
echo %USERDOMAIN%

# 2. Mostrar el nombre del dominio completamente calificado (FQDN)
whoami /fqdn

# 3. Obtener información detallada del sistema y buscar el dominio
systeminfo | findstr /i "Domain"

# Ejemplo de salida:
# Domain:                    essos.local
# Logon Server:              \\DC01

# 4. Usar nltest para obtener detalles del controlador de dominio
# Este comando requiere permisos de administrador.
nltest /dsgetdc:.

# Ejemplo de salida:
# DC: \\DC01
# Address: \\192.168.1.10
# Dom Name: ESSOS.LOCAL
# Forest Name: ESSOS.LOCAL

# 5. Listar todas las variables de entorno relacionadas con el dominio
set

# 6. Usar WMIC para obtener el dominio de la máquina actual
wmic computersystem get domain

# Ejemplo de salida:
# Domain
# essos.local

# Notas adicionales:
# - Algunos comandos, como nltest, pueden requerir permisos de administrador para ejecutarse correctamente.
# - Asegúrate de que tu máquina esté conectada a un dominio, de lo contrario, podrías obtener resultados relacionados con Workgroup.
# - Usa estos comandos para confirmar configuraciones y diagnosticar problemas en el entorno de red.
