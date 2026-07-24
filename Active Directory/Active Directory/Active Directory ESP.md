# 🔐 ACTIVE DIRECTORY: Guía Completa para Blue Team
## Identidades, Políticas, Auditoría y Ataques

---

## 📖 ¿Qué es Active Directory?

**En 30 segundos:**
Active Directory (AD) es el **servicio centralizado de Microsoft que gestiona identidades (usuarios, grupos, computadoras), políticas de seguridad (GPO), permisos y auditoría en una red corporativa Windows**. No es solo almacenamiento de contraseñas — es el **cerebro de la infraestructura que controla quién puede hacer qué, dónde y cuándo**.

**Analogía:**
```
CSV de contraseñas = Un archivo estático con datos
Active Directory = Una ciudad con gobierno, leyes, policía y registros

En el CSV: "jsmith: contraseña123"
En AD: "jsmith puede acceder a estos servidores, no puede instalar software,
        debe cambiar contraseña cada 30 días, está en estos grupos,
        su cuenta fue bloqueada si falla 5 logins, etc."
```

---

## 🎯 ¿Por Qué Un SOC/Blue Team Necesita Entender AD?

### En La Realidad del Trabajo

| Escenario | Sin entender AD | Con entender AD |
|-----------|-----------------|-----------------|
| Atacante crea cuenta admin oculta | "¿Cómo entró?" | Busco EventID 4720 en AD audit logs |
| Usuario no puede acceder a carpeta compartida | "¿Por qué?" | Revisar permisos NTFS + membresía de grupo |
| Cambio de contraseña admin a las 3 AM | "¿Normal?" | No, revisar EventID 4723 + IP origen |
| Malware intenta movimiento lateral | "¿Puede?" | Depende de permisos delegados en AD |
| Compromiso de cuenta de servicio | "¿Qué acceso tiene?" | Revisar grupos + ACLs en AD |

### Preguntas de Entrevista

1. **"¿Qué es Active Directory y por qué es importante?"**
   - Esperado: "Sistema centralizado que gestiona identidades, políticas y auditoría. Si se compromete, el atacante controla toda la red."

2. **"¿Cuál es la diferencia entre una Cuenta y un Objeto en AD?"**
   - Esperado: "Una cuenta es un usuario. Un objeto es cualquier entidad (usuario, grupo, computadora, impresora). Los atacantes buscan comprometer cuentas pero crear objetos para persistencia."

3. **"¿Cómo detectarías una cuenta de administrador oculta creada por un atacante?"**
   - Esperado: "Buscaría EventID 4720 (Cuenta creada), especialmente en horarios anormales, y correlacionaría con EventID 4728 (Miembro agregado a grupo Administrators)."

4. **"¿Qué es una Golden Ticket?"**
   - Esperado: "Es un ticket Kerberos falsificado. Un atacante puede hacer "cualquier cosa" sin necesidad de contraseña. Es muy peligrosa porque es difícil de detectar."

---

## 🔍 El Concepto Desglosado

### Parte 1: La Estructura Jerárquica de AD

```
DOMINIO (ejemplo: corp.local)
│
├── FOREST (Árbol de dominios)
│   └── Contiene 1+ dominios relacionados
│
├── Domain Controllers (DC)
│   ├── DC01
│   ├── DC02
│   └── Replican la información entre ellos
│
└── ORGANIZATIONAL UNITS (OUs) - Contenedores lógicos
    ├── OU=Usuarios
    │   ├── OU=Administrativos
    │   │   └── Admin accounts
    │   ├── OU=Developers
    │   │   └── Developer accounts
    │   └── OU=Empleados
    │       └── Regular user accounts
    │
    ├── OU=Computadoras
    │   ├── OU=Servidores
    │   ├── OU=Workstations
    │   └── OU=Laptops
    │
    └── OU=Grupos
        ├── Grupo: Administrators
        ├── Grupo: Domain Admins
        ├── Grupo: Finance-Users
        └── Grupo: IT-Support
```

