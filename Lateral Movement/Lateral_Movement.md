# 🔄 LATERAL MOVEMENT: El Arte de Moverse por la Red

---

## 📖 ¿QUÉ ES LATERAL MOVEMENT?

**Definición de 30 segundos:**
Lateral movement es la técnica que usa un atacante para **moverse de un host a otro dentro de la red**, después de haber comprometido inicialmente un dispositivo. No es escalar privilegios EN el mismo host; es **extender su acceso horizontalmente** a otros servidores, workstations, o dispositivos críticos.

**Fórmula mental:**
```
Initial Access (ej: phishing) 
    ↓
Privilege Escalation (admin en ESTE host)
    ↓
LATERAL MOVEMENT ← TÚ ESTÁS AQUÍ (moverse a OTROS hosts)
    ↓
Persistence + Exfiltration
```

---

## 🎯 ¿POR QUÉ UN PROFESIONAL DE SOC/BLUE TEAM NECESITA ESTO?

### En Entrevistas Te Preguntarán:
- *"¿Cómo detectarías un ataque de lateral movement en tu SIEM?"*
- *"¿Cuál es la diferencia entre privilege escalation y lateral movement?"*
- *"¿Qué indicadores buscarías para identificar Pass the Hash?"*
- *"¿Cómo investigarías si alguien accedió a un servidor que no debería?"*
- *"¿Qué eventos de Windows están asociados a lateral movement?"*

### En tu Trabajo en SOC:
- **60-70% de los brechas** usan lateral movement para expandir acceso
- Detectar lateral movement **ANTES de que lleguen a servidores críticos** es tu trabajo
- La mayoría de ataques se descubren por **anomalías en logs de lateral movement**, no por el acceso inicial
- Necesitas entender **técnicas de atacantes** para saber **qué buscar en defensa**

---

## 🔍 EL CONCEPTO DESGLOSADO

### **Parte 1: El Escenario Típico**

**ESCENARIO REAL EN SOC:**
```
09:15 - Ataque phishing → Usuario abre documento
        ✓ Atacante tiene acceso a workstation "LAPTOP-JUAN"
        ✓ Pero como usuario normal (sin admin)

09:20 - Privilege Escalation local
        ✓ Exploit de Windows (ej: CVE-2021-1732)
        ✓ Atacante ahora es SYSTEM en LAPTOP-JUAN

09:25 - Lateral Movement COMIENZA
        ✓ Atacante ve que hay un servidor "FILE-SERVER-01"
        ✓ Usa credenciales del admin para acceder vía SMB
        ✓ Ahora está en FILE-SERVER-01

09:30 - Movimiento continúa
        ✓ Desde FILE-SERVER-01, busca acceso a DB-SERVER
        ✓ Usa técnica Pass the Hash
        ✓ Volcado de datos sensibles

10:00 - TÚ (SIEM analyst) lo detectas en los logs
        ✓ Si lo hiciste bien, lo paras aquí
```

**¿Por qué no lo detectaste antes?** Porque los atacantes usan técnicas para que parezca legítimo.

---

### **Parte 2: ¿POR QUÉ ES TAN IMPORTANTE?**

#### El Problema:
Un servidor compromised en tu red es como tener un "puente" hacia todos los demás servidores que pueda alcanzar.

**Si un atacante tiene:**
- ✅ Admin en FILE-SERVER
- ✅ Permisos de red abiertos
- ✅ Credenciales válidas (robadas o recicladas)

**Puede llegar a:**
→ Database servers
→ Domain Controller
→ Backup systems
→ Cloud infrastructure
→ Sistemas críticos

#### Por qué los atacantes lo hacen:
1. **Expansión de acceso** → más hosts = más datos
2. **Redundancia** → si pierden un host, tienen 5 más
3. **Objetivos secundarios** → necesitan llegar a DB-SERVER, no solo FILE-SERVER
4. **Esconderse** → distribuir actividad en múltiples hosts evita detección

---

### **Parte 3: Técnicas Específicas de Lateral Movement**

**Ahora lo importante: CÓMO ATACAN = CÓMO TÚ DEFENSAS**

#### **#1: PASS THE HASH (PtH)**