**Por qué importa:** Los atacantes buscan cambiar políticas a nivel de FOREST o modificar grupos de administradores en una OU específica.

### Parte 2: Objetos vs Cuentas vs Grupos

**OBJETO** = Entidad en el directorio

```json
{
  "objectClass": "user",
  "sAMAccountName": "jsmith",
  "displayName": "John Smith",
  "userPrincipalName": "jsmith@corp.local",
  "mail": "jsmith@corp.local",
  "department": "Sales",
  "manager": "mgonzalez",
  "memberOf": ["group-sales", "group-vpn-access"],
  "lastLogonTimestamp": "2024-01-15T14:32:00Z",
  "pwdLastSet": "2024-01-01T08:00:00Z",
  "accountExpires": "2025-01-01",
  "userAccountControl": "512"
}
```

**CUENTA** = Concepto específico de login

```
Una CUENTA puede ser:
├─ Usuario (user) - Login interactivo
├─ Service Account - Para ejecutar servicios/trabajos programados
├─ Computer Account - Para que se autentique la máquina en la red
└─ Managed Service Account (MSA) - Tipo especial de servicio
```

**GRUPO** = Colección de objetos (típicamente usuarios y computadoras)

```
Grupo-Ejemplo: "Domain Admins"
├─ Miembros: user1, user2, user3, ...
├─ Permisos: Full Control en todos los servidores
├─ Scope:
│   ├─ Global (dentro del dominio)
│   ├─ Domain Local (un servidor específico)
│   └─ Universal (múltiples dominios en la organización)
└─ Type:
    ├─ Security group (para permisos)
    └─ Distribution group (para emails)
```

**Por qué importa:** Los atacantes comprometen cuentas, pero se vuelven persistentes mediante grupos (especialmente "Domain Admins" o grupos ocultos).

### Parte 3: El Protocolo de Autenticación (Kerberos)

Active Directory usa **Kerberos** para autenticación. Aquí está el flujo:

```
┌───────────┐
│   USER    │ "Quiero acceder al servidor FILE-01"
└─────┬─────┘
      │ 1. Envía credenciales
      ▼
┌──────────────────────────────────────┐
│  Authentication Server (AS)          │
│  (Part del Domain Controller)         │
│  - Valida contraseña                 │
│  - Emite TGT (Ticket Granting Ticket)│
└─────┬────────────────────────────────┘
      │ 2. Recibe TGT (válido 10 horas por defecto)
      ▼
┌──────────────────────────────────────┐
│  Ticket Granting Server (TGS)        │
│  (Part del Domain Controller)         │
│  - Recibe TGT                         │
│  - Emite Service Ticket              │
└─────┬────────────────────────────────┘
      │ 3. Recibe Service Ticket
      ▼
┌──────────────────────────────────────┐
│       FILE-01 (Servidor)             │
│  - Valida Service Ticket             │
│  - Otorga acceso                     │
└──────────────────────────────────────┘
```

**En un ataque (Golden Ticket):**

```
Atacante + compromiso AD = Puede falsificar un TGT
→ Acceso a CUALQUIER servidor sin necesidad de contraseña
→ Válido por 10 horas (por defecto)
→ Muy difícil de detectar
```

---

## ⚙️ EventIDs de Active Directory — El Corazón de la Auditoría

### Los EventIDs Más Críticos

Cuando AD emite un evento, se registra en el **Security Event Log** de los Domain Controllers.

| EventID | Evento | Ejemplo Log | Criticidad | Detección |
|---------|--------|-------------|-----------|-----------|
| **4624** | Login exitoso | User jsmith logged on | 🟡 Medium | Solo si IP rara u hora rara |
| **4625** | Login fallido | Failed password for jsmith | 🟡 Medium | 5+ en 5 min = brute force |
| **4720** | Cuenta creada | New user account created | 🔴 CRITICAL | Cualquier creación de cuenta a las 2-5 AM |
| **4722** | Cuenta habilitada | User account enabled | 🔴 CRITICAL | Reactivación de cuenta deshabilitada |
| **4723** | Cambio de contraseña | Password changed for admin | 🔴 CRITICAL | Especialmente si es admin a hora rara |
| **4728** | Miembro agregado a grupo | User added to Administrators | 🔴 CRITICAL | Siempre, especialmente si es grupo admin |
| **4729** | Miembro removido de grupo | User removed from group | 🟠 HIGH | Si es grupo crítico |
| **4730** | Grupo eliminado | Security group deleted | 🔴 CRITICAL | Posible eliminación de grupo audit |
| **4739** | Cambio de política | Domain policy changed | 🔴 CRITICAL | Cambio en GPO (antivirus, auditoría, firewall) |
| **4752** | Miembro agregado a grupo global | Added to global security group | 🟠 HIGH | Depende del grupo |
| **4768** | Ticket Kerberos concedido | Kerberos TGT issued | 🟡 Medium | Base para detectar Golden Ticket (muy ruidoso) |
| **4769** | Ticket Kerberos de servicio | Service Ticket requested | 🟡 Medium | Kerberoasting detection |
| **4771** | Autenticación Kerberos fallida | Pre-authentication failed | 🟡 Medium | Possible Kerberoasting attempt |
| **5136** | Objeto en AD modificado | Directory Service Object Modified | 🔴 CRITICAL | Cambios en atributos sensibles |

### Logs Crudos Ejemplo

**EventID 4720 (Cuenta creada) - CRÍTICO:**

```text
<134>Jan 15 03:14:22 DC01 MSWinEventLog 1 Security 94512
Mon Jan 15 03:14:22 2024 4720 Security SYSTEM User Success Audit
DC01 User Account Management

A user account was created.

Target Account Name: backdoor_admin
Target Account ID: CORP\S-1-5-21-3623811015-3361044348-30300820-2157
Creator Subject: Security ID: CORP\Administrator Account Name: Administrator
Domain Name: CORP Logon ID: 0x3E7

Attributes:
SAM Account Name: backdoor_admin
Display Name: (none)
User Principal Name: -
Home Directory: -
Script Path: -
Profile Path: -
User Workstations: -
Password Last Set: <never>
Account Expires: <never>
Primary Group ID: 513
AllowReversibleEncryption - Disabled
```

**¿Por qué es crítico?**
- Hora: 3:14 AM (anormal)
- Account Name: "backdoor_admin" (sospechoso)
- Creator: Administrator (podría ser atacante con privilegios)
- Sin contraseña inicial (puede indicar que va a ser cambiada después)

---

**EventID 4728 (Miembro agregado a grupo Administrators) - CRÍTICO:**

```text
<134>Jan 15 03:16:45 DC01 MSWinEventLog 1 Security 94513
Mon Jan 15 03:16:45 2024 4728 Security SYSTEM User Success Audit
DC01 User Account Management

A member was added to a security-enabled global group.

Group: Group Name: Administrators
Group Domain: CORP
Actors: Subject Security ID: CORP\S-1-5-21-3623811015-3361044348-30300820-500
Member Added: Member Name: backdoor_admin
Member SID: CORP\S-1-5-21-3623811015-3361044348-30300820-2157
```

**¿Por qué es crítico?**
- Grupo afectado: Administrators (máximo privilegio)
- Usuario agregado: backdoor_admin (la cuenta sospechosa)
- Correlación con EventID 4720: Creada hace 2 minutos

---

**EventID 4623 (Cambio de contraseña) - CRÍTICO si es admin:**

```text
<134>Jan 15 03:18:00 DC01 MSWinEventLog 1 Security 94514
Mon Jan 15 03:18:00 2024 4723 Security SYSTEM User Success Audit
DC01 User Account Management

An attempt was made to change an account's password.

Subject: Account Name: Administrator
Domain Name: CORP
Logon ID: 0x3E7

Target Account: Account Name: domain_admin_main
Account Domain: CORP

Change Type: Computer Access Control (Caller changed password of target account)
```