**¿Qué es?**
```
Atacante obtiene el HASH NTLM de una cuenta (ej: administrator)
Sin necesitar la contraseña plaintext
Lo usa directamente en SMB, RDP, WinRM
Sistema: "Tienes el hash correcto → acceso concedido"
```

**¿Cómo funciona técnicamente?**
```
Windows autentica con NTLM así:
1. Cliente envía: username + HASH NTLM
2. Servidor valida el hash contra su base de datos
3. Si coincide → acceso

Diferencia con contraseña:
- Normal: contraseña → hash → comparación
- PtH: atacante ya tiene el hash → comparación directa
- Windows no detecta que la contraseña NO fue usada
```

**¿Cómo un atacante obtiene el hash NTLM?**
- Volcado de LSASS (Local Security Authority)
- Extracción del archivo SAM (con permisos SYSTEM)
- Herramientas: mimikatz, secretsdump.py, gsecdump

**Ejemplo de herramienta (para entender, no para usar):**
```bash
# Atacante con access a SYSTEM:
mimikatz.exe
mimikatz > privilege::debug
mimikatz > sekurlsa::logonpasswords
# Output: Hashes NTLM de usuarios actuales

# Luego, ataca a SMB con ese hash:
psexec.py -hashes :HASHNTLM usuario@target-server
```

**¿Por qué es peligroso?**
- No necesita descifrar la contraseña
- Contraseña nunca atraviesa la red
- Muchas herramientas aceptan hashes directamente
- Antigua técnica pero SIGUE FUNCIONANDO

---

#### **#2: PASS THE TICKET (PtT) - Kerberos**

**¿Qué es?**
```
En lugar de hash NTLM, atacante roba un Kerberos Ticket
(TGT = Ticket Granting Ticket)
Lo usa para autenticarse en otros hosts sin la contraseña
```

**¿Cuándo se usa?**
- Ambiente Windows con Active Directory
- Más moderno que PtH
- Kerberos es el estándar en AD

**Detalle técnico (lo que necesitas saber):**
```
Kerberos flow normal:
Usuario → solicita TGT al DC
DC → da TGT válido
Usuario → usa TGT para pedir acceso a servidor X

Ataque PtT:
Atacante roba el TGT
Usa ese TGT sin conocer la contraseña
Acceso concedido
```

---

#### **#3: RDP LATERAL MOVEMENT**

**¿Qué es?**
Atacante accede a otro host vía **Remote Desktop Protocol (RDP)** usando credenciales robadas.

**Pasos:**
```
1. Atacante tiene credenciales de usuario (admin o no)
2. Identifica servidor con RDP abierto (puerto 3389)
3. Conecta: mstsc.exe o xfreerdp a target
4. Acceso visual interactivo al servidor remoto
```

**¿Por qué es efectivo?**
- RDP es legítimo y común (IT usa RDP todos los días)
- Credenciales robadas pueden ser de cualquier usuario
- Si un admin usó RDP desde ese host, sus credenciales están en caché

---

#### **#4: SMB LATERAL MOVEMENT (Shares)**

**¿Qué es?**
Atacante accede a **carpetas compartidas** de otros hosts para:
- Explorar archivos sensibles
- Depositar malware
- Exfiltrar datos

**Pasos:**
```
1. Atacante en HOST-A
2. Ve que HOST-B tiene \\HOST-B\Shared abierto
3. Accede con credenciales comprometidas: net use \\HOST-B\Shared
4. Explora archivos / deposita payload
```

**Esto es CRÍTICO porque:**
- SMB es cómo Windows comparte archivos entre máquinas
- Muchos servidores abren shares sin restricción
- Logs de SMB te dirán QUÉ ARCHIVOS accedió

---

#### **#5: WINRM (Windows Remote Management)**

**¿Qué es?**
Atacante ejecuta comandos remotamente vía WinRM (puerto 5985).

**Comando típico:**
```powershell
# Atacante:
Invoke-Command -ComputerName TARGET-SERVER -Credential $cred -ScriptBlock { whoami }
```

**¿Por qué es peligroso?**
- Es legítimo (admins usan WinRM)
- Si el puerto está abierto y tienes credenciales → acceso
- Ejecución de comandos = control total

---

### **Parte 4: Diferencia CRÍTICA: Lateral Movement vs Privilege Escalation**

**ESTO SALE EN ENTREVISTAS:**