**¿Por qué es crítico?**
- Admin main (cuenta principal de administrador)
- Hora: 3:18 AM (atacante trabajando)
- Patrón visible: 4720 (crear) → 4728 (agregar a grupo) → 4723 (cambiar contraseña)

---

### Correlación de EventIDs — Detectar un Ataque Completo

**Escenario: Atacante crea persistencia en AD**

```
Timeline:
2024-01-15 03:14:22 → EventID 4720: Cuenta "backdoor_admin" creada
2024-01-15 03:16:45 → EventID 4728: "backdoor_admin" agregado a "Administrators"
2024-01-15 03:18:00 → EventID 4723: Contraseña cambiada para "backdoor_admin"
2024-01-15 03:20:15 → EventID 4624: "backdoor_admin" login exitoso desde IP 198.51.100.45

SIEM correlación:
└─> 4720 + 4728 + 4723 + 4624 en 6 minutos = CRITICAL ATTACK
    Acción: Bloquear IP, deshabilitar cuenta, investigar acceso
```

---

## 🚨 Técnicas de Ataque Contra AD (MITRE ATT&CK)

### Ataque #1: Brute Force (T1110 - Credential Access)

**¿Qué es?**
Intentar múltiples contraseñas contra una cuenta.

**Log:**
```text
<134>Jan 15 09:14:01 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:03 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:05 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
[5+ veces en 5 minutos, misma IP]
```

**Detección (Regla SIEM):**
```yaml
title: Brute Force Against AD Account
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4625
    Status: 'C0000064'  # Contraseña incorrecta
  timeframe: 5m
  condition: selection | count(TargetUserName) by SourceIp >= 5
level: high
tags:
  - attack.credential_access
  - attack.t1110_001
```

---

### Ataque #2: Golden Ticket (T1558_001 - Privilege Escalation)

**¿Qué es?**
Un atacante que ha dumped el `krbtgt` hash puede falsificar un ticket Kerberos (TGT) válido y acceder a cualquier servidor sin contraseña.

**Por qué es peligrosa:**
- No necesita cambiar contraseñas
- Válida por 10 horas por defecto (sin volver a autenticarse)
- **MUY difícil de detectar** en logs normales

**Indicadores (difíciles, pero existen):**
```text
EventID 4768/4769 con timestamps anormales
EventID 4768 sin correspondiente EventID 4624
Múltiples Service Tickets en segundos (patrón anormal)
```

---

### Ataque #3: Kerberoasting (T1558_003 - Credential Access)

**¿Qué es?**
Un usuario puede solicitar un Service Ticket para cualquier servicio sin privilegios especiales. El ticket contiene un hash que puede ser crackeado offline.

**Log:**
```text
<134>Jan 15 14:32:01 DC01 ... 4769 ...
  Service Name: MSSQLSvc/database.corp.local
  Client: jsmith@CORP
  Ticket Encryption Type: 0x17 (RC4 - fácil de crackear)
  Failure Code: 0x0 (exitoso)
```

**Detección:**
Buscar EventID 4769 con múltiples Service Tickets en poco tiempo, especialmente con encryption type RC4.

---

### Ataque #4: Credential Dumping (T1003 - Credential Access)

**¿Qué es?**
Atacante ejecuta herramientas (mimikatz, ntdsutil) para dumpear contraseñas y hashes desde AD.

**Indicadores:**
```text
EventID 4656: Access to NTDS.dit (base de datos AD)
EventID 5136: Cambios en objetos AD (puede indicar post-dump)
EventID 4662: Acceso a objetos sensibles
```

---

### Ataque #5: ACL Abuse (T1098_003 - Persistence)

**¿Qué es?**
Modificar permisos (ACLs) en objetos AD para que el atacante pueda:
- Cambiar contraseñas de otros usuarios
- Agregar miembros a grupos
- Crear cuentas
- Modificar GPOs

**Log:**
```text
<134>Jan 15 15:45:22 DC01 ... 5136 ...
  Object: CN=Admins,CN=Groups,DC=corp,DC=local
  Attribute: nTSecurityDescriptor
  Operation: Modify
  Modification: Added GenericAll permission to S-1-5-21-...-2157
```

**Peligro:** El atacante puede hacer cambios sin necesidad de estar en el grupo Administrators.

---

## 🛡️ Defensa y Detección en AD

### Tier 1: Protección Básica

```
✅ Políticas de contraseña fuertes:
   ├─ Mínimo 14 caracteres
   ├─ Complejidad (mayúsculas, minúsculas, números, símbolos)
   ├─ Cambio cada 90 días
   └─ Historial: no reutilizar últimas 24 contraseñas

✅ Lockout de cuentas:
   ├─ Tras 5 intentos fallidos
   └─ Bloqueo por 30 minutos

✅ Auditoría habilitada:
   ├─ Success + Failure en todos los eventos sensibles
   └─ Retención: mínimo 90 días
```

### Tier 2: Hardening de AD

```
✅ Cuentas de administrador:
   ├─ Crear cuenta "Tier 0" dedicada
   ├─ No usar para tareas cotidianas
   ├─ Usar PAM (Privileged Access Management)
   └─ MFA obligatorio

✅ Grupos sensibles (monitoreo 24/7):
   ├─ Domain Admins
   ├─ Enterprise Admins
   ├─ Schema Admins
   ├─ Backup Operators
   └─ Account Operators

✅ Protección de Domain Controllers:
   ├─ Firewall restrictivo
   ├─ Solo RPC en puerto 135
   ├─ No SMB desde workstations
   └─ Sin acceso directo a internet
```

### Tier 3: Detección en SIEM

```
Regla: Creación de cuenta + Adición a grupo admin en <5 minutos

title: Likely Persistence - Account Creation + Admin Group Addition
logsource:
  product: windows
  service: security
detection:
  account_created:
    EventID: 4720
  added_to_admin:
    EventID: 4728
    Group: 'Administrators'
  timeframe: 5m
  condition: account_created AND added_to_admin
level: critical
tags:
  - attack.persistence
  - attack.t1098
```

---

## ❌ Errores Comunes que Cometen Analistas

### Error #1: Ignorar "cambios de contraseña normales"

```
❌ MAL:
"User cambió su contraseña, es normal"

✅ BIEN:
"¿Fue el mismo usuario quien cambió su contraseña?
¿O fue un admin? ¿A qué hora? ¿Desde qué IP?
Si admin cambió password de otro admin a las 3 AM = INVESTIGAR"
```

---

### Error #2: Confundir "Cuenta" con "Objeto"

```
❌ MAL:
"Monitoreo todas las cuentas en AD"

✅ BIEN:
"Monitoreo cuentas CRÍTICAS (admins) Y todos los OBJETOS (grupos, GPOs)
porque los atacantes se vuelven persistentes mediante grupos"
```

---

### Error #3: No correlacionar AD con otros logs

```
❌ MAL:
"Veo EventID 4720 (cuenta creada), cierro como admin maltrecho"

✅ BIEN:
"Veo EventID 4720 + busco dentro de 10 minutos:
├─ EventID 4728 (agregado a grupo admin)?
├─ EventID 4624 (login exitoso desde IP rara)?
├─ EventID 4723 (contraseña cambiada)?
└─ Si encuentro estos, es ATTACK PATTERN = ESCALADO"
```

---

### Error #4: Confiar solo en "nombre de cuenta"

```
❌ MAL:
Regla: "Alerta si se crea cuenta llamada 'admin%'"

✅ BIEN:
"Alertar si se crea CUALQUIER cuenta a las 2-5 AM,
independientemente del nombre. Un atacante puede llamarla
'john_backup' para parecer legítima"
```

---

## 🧪 Ejercicios Prácticos