| Aspecto | Privilege Escalation | Lateral Movement |
|--------|---------------------|------------------|
| **¿Qué es?** | Pasar de usuario normal a admin EN EL MISMO HOST | Moverse de un host a OTRO host |
| **Ejemplo** | Usuario → SYSTEM en LAPTOP-JUAN | LAPTOP-JUAN (admin) → FILE-SERVER |
| **Requisito previo** | Initial access al host | Privilege escalation + credenciales del siguiente host |
| **Resultado** | Más permisos en 1 máquina | Acceso a múltiples máquinas |
| **Detecta** | Eventos de elevación de privilegios | Logs de acceso remoto a múltiples hosts |

**ORDEN CORRECTO DEL ATAQUE:**
```
1. INITIAL ACCESS (phishing, RCE) → acceso básico a 1 host
2. PRIVILEGE ESCALATION → admin en ese host
3. LATERAL MOVEMENT ← es DESPUÉS (necesitas admin para mover efectivamente)
4. PERSISTENCE (malware para mantener acceso)
5. EXFILTRATION (robar datos)
```

**¿Por qué el orden importa?**
Si atacante NO escala antes de moverse:
- Movimientos limitados (solo a shares donde tiene permiso)
- No puede volcar hashes
- No puede usar PtH

Si ESCALA primero:
- Puede volcar todas las credenciales (mimikatz)
- Puede usar PtH sin límites
- Acceso AGRESIVO a toda la red

---

## ⚙️ LO QUE DEBES MEMORIZAR

### **Memorización #1: Las 5 Técnicas Principales**

```
🔑 PtH (Pass the Hash)
   → Roba hash NTLM, lo usa directamente
   → Detectar: Event ID 4625 + 4624 sin cambio de password

🎫 PtT (Pass the Ticket)
   → Roba Kerberos ticket
   → Detectar: TGT usage sin evento de autenticación previo

🖥️ RDP
   → Acceso remoto visual
   → Detectar: Event ID 3389 desde IPs internas inusuales

📁 SMB
   → Acceso a shares de red
   → Detectar: Event ID 5140 (share access) + archivos no rutinarios

💻 WinRM
   → Comandos remotos
   → Detectar: Event ID 91 (WinRM connection)
```

### **Memorización #2: Los Event IDs que IMPORTAN**

```
Windows Event Viewer → Security logs:

4624 = Successful logon
4625 = Failed logon attempt
4672 = Special privileges assigned
4720 = User account created / password changed
3389 = RDP connection attempt
5140 = Share object accessed
5145 = Share object detailed check
91   = WinRM connection

Si ves: 4625 (fail) + 4624 (success) + SIN 4720 = PROBABLE PtH
```

### **Memorización #3: Indicadores de Lateral Movement**

**ROJO BRILLANTE 🚨 (Muy sospechoso):**
- ✗ Usuario accediendo a servidor que NUNCA accedió antes
- ✗ Acceso a servidores a las 3 AM (fuera de horario)
- ✗ Múltiples intentos fallidos de logon seguidos de acceso exitoso
- ✗ Uso de credenciales de admin en hosts que no son admin's típicos

**AMARILLO ⚠️ (Investigar):**
- Actividad SMB anormal (muchos archivos accedidos)
- WinRM desde IPs que normalmente no usan WinRM
- RDP desde workstations (usualmente es admin → workstation, no inverso)

---

## 📚 LO QUE DEBES ENTENDER PROFUNDAMENTE

### **Comprensión #1: ¿Por qué los atacantes necesitan múltiples cuentas?**

**Scenario:**
```
Atacante compromete cuenta: "juan@empresa.com" (usuario normal)
Escala a SYSTEM
¿Ahora puede acceder a FILE-SERVER?

Depende:
- Si FILE-SERVER requiere admin específico → NO
- Si FILE-SERVER tiene permisos abiertos → SÍ

Pero hay problema: Si hace 10 conexiones con JUAN,
el SOC ve patrón y sospecha.

SOLUCIÓN DEL ATACANTE: Usa múltiples cuentas
- Conexión 1 con juan@empresa.com
- Conexión 2 con maria@empresa.com
- Conexión 3 con admin-service@empresa.com
- Conexión 4 con backup-admin@empresa.com

Result: Logs dispersados, alertas en diferentes usuarios,
correlación difícil. Y si una cuenta está "deshabilitada",
tiene otras 3.
```