### Ejercicio #1: Analizar una Correlación de Eventos

**Logs presentados en orden:**

```
[2024-01-15 03:14:22] DC01 EventID 4720
  New Account: temporary_admin
  Creator: Administrator

[2024-01-15 03:16:45] DC01 EventID 4728
  Member Added: temporary_admin
  Group: Domain Admins

[2024-01-15 03:20:00] DC01 EventID 4624
  Successful Login: temporary_admin
  Source IP: 203.0.113.45
  Logon Type: 3 (Network)

[2024-01-15 03:21:30] FW-EDGE-01 Firewall Log
  Outbound Connection: 203.0.113.45 -> 185.220.100.50:443
  Protocol: HTTPS
```

**Preguntas:**
1. ¿Cuál es el patrón de ataque?
2. ¿Qué EventID es el más crítico?
3. ¿Cómo escribirías una regla SIEM para detectar esto?

**Respuesta Esperada:**
1. **Patrón:** Persistencia. Atacante crea cuenta admin, la habilita, hace login, exfiltra datos.
2. **EventID más crítico:** 4728 (agregar a Domain Admins = máximo acceso).
3. **Regla:**
```yaml
title: Likely Attacker Persistence
detection:
  account_created:
    EventID: 4720
  added_to_domain_admins:
    EventID: 4728
    Group: 'Domain Admins'
  successful_login:
    EventID: 4624
    TargetUserName: # la cuenta creada
  timeframe: 10m
  condition: account_created AND added_to_domain_admins AND successful_login
level: critical
```

---

### Ejercicio #2: ¿Golden Ticket o Login Normal?

**Escenario:** Tu SIEM detecta:
```
User: admin_main
Login Time: 2024-01-15 14:32:00
From IP: 192.168.1.100 (internal workstation)
EventID: 4624 (successful login)
Logon Type: 3 (Network)
```

**Pregunta:** ¿Es sospechoso?

**Análisis:**
```
✓ Hora normal (14:32 en horario laboral)
✓ IP interna (workstation, no internet)
✓ EventID 4624 normal

PERO ¿hay Golden Ticket indicador?
├─ ¿Hay EventID 4768 sin 4624 previo? (TGT sin login normal)
├─ ¿Hay múltiples Service Tickets en segundos? (4769 patrón anormal)
└─ ¿El login es "normal" pero después accede a recursos que nunca accedía?

Conclusión: NO es Golden Ticket solo con esto.
Necesitarías: EventID 4768 anormal OR EventID 5136 cambios en ACLs.
```

---

### Ejercicio #3: Diseña una Política de AD Segura

**Escenario:** Eres Security Architect en una empresa con 5,000 usuarios.

**Tarea:** Define:
1. Estructura de OUs
2. Grupos de administración
3. Políticas de contraseña
4. Auditoría

**Respuesta Esperada:**

```
OUs:
├─ OU=Tier0 (Domain Admins only)
│  └─ Acceso: MFA, PAM, máximo 2 admins
│
├─ OU=Tier1 (Server Admins)
│  ├─ OU=Database-Admins
│  ├─ OU=Infrastructure-Admins
│  └─ Acceso: MFA, cambio contraseña cada 60 días
│
├─ OU=Tier2 (Regular Admins)
│  ├─ OU=Team-Leads
│  ├─ OU=IT-Support
│  └─ Acceso: MFA, cambio cada 90 días
│
└─ OU=Users (empleados regulares)
   ├─ OU=Finance
   ├─ OU=Sales
   └─ OU=Engineering

Auditoría:
├─ EventID 4720, 4722, 4723, 4728, 4729, 4730: 100% logging
├─ EventID 4624 (login exitoso): Solo IP rara O admin
├─ EventID 5136 (cambios): 100% logging de ACL changes
└─ Retención: 180 días (compliance)
```

---

## 🎯 Preguntas de Entrevista

### Level 1 (Entry-Level SOC Analyst)