**Por qué es importante para Blue Team:**
- Debes correlacionar eventos DE MÚLTIPLES USUARIOS
- Si ves 4 usuarios diferentes accediendo a mismo servidor en 1 hora → RED FLAG
- No busques solo 1 usuario sospechoso, busca PATRONES

### **Comprensión #2: El "Slow and Low" en Lateral Movement**

**¿Qué es?**
Atacante se mueve LENTAMENTE para evitar detección.

**Ejemplo:**
```
Opción 1 (EVIDENTE - SOC lo ve inmediato):
09:15 - Conexión a FILE-SERVER
09:16 - Descarga 100 GB
09:17 - Conexión a DB-SERVER
09:18 - Descarga datos
09:19 - Conexión a DC (Domain Controller)
RESULTADO: Alert masivo, detección en minutos

Opción 2 (SLOW AND LOW - Difícil detectar):
09:15 - Conexión a FILE-SERVER (descarga 5 MB)
14:30 - Conexión a DB-SERVER (explora carpetas)
18:45 - Reconocimiento pasivo del DC (sin acceso)
Día 2: 11:00 - Acceso al DC

RESULTADO: Eventos distribuidos, parecen legítimos, difícil correlacionar
```

**Defensa contra esto:**
- Baseline de comportamiento (qué es "normal" para cada usuario)
- Machine learning en SIEM (detecta desviaciones leves)
- Correlación de eventos a través de HORAS/DÍAS, no solo minutos

### **Comprensión #3: Diferencia entre acceso legítimo y lateral movement**

**LO DIFÍCIL:** Un admin accediendo a FILE-SERVER vía RDP es... legítimo.
Un atacante accediendo a FILE-SERVER vía RDP también se ve igual en los logs básicos.

**¿CÓMO DIFERENCIAR?**

| Factor | Legítimo | Lateral Movement |
|--------|----------|------------------|
| **Hora** | Horario de trabajo | 3 AM / fin de semana |
| **Origen** | IP del admin conocida | IP diferente (workstation comprometida) |
| **Patrón** | Accede a mismo servidor siempre | Nuevo servidor cada vez |
| **Usuarios** | Mismo usuario siempre | Múltiples usuarios diferentes |
| **Velocidad** | Acciones normales de admin | Comandos/scripts automatizados muy rápido |
| **Objetivo** | Trabajo legítimo | Búsqueda de credenciales / datos sensibles |

**En SIEM:**
```
Red FLAG:
IF (usuario == "admin" AND 
    hora == "03:00 AM" AND 
    ip_origen DIFERENTE_DE_NORMAL AND 
    servidor NUNCA_ACCEDIDO_ANTES)
THEN Alert: "Posible lateral movement con credenciales de admin"
```

---

## 🚨 DETECCIÓN PRÁCTICA EN SOC (ENFOQUE BLUE TEAM)

### **¿Dónde ves los logs?**

1. **Windows Event Viewer** (local, pero impractical a escala)
2. **SIEM** (centralizado, recolecta de todos los hosts)
3. **Proxy logs** (si lateral movement atraviesa internet)
4. **Firewall logs** (conexiones entre hosts)
5. **Endpoint Detection & Response (EDR)** (muy sensible a movimientos)

### **Flujo de Investigación en SOC**

**PASO 1: Alerta llega al SIEM**
```
Alert: "Usuario 'juan' accedió a FILE-SERVER a las 02:45 AM"
(evento que normalmente NO hace)
```

**PASO 2: TÚ investigas**
```
Preguntas:
- ¿Es juan un admin? ¿Necesita acceso a FILE-SERVER?
- ¿De qué IP conectó?
- ¿Qué hizo después de conectar? (archivos, comandos)
- ¿Hay otros eventos sospechosos del mismo usuario esa noche?
```

**PASO 3: Correlación**
```
Si ves:
- 02:45 AM: juan → FILE-SERVER (RDP)
- 02:50 AM: juan → DB-SERVER (SMB)
- 02:55 AM: maria (cuenta diferente) → BACKUP-SERVER

Conclusión: Lateral movement coordinado con múltiples cuentas
```