**P1: "¿Qué es Active Directory?"**
> "Un servicio de Microsoft que gestiona identidades (usuarios, computadoras, grupos), políticas (GPO) y auditoría en una red Windows corporativa."

**P2: "¿Cuál es la diferencia entre una Cuenta y un Objeto en AD?"**
> "Una Cuenta es un usuario con credenciales de login. Un Objeto es cualquier entidad en AD: usuario, grupo, computadora, impresora, etc."

**P3: "¿Qué EventID indica que un atacante creó una cuenta de administrador?"**
> "EventID 4720 (Cuenta creada). Especialmente si ocurre a las 2-5 AM y luego hay EventID 4728 (agregado a Administrators)."

### Level 2 (Mid-Level SOC Analyst)

**P1: "¿Cómo detectarías una Golden Ticket?"**
> "Buscaría EventID 4768 (TGT emitido) sin EventID 4624 previo, o múltiples EventID 4769 (Service Tickets) en segundos. También podría indicarse como cambios en objetos sensibles sin logs de login normal (EventID 5136)."

**P2: "¿Por qué es importante monitorear EventID 4728 (Miembro agregado a grupo)?"**
> "Porque indica que alguien agregó un usuario a un grupo. Si es grupo admin, un atacante acaba de elevar sus privilegios. Si es grupo crítico, podría afectar a múltiples servidores."

**P3: "¿Cuál es la diferencia entre Kerberoasting y brute force contra AD?"**
> "Brute force: intentar múltiples contraseñas contra una cuenta online (EventID 4625). Kerberoasting: solicitar Service Tickets (EventID 4769), que contienen hashes que se pueden crackear offline sin ser detectado directamente."

### Level 3 (Senior Analyst / Detection Engineer)

**P1: "¿Cómo diseñarías un programa de detección para AD en un SOC con 10,000 usuarios?"**
> "1) Definir 'Tier 0' (Domain Admins, Enterprise Admins) con monitoreo 24/7. 2) Crear reglas SIEM para patrones de persistencia (4720+4728+4723). 3) Establecer baselines de login normal por usuario. 4) Correlacionar EventID 4625 (fallidos) con 4624 (exitosos) = posible ataque. 5) Monitorear cambios de GPO (EventID 5136). 6) Usar threat intelligence para detectar credenciales dumped."

**P2: "¿Cuál es el ataque más peligroso contra AD y cómo lo detectarías?"**
> "Golden Ticket, porque permite acceso sin cambiar contraseñas. Detección difícil: necesitarías EventID 4768 sin 4624 previo, múltiples 4769 anormales, o correlacionar con cambios en ACLs/GPO (EventID 5136). Alternativa: usar EDR (Endpoint Detection & Response) para detectar mimikatz/ntdsutil en ejecución."

**P3: "¿Cómo mitigarías un comprometimiento de Domain Admin?"**
> "1) Cambiar contraseña inmediatamente (pero atacante podría tener Golden Ticket). 2) Resetear krbtgt password (invalida todos los Golden Tickets existentes). 3) Forzar logout de todas las sesiones. 4) Auditar EventID 5136 para detectar cambios en ACLs. 5) Investigar qué accedió durante el tiempo de compromiso. 6) Considerar domain controller reset si es muy grave."

---

## 🔗 Conexiones a Otros Temas

- 🔥 **SIEM & Detection**: Reglas SIEM para detectar ataques contra AD (EventID 4720, 4728, etc.)
- 🛡️ **Incident Response**: Cómo responder a un compromiso de AD
- 📊 **MITRE ATT&CK**: Técnicas contra AD (T1098, T1110, T1558, T1003)
- 🔐 **Kerberos**: Protocolo de autenticación en AD
- 🌐 **Windows Security Logs**: Fuente de EventIDs de AD
- 🔍 **Threat Hunting**: Buscar indicadores de compromiso en AD
- 🛠️ **Forensics**: Análisis de brechas relacionadas con AD

---

## 💾 TL;DR - Referencia Rápida