**PASO 4: Containment (Aislamiento)**
```
Acciones:
- Resetear contraseña de juan
- Resetear contraseña de maria
- Revisar si los servidores fueron modificados
- Buscar persistencia (backdoors)
```

---

### **Búsquedas SIEM típicas para detectar Lateral Movement**

#### **Búsqueda 1: Acceso a servidor inusual**
```
event_id: (4624 OR 3389 OR 5140) AND
host: FILE-SERVER AND
user NOT IN (admin-juan, admin-maria, sistema) AND
timestamp: [today]

Resultado: Usuarios no autorizados accediendo a FILE-SERVER
```

#### **Búsqueda 2: Múltiples servidores en corto tiempo**
```
event_id: (4624 OR 3389) AND
timeline: 1 hour AND
count(distinct hosts) > 5

Resultado: Un usuario accediendo a 5+ servidores en 1 hora (sospechoso)
```

#### **Búsqueda 3: Pass the Hash indicator**
```
event_id: 4625 (failed login) AND
event_id: 4624 (successful login) AND
NO event_id: 4720 (password change) AND
mismo usuario Y host
within 2 minutes

Resultado: Patrón típico de PtH
```

#### **Búsqueda 4: RDP anormal**
```
event_id: 3389 AND
timestamp: (after hours OR weekend) AND
ip_origen: internal_subnet (no remote)

Resultado: Conexiones RDP internas en horarios no típicos
```

---

## ❌ ERRORES COMUNES QUE HACEN ESTUDIANTES

### **Error 1: Confundir lateral movement con privilege escalation**

**Estudiante dice:**
> "Lateral movement es cuando subes de usuario normal a admin"

**Corrección:**
- Eso es PRIVILEGE ESCALATION (escalar EN EL MISMO HOST)
- Lateral movement es moverse A OTRO HOST
- **Orden correcto:** Privilege escalation → Lateral movement

**Consecuencia en entrevista:**
Te preguntarán: "Describe el flujo de ataque después de compromise inicial"
Si dices "lateral movement" cuando es "privilege escalation" = fallo conceptual

---

### **Error 2: Creer que Pass the Hash necesita la contraseña**

**Estudiante dice:**
> "Pass the Hash es robar la contraseña y usarla en otro host"

**Corrección:**
- PtH NO necesita la contraseña
- Solo necesita el HASH NTLM
- Atacante NUNCA descubre la contraseña plaintext

**Consecuencia:**
Investigarás un evento PtH buscando "cambio de contraseña" = no lo encontrarás

---

### **Error 3: Solo buscar eventos de un usuario**

**Estudiante dice:**
> "Si busco eventos del usuario 'admin', veré todo el lateral movement"

**Corrección:**
- Atacante usa MÚLTIPLES CUENTAS
- Eventos están dispersados entre usuarios diferentes
- Debes correlacionar eventos DE MÚLTIPLES USUARIOS

**Consecuencia:**
Atacante se mueve 10 veces pero TÚ investigas 1 usuario → lo pierdes

---

### **Error 4: Investigar solo "logins exitosos"**

**Estudiante dice:**
> "Voy a buscar todos los Event ID 4624 (successful login)"

**Corrección:**
- Patrón de lateral movement TÍPICO: fallos + 1 éxito
- Muchos intentos fallidos = credential stuffing / ataque
- El éxito después de 20 fallos = RED FLAG

**Búsqueda correcta:**
```
4625 (failed) + 4624 (success) en mismo usuario/host dentro de 5 minutos
```

---

## 🧪 PRÁCTICA: ANALIZA ESTE ESCENARIO SOC

**ESCENARIO REAL:**
```
Timeline de eventos en tu SIEM:

01:30 AM - LAPTOP-MARIA (workstation)
  Event 4625: Failed RDP login to FILE-SERVER (usuario: admin-backup)
  Event 4625: Failed RDP login to FILE-SERVER (usuario: admin-backup)
  Event 4625: Failed RDP login to FILE-SERVER (usuario: admin-backup)
  Event 4624: Successful RDP login to FILE-SERVER (usuario: admin-backup)

01:35 AM - FILE-SERVER
  Event 5140: Share accessed - \\FILE-SERVER\Confidential
  Event 5140: Share accessed - \\FILE-SERVER\HR_DATA
  Event 5145: Object "salary_2024.xlsx" accessed
  Event 5145: Object "employee_list.csv" accessed

01:40 AM - FILE-SERVER
  Event 4624: Successful login to DB-SERVER (usuario: db-admin)

01:45 AM - DB-SERVER
  Event 91: WinRM command executed: "whoami"
  Event 91: WinRM command executed: "dir C:\Backups"

---

PREGUNTAS A RESPONDER:

1. ¿QUÉ INDICADORES VES DE LATERAL MOVEMENT?
2. ¿EN QUÉ ETAPA DEL ATAQUE CREES QUE ESTÁ?
3. ¿QUÉ HARÍAS AHORA COMO ANALYST DE SOC?
4. ¿CUÁL TÉCNICA USÓ? (PtH, RDP, SMB, WinRM, etc)
```

### **RESPUESTAS:**

**1. Indicadores de lateral movement:**
- ✓ Múltiples intentos fallidos de RDP (4625) → ataque de credenciales
- ✓ Éxito después de fallos → credenciales válidas encontradas
- ✓ Acceso a archivos sensibles (HR_DATA, salarios) → DATA HUNTING
- ✓ Movimiento a DB-SERVER → expansión de acceso
- ✓ WinRM en DB-SERVER → ejecución de comandos

**2. Etapa del ataque:**
- ✅ Initial access: SÍ (acceso a LAPTOP-MARIA)
- ✅ Privilege escalation: Probablemente SÍ (tienen credenciales de admin)
- ✅ Lateral movement: AHORA (FILE-SERVER → DB-SERVER)
- ⏳ Persistencia: Aún no vista
- ⏳ Exfiltración: En progreso (accediendo a datos sensibles)

**3. Acciones inmediatas:**
```
1. AISLAMIENTO INMEDIATO:
   - Desconectar LAPTOP-MARIA de la red
   - Resetear password de admin-backup
   - Resetear password de db-admin

2. INVESTIGACIÓN:
   - ¿Quién usa normalmente admin-backup?
   - ¿Fue ese usuario el que comprometió LAPTOP-MARIA?
   - ¿Qué más hizo en DB-SERVER?

3. BÚSQUEDA DE PERSISTENCIA:
   - ¿Creó nuevas cuentas?
   - ¿Instaló backdoor?
   - ¿Modificó archivos del sistema?

4. SCOPE:
   - ¿Hay otro hosts comprometidos?
   - ¿Extrajo datos? (revisar logs de exfiltración)
```

**4. Técnicas usadas:**
- RDP lateral movement (PORT 3389)
- SMB file share access
- WinRM command execution
- Posible: Pass the Hash (si el hash fue robado en LAPTOP-MARIA)

---

## 🎯 PREGUNTAS DE ENTREVISTA QUE TE HARÁN

### **Nivel Fácil (L1 - Analyst Junior):**

**P1:** "¿Qué es lateral movement y por qué es peligroso?"
**Respuesta esperada:**
> "Lateral movement es cuando un atacante que tiene acceso a un host intenta acceder a otros hosts en la red. Es peligroso porque si obtienen admin en un servidor, pueden comprometer toda la red. Es típicamente la fase después de privilege escalation."

**P2:** "¿Cuál es la diferencia entre RDP y SMB lateral movement?"
**Respuesta esperada:**
> "RDP es acceso visual remoto (como escritorio compartido). SMB es acceso a carpetas compartidas. RDP requiere un usuario interactivo, SMB es silencioso. Ambos pueden ser detectados en logs pero de formas diferentes."

**P3:** "¿Qué evento de Windows indica RDP login?"
**Respuesta esperada:**
> "Event ID 3389 en Security logs, o buscar conexiones al puerto 3389."

---

### **Nivel Medio (L2 - Analyst Senior):**

**P1:** "Describe el flujo completo de un ataque: Initial Access → Lateral Movement. ¿Qué debe pasar ANTES de lateral movement?"
**Respuesta esperada:**
> "Primero: Initial access (phishing). Segundo: Privilege escalation a admin en ESE host. TERCERO: Lateral movement. Sin admin, es muy limitado moverse a otros hosts. Con admin, puedo volcar credenciales, usar PtH, y acceder a cualquier sistema que tenga permiso de red."