### Jerarquía de AD
```
FOREST → DOMAIN → OUs → OBJETOS (Usuarios, Grupos, Computadoras)
```

### EventIDs Críticos (para memorizar)
| EventID | Evento | Acción |
|---------|--------|--------|
| 4720 | Cuenta creada | Investigar si es a las 2-5 AM |
| 4728 | Agregado a grupo | Crítico si es grupo admin |
| 4723 | Contraseña cambiada | Crítico si es admin |
| 4625 | Login fallido | Alerta si 5+ en 5 min |
| 4624 | Login exitoso | Correlacionar con 4625 + firewall |
| 5136 | Objeto modificado | Cualquier cambio sensible |

### Técnicas de Ataque
```
T1110 → Brute Force
T1558 → Golden Ticket / Kerberoasting
T1003 → Credential Dumping
T1098 → Account Manipulation
T1484 → Domain Policy Modification (cambios en GPO)
```

### Indicadores de Compromiso (IOCs)
```
✓ Cuenta creada + agregada a admin group en <5 min
✓ Admin login desde IP externa a las 3 AM
✓ Cambio masivo de contraseñas para múltiples admins
✓ Cambios en GPO deshabilitando antivirus/audit
✓ EventID 4768 sin EventID 4624 previo (Golden Ticket posible)
```

### Defensa Básica
```
1. Políticas de contraseña fuerte (14+ caracteres, 90 días)
2. MFA para todos los admins
3. Monitoreo 24/7 de Tier 0 (Domain Admins)
4. Auditoría 100% de EventID críticos
5. PAM (Privileged Access Management)
```

---

## 📌 Realidad en Producción

### Caso Real #1: Compromiso de AD detectado por SIEM

```
Timeline:
- Viernes 18:00 → Último backup exitoso
- Viernes 22:14 → EventID 4720: Cuenta "admin_backup" creada
- Viernes 22:16 → EventID 4728: Agregado a Domain Admins
- Sábado 03:45 → EventID 5136: Cambio de GPO (antivirus deshabilitado)
- Sábado 04:20 → EventID 4769: Kerberoasting detectado
- Domingo 14:00 → Descubrimiento: 2 TB de datos exfiltrados

Lección: Las correlaciones de EventID detectaron el ataque
5+ horas antes del descubrimiento manual.
```

### Caso Real #2: False Positive común

```
❌ FALSA ALARMA:
Regla: "EventID 4728 + grupo Administrators = CRITICAL"

Realidad: Nuevo admin recién contratado siendo onboarded por IT manager.
Ocurrió a las 09:30 AM en horario laboral, con justificación válida.

✅ SOLUCIÓN:
Cambiar regla:
"EventID 4728 + Domain Admins + (hora < 6 AM OR hora > 22 PM) + IP externa = CRITICAL"
```

---

## 📚 Próximos Pasos

1. **Aprende los EventIDs de memoria** (4720, 4728, 4723, 4625, 5136)
2. **Entiende Kerberos** — es la base de toda autenticación AD
3. **Practica escribiendo reglas SIEM** para detectar patrones de ataque
4. **Estudia MITRE ATT&CK** específicamente técnicas contra AD
5. **Laboratorio**: Crea un AD pequeño, simula ataques, detecta en SIEM

---

## 📌 Recuerda

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Active Directory = El corazón de la red corporativa      │
│                                                            │
│  Si se compromete AD, se compromete la TODA red.         │
│                                                            │
│  Los atacantes buscan:                                     │
│  1) Acceso inicial (Brute Force, Kerberoasting)          │
│  2) Persistencia (Crear cuenta, Golden Ticket)           │
│  3) Escalada (Agregar a Domain Admins)                   │
│  4) Exfiltración (Cambiar GPO, dumping credenciales)     │
│                                                            │
│  Tu trabajo: Detectar cada paso.                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

**Última actualización**: 2024
**Versión**: 1.0
**Para**: Blue Team Students - TripleTen Bootcamp