**P2:** "¿Cómo detectarías Pass the Hash vs un RDP login legítimo?"
**Respuesta esperada:**
> "PtH tiene patrón: múltiples 4625 (fallos) seguidos de 4624 (éxito) sin cambio de password. RDP legítimo tendría líneas auditadas en login normal. Pero podría usar heurística: hora anormal, origen IP diferente, acceso a servidor no rutinario, velocidad de acceso anormal."

**P3:** "Si ves 5 usuarios diferentes accediendo a FILE-SERVER en 1 hora, ¿cuál es tu hipótesis?"
**Respuesta esperada:**
> "Hipótesis: Lateral movement con múltiples cuentas comprometidas. El atacante distribuye la actividad para evitar correlación. Investigaría: origen IP de todas esas conexiones (¿misma IP?), hora (¿todas fuera de horario?), qué archivos accedieron, si hay patrón de escalation posterior."

---

### **Nivel Difícil (L3 - Senior / Threat Hunter):**

**P1:** "Describe cómo construirías una búsqueda SIEM para detectar lateral movement multi-stage que ocurre durante días, no minutos."
**Respuesta esperada:**
> "Usaría behavioral baseline: qué es normal para ese usuario/host. Buscaría desviaciones: nuevos hosts accedidos, nuevas horas, nuevas IPs de origen. Correlacionaría eventos de múltiples usuarios con patrón similar. Usaría machine learning o statistical modeling para detectar 'comunidades' de actividad correlacionada."

**P2:** "¿Cómo diferenciastes entre administrador legítimo haciendo su trabajo y atacante usando credenciales de admin robadas?"
**Respuesta esperada:**
> "Contexto es crítico. Cosas que buscaría: ¿Es la hora/día típica? ¿IP origen es la usualmente en logs del admin? ¿Está visitando servidores que normalmente NO visita? ¿Velocidad de acciones? ¿Está buscando datos (carpetas de HR, salarios) vs mantenimiento rutinario? Correlacionaría con alertas de endpoint (mimikatz ejecutado) o cambios no autorizados en los hosts."

**P3:** "¿Qué alternativas a Pass the Hash existen y cómo las detectarías?"
**Respuesta esperada:**
> "Pass the Ticket (Kerberos) - buscaría TGT usage sin autenticación previa. Credential injection en memoria - necesitarías EDR para verlo. Credential stuffing automatizado - sería patrón de múltiples 4625. Cada una tiene signature diferente. PtH es antigua pero sigue siendo útil para atacantes porque muchos sistemas aún usan NTLM."

---

## 🔗 CÓMO LATERAL MOVEMENT CONECTA CON TODO LO DEMÁS

### **En el Ataque:**
```
Foundations (TCP/IP, authentication)
    ↓
Network Basics (subnets, routing)
    ↓
Initial Access (phishing, exploit)
    ↓
Privilege Escalation (exploit local)
    ↓
LATERAL MOVEMENT ← TÚ ESTÁS AQUÍ
    ↓
Active Directory (compromiso de DC)
    ↓
Persistence (backdoors)
    ↓
Incident Response (detección y mitigación)
```

### **En Defensa (Blue Team):**
```
Detection Engineering
  → ¿Qué queries escribo para detectar LM?
  
SIEM
  → ¿Dónde centralizo los logs?
  
Windows/Linux/AD
  → ¿Qué eventos genera cada OS durante LM?
  
DFIR
  → ¿Cómo investigo si ya ocurrió LM?
  
Incident Response
  → ¿Cómo contengo lateral movement en progreso?
```

### **En Entrevistas:**
```
"¿Cómo correlacionarías eventos de lateral movement?"
→ Respuesta usa: SIEM knowledge + AD knowledge + log analysis

"¿Cómo escribirías una detection rule?"
→ Respuesta usa: Detection Engineering + Windows events

"¿Cómo investigarías un host comprometido?"
→ Respuesta usa: DFIR + Windows forensics + AD knowledge
```

---

## 💾 RESUMEN ULTRA RÁPIDO (TL;DR)

| Concepto | Definición | Detectar |
|----------|-----------|----------|
| **Lateral Movement** | Movimiento de un host a otro | Múltiples hosts accedidos por mismo usuario en poco tiempo |
| **Pass the Hash** | Roba hash NTLM, lo usa sin contraseña | Event 4625 + 4624 sin event 4720 |
| **Pass the Ticket** | Roba Kerberos ticket | TGT usage sin autenticación previa |
| **RDP lateral** | Acceso remoto visual | Event ID 3389 desde IP interna anormal |
| **SMB lateral** | Acceso a shares de red | Event ID 5140 + archivos sensibles accedidos |
| **WinRM lateral** | Comandos remotos | Event ID 91 desde origen sospechoso |
| **Privilege Escalation** | User → Admin en MISMO host | Eventos de elevación de privilegios (4672) |
| **Diferencia crítica** | PE = mismo host, LM = otro host | Buscar eventos en hosts DIFERENTES |

---

## 📌 REALIDAD EN PRODUCCIÓN

### **En SOC real (90% de tu trabajo):**

**Escenario 1: Detección automatizada**
```
08:15 AM - Tu SIEM trigger alerta
"User 'admin-backup' RDP to FILE-SERVER at 02:45 AM (unusual hour)"

Tu trabajo:
1. ¿Es falso positivo? (busca contexto)
2. Si es real, ¿cuántos hosts fueron accedidos?
3. ¿Qué datos fueron accedidos?
4. Isolation inmediato
5. Threat hunt: ¿Hay otros hosts comprometidos?
```

**Escenario 2: Threat hunting (buscar lo que las alertas automáticas no ven)**
```
Tu auditor te dice: "Implementamos logs de SMB hace 3 días"

Tu tarea:
1. Buscar SMB patterns que DEBERÍAN ser alerts pero no lo son
2. Encontrar usuarios accediendo a shares anormales
3. Usar correlación multi-usuario
4. Documentar hallazgos
```

**Escenario 3: Respuesta a incidente**
```
Descubren malware en LAPTOP-JUAN

Tu investigación:
1. ¿Cuándo fue el primer acceso?
2. ¿A qué hosts accedió JUAN después?
3. ¿Qué cuentas comprometió?
4. ¿Cuándo fue detectado el lateral movement?
5. Remediation: resetear todas las credenciales usadas
```

### **Lo que te dirán los Senior Analysts:**

> "90% de los ataques sofisticados que vemos se descubren por LOGS DE LATERAL MOVEMENT, no por el acceso inicial. El acceso inicial es fácil de ocultar. El lateral movement es difícil de hacer silenciosamente. Así que DOMINA LA DETECCIÓN DE LATERAL MOVEMENT."

---

## 📚 REFERENCIAS Y LECTURA ADICIONAL

### **MITRE ATT&CK Framework:**
- **T1021: Remote Service Session Initiation**
  https://attack.mitre.org/techniques/T1021/
  - T1021.001: Remote Desktop Protocol
  - T1021.002: SMB/Windows Admin Shares
  - T1021.006: Windows Remote Management

- **T1550: Use Alternate Authentication Material**
  https://attack.mitre.org/techniques/T1550/
  - T1550.002: Pass the Hash
  - T1550.003: Pass the Ticket

- **T1570: Lateral Tool Transfer**
  https://attack.mitre.org/techniques/T1570/

### **Windows Event IDs (Microsoft):**
- Event ID 4624-4625: Logon events
- Event ID 4720: Password change
- Event ID 3389: RDP connections
- Event ID 5140-5145: SMB share access

### **Herramientas de estudios (NO para atacar):**
- Mimikatz (educational purposes only)
- Impacket suite (entendimiento de protocolos)
- wireshark (captura de tráfico)

### **Práctica segura:**
- HackTheBox: Máquinas con scenarios de lateral movement
- TryHackMe: Rooms sobre Active Directory attacks
- SANS Cyber Aces: Ejercicios de log analysis

---

## 🎓 PRÓXIMOS PASOS

**Después de dominar este tema, estudia:**

1. **Active Directory Attacks** (lateral movement es 80% AD-based)
2. **Privilege Escalation techniques** (requisito previo)
3. **Detection Engineering** (escribir tus propias reglas)
4. **Windows Forensics** (buscar evidencia después del ataque)
5. **Threat Hunting** (búsqueda proactiva)

---

**Última actualización:** Julio 2026
**Nivel:** Intermediate (post-Foundations)
**Tiempo de estudio:** 4-6 horas
**Necesario para:** SOC Analyst, Threat Hunter, Incident Response

