# 🎯 Framework MITRE ATT&CK: Guía Completa de Dominio para Blue Team

---

## 📖 ¿Qué Es Esto?

MITRE ATT&CK (Tácticas, Técnicas y Conocimiento Común del Adversario) es la **base de conocimiento reconocida globalmente y empírica de tácticas y técnicas del adversario** basada en datos de brechas del mundo real. No es teórico—está basado en ataques reales analizados por investigadores de seguridad. Piénsalo como un **mapa universal** donde cada ataque jamás documentado se categoriza en 14 fases de táctica, cada una conteniendo múltiples técnicas que muestran CÓMO los atacantes trabajan realmente. Para un analista de blue team, MITRE ATT&CK es tu **Piedra de Rosetta**—traduce registros de eventos de Windows crudos y tráfico de red en un lenguaje estandarizado que te ayuda a entender patrones de ataque, priorizar amenazas y construir reglas de detección.

---

## 🎯 ¿Por Qué Un Profesional de SOC/Blue Team Necesita Esto?

### **Escenarios Reales del Trabajo**

1. **Investigación de Incidente: "¿Qué Patrón de Ataque Es Este?"**
   - Investigas una brecha: Event 4688 (PowerShell), Event 7045 (servicio), Event 5140 (acceso de red)
   - Sin MITRE: "Se ve mal, pero ¿en dónde me enfoco?"
   - Con MITRE: "Esto mapea a Execution (T1059) → Persistence (T1543) → Lateral Movement (T1021). Conozco la cadena de ataque."
   - Ahora puedes predecir el próximo movimiento del atacante

2. **Threat Hunting: "¿Qué Debo Buscar?"**
   - Tu SOC tiene 500 máquinas Windows generando 1 millón de eventos por hora
   - Sin MITRE: Búsquedas aleatorias, esperanza de encontrar algo
   - Con MITRE: "Vimos robo de credenciales (T1110), así que ahora busca Lateral Movement (T1021). Revisa tickets Kerberos (T1558), conexiones RDP (T1021.001) y Pass-the-Hash (T1550.002)."
   - Buscas con propósito y encuentras brechas ocultas

3. **Ingeniería de Detección: "¿En Qué Debo Alertar?"**
   - Tu equipo necesita construir reglas SIEM
   - Sin MITRE: Construye reglas para eventos individuales (ruidoso, muchos falsos positivos)
   - Con MITRE: "Mapea Técnicas a Tácticas. Construye reglas que detecten combinaciones de técnicas (no solo eventos individuales). Alerta cuando ves Execution + Persistence + Lateral Movement en secuencia."
   - Tus detecciones atrapan ataques reales, no ruido

4. **Inteligencia de Amenazas: "¿Quién Nos Ataca?"**
   - Analista dice: "Vimos estas 12 técnicas usadas en el ataque"
   - Con MITRE: Verificas qué grupos APT usan esas mismas 12 técnicas
   - Ejemplo: Si ves T1021.001 (RDP), T1110 (fuerza bruta), T1005 (recolección de datos), podrías estar mirando a Wizard Spider o FIN7
   - Ahora conoces el playbook del atacante y puedes predecir sus próximos movimientos

### **Preguntas de Entrevista Que Enfrentarás**

- *"Camina a través de cómo mapearías un Event Log de Windows a técnicas MITRE ATT&CK."*
- *"Explica la diferencia entre una Táctica, una Técnica, y un Procedimiento en MITRE."*
- *"Descubres un ataque usando 7 técnicas MITRE diferentes. ¿Cómo priorizarías cuál investigar primero?"*
- *"Nombra 3 técnicas usadas en movimiento lateral y cómo detectarías cada una."*
- *"¿Cómo usarías MITRE para atribuir un ataque a un grupo de actor de amenaza específico?"*
- *"Diseña una regla de detección para la táctica Execution que atrape la mayoría de ataques basados en PowerShell pero minimice falsos positivos."*

---

## 🔍 El Concepto Desglosado

### **Parte 1: Los Fundamentos — ¿Qué Son Táctica, Técnica y Procedimiento?**

MITRE tiene tres capas que se construyen una sobre la otra:

#### **Capa 1: TÁCTICA (¿Cuál es el objetivo?)**

**Definición:** El objetivo táctico que un adversario intenta lograr en cada fase de su ataque.

**Hay 14 Tácticas en MITRE ATT&CK:**

| # | Táctica | Objetivo | Cuándo Ocurre |
|---|---------|----------|---------------|
| 1 | **Reconocimiento** | Reunir información sobre objetivo | ANTES de brecha |
| 2 | **Desarrollo de Recursos** | Construir infraestructura de ataque | ANTES de brecha |
| 3 | **Acceso Inicial** | Entrar al sistema | PRIMER paso |
| 4 | **Ejecución** | Ejecutar malware/herramientas | DURANTE brecha |
| 5 | **Persistencia** | Permanecer en el sistema | DESPUÉS de acceso |
| 6 | **Escalada de Privilegios** | Obtener derechos de admin | DURANTE brecha |
| 7 | **Evasión de Defensa** | Esconderse de herramientas de seguridad | A LO LARGO de brecha |
| 8 | **Acceso a Credenciales** | Robar contraseñas/tokens | DURANTE brecha |
| 9 | **Descubrimiento** | Mapear red/sistemas | DURANTE brecha |
| 10 | **Movimiento Lateral** | Moverse a otras máquinas | DESPUÉS de acceso |
| 11 | **Recolección** | Reunir datos de objetivo | ANTES de exfil |
| 12 | **Comando & Control** | Comunicarse con malware | A LO LARGO de brecha |
| 13 | **Exfiltración** | Robar datos afuera | CERCA del final |
| 14 | **Impacto** | Dañar/destruir sistemas | PASO FINAL |

**Insight clave:** Los atacantes no siempre usan los 14—eligen los que necesitan. Un grupo de ransomware podría saltar Recolección e ir directo a Impacto. Un espía podría saltar Impacto y enfocarse en Recolección.

---

#### **Capa 2: TÉCNICA (¿Cómo se logra la táctica?)**

**Definición:** El método específico usado para lograr una táctica.

**Ejemplo: Táctica Ejecución (T04) tiene muchas técnicas:**

```
Táctica: EJECUCIÓN (Ejecutar código)
├─ Técnica T1059: Intérprete de Comandos y Scripting
│  └─ CÓMO: Usar intérpretes incorporados (PowerShell, cmd, bash)
│
├─ Técnica T1106: API Nativa
│  └─ CÓMO: Llamar APIs de Windows directamente (bypassa cmd/PowerShell)
│
├─ Técnica T1053: Tarea Programada/Trabajo
│  └─ CÓMO: Crear tarea programada que ejecute malware al arranque
│
├─ Técnica T1609: Comando de Administración de Contenedores
│  └─ CÓMO: Si cloud: usar APIs de contenedor
│
└─ Técnica T1559: Comunicación Entre Procesos
   └─ CÓMO: El malware habla a otro proceso para ejecutar código
```

Cada técnica tiene múltiples **sub-técnicas** (la variante):

```
T1059: Intérprete de Comandos y Scripting
├─ T1059.001: PowerShell
├─ T1059.003: Windows Command Shell (cmd.exe)
├─ T1059.005: Visual Basic
├─ T1059.007: JavaScript
└─ T1059.008: Unix Shell
```

**Por qué importa para SOC:** 
- Misma táctica (Ejecución), múltiples técnicas (T1059, T1106, T1053)
- Misma técnica (T1059), múltiples sub-técnicas (PowerShell vs cmd)
- Una regla de detección para T1059.001 (PowerShell) no atrapará T1059.003 (cmd.exe)

---

#### **Capa 3: PROCEDIMIENTO (¿Cómo hace un grupo específico?)**

**Definición:** La implementación actual por un grupo de actor de amenaza real (APT).

**Ejemplo: APT-28 (Fancy Bear) ejecutando T1059.001 (PowerShell):**

```
Grupo: APT-28 (Rusia, Guccifer 2.0, atribuido a GRU)
Táctica: Ejecución
Técnica: T1059.001 (PowerShell)
Procedimiento: APT-28 usa comandos PowerShell codificados en Base64 para:
  1. Descargar herramientas adicionales
  2. Ejecutar comandos de reconocimiento (Get-AdUser, Get-ADComputer)
  3. Establecer comando & control

Evidencia:
- Usado en brecha DNC 2016
- Usado en ataque de cadena de suministro SolarWinds 2020
- Malware: Sofacy, NotPetya
```

**Por qué importa para SOC:**
- Puedes decir: "Este patrón de ataque coincide con procedimiento conocido de APT-28"
- Puedes predecir: "Si esto es APT-28, probablemente intentarán movimiento lateral siguiente usando Kerberos (siempre lo hacen)"
- Puedes preparar: "APT-28 apunta a campañas políticas e infraestructura crítica—ajusta postura defensiva"

---

### **Parte 2: Las 14 Tácticas Explicadas (Tu Modelo Mental)**

#### **ANTES DE LA BRECHA (Tácticas 1-2)**

**Táctica 1: RECONOCIMIENTO**
- Atacante reúne información sobre ti antes de atacar
- Ejemplos: Revisar tu sitio web, escanear tu rango IP, encontrar empleados en LinkedIn
- Relevancia SOC: No verás estos en logs (ocurre externamente)
- Pero equipos de inteligencia rastrea esto para advertencia temprana

**Táctica 2: DESARROLLO DE RECURSOS**
- Atacante construye herramientas e infraestructura
- Ejemplos: Registrar dominio malicioso, configurar botnet, comprar exploit zero-day
- Relevancia SOC: Firewall bloquea dominios maliciosos, DNS bloquea servidores C&C
- Prevención: Trabaja con inteligencia de amenazas para identificar dominios maliciosos temprano

---

#### **BRECHA INICIAL (Táctica 3)**

**Táctica 3: ACCESO INICIAL**
- Atacante entra al sistema por primera vez
- Técnicas Principales:
  - **T1566: Phishing** (email con link/attachment malicioso)
  - **T1133: Servicios Remotos Externos** (RDP expuesto a internet, atacante fuerza bruta)
  - **T1199: Relación de Confianza** (Ataca socio/vendor, usa ese acceso para ti)
  - **T1190: Exploit de Aplicación Facing Pública** (Vulnerabilidad de sitio web)
  - **T1195: Compromiso de Cadena de Suministro** (Ataca cadena de suministro de software)

**Detección SOC:**
```
Si T1566 (Phishing):
  → Event 4688: Usuario hace click en link, malware.exe se lanza
  → Event 5156: Conexión saliente a servidor del atacante

Si T1133 (Fuerza Bruta de RDP):
  → Event 4625: 200 intentos fallidos de RDP en 10 minutos
  → Event 4624: RDP exitoso desde IP externa
```

**Tu trabajo:** Detecta y bloquea en ESTA táctica si es posible. Todo lo demás sigue.

---

#### **FASE DE EJECUCIÓN (Táctica 4)**

**Táctica 4: EJECUCIÓN**
- Atacante ejecuta código/malware en el sistema
- Técnicas Principales:
  - **T1059: Intérprete de Comandos y Scripting** (PowerShell, cmd, bash)
  - **T1106: API Nativa** (Llamar APIs de Windows directamente—más difícil de detectar)
  - **T1053: Tarea Programada/Trabajo** (Crear tarea para ejecutar malware después)
  - **T1648: Ejecución Serverless** (Cloud: AWS Lambda, Azure Functions)
  - **T1204: Ejecución de Usuario** (Usuario hace doble click en malware—ingeniería social)

**Detección SOC:**
```
Event 4688: Creación de Proceso
Image: powershell.exe
CommandLine: powershell.exe -enc JABzAGM= [COMANDO CODIFICADO]
Parent: explorer.exe

→ Mapea a T1059.001 (PowerShell)
```

**Insight crítico:** Event 4688 es tu fuente PRIMARIA para detectar técnicas de Ejecución. Si dominas Event 4688 + mapeo MITRE, atrapas 70% de ataques en esta fase.

---

#### **FASE DE PERSISTENCIA (Táctica 5)**

**Táctica 5: PERSISTENCIA**
- Atacante asegura que permanecen en sistema incluso después de reinicio
- Técnicas Principales:
  - **T1547: Ejecución de Autostart de Arranque o Logon** (Claves de Registro, carpetas de startup)
  - **T1543: Crear o Modificar Proceso de Sistema** (Servicio malicioso, driver)
  - **T1053: Tarea Programada/Trabajo** (Tarea se ejecuta al arranque o en cronograma)
  - **T1136: Crear Cuenta** (Crear cuenta usuario backdoor)
  - **T1137: Inicio de Aplicación de Office** (Macro malicioso en Office)

**Detección SOC:**
```
Event 7045: Servicio Instalado
ServiceName: WindowsUpdate (nombre legítimo, binario malicioso)
ImagePath: C:\ProgramData\malware.exe

→ Mapea a T1543.003 (Servicio de Windows)

Event 4698: Tarea Programada Creada
TaskName: \Microsoft\Windows\System Restore\SystemRestore
Command: C:\malware.exe

→ Mapea a T1053.005 (Tarea Programada)

Event 4720: Cuenta de Usuario Creada
NewAccountName: svc_admin (cuenta backdoor)

→ Mapea a T1136.001 (Cuenta Local)
```

**Tu trabajo:** Si pierdes Acceso Inicial, atrápales aquí. DEBEN crear persistencia o perder acceso al reinicio.

---

#### **ESCALADA DE PRIVILEGIOS (Táctica 6)**

**Táctica 6: ESCALADA DE PRIVILEGIOS**
- Atacante sube de usuario → admin o admin local → admin de dominio
- Técnicas Principales:
  - **T1548: Abusar Mecanismo de Control de Elevación** (Bypass UAC)
  - **T1134: Manipulación de Token de Acceso** (Robo de token, suplantación de token)
  - **T1547: Ejecución de Autostart de Arranque o Logon** (Modificar proceso de arranque)
  - **T1542: Pre-OS Boot** (Modificar BIOS/firmware—raro pero devastador)
  - **T1611: Escape a Host** (Escapar de contenedor a SO host)

**Detección SOC:**
```
Event 4688: Creación de Proceso
Image: C:\temp\uacbypass.exe
ParentImage: explorer.exe
IntegrityLevel: Medio → Alto (¡escalación!)

→ Mapea a T1548 (UAC Bypass)

Event 4720: Cuenta de Usuario Creada
NewAccountName: admin_backup
Comment: "(vacío)" o sospechoso
GroupMembership: Administradores

→ Mapea a T1136 (Crear Cuenta Privilegiada)
```

**Por qué importa:** Una vez que atacante tiene admin, pueden:
- Leer todos los archivos
- Instalar persistencia
- Crear cuentas backdoor
- Moverse lateralmente a todas las máquinas en dominio

---

#### **EVASIÓN DE DEFENSA (Táctica 7)**

**Táctica 7: EVASIÓN DE DEFENSA**
- Atacante esconde sus huellas de herramientas de seguridad
- Técnicas Principales:
  - **T1027: Ofuscación o Codificación** (Codificar PowerShell, esconder strings)
  - **T1562: Afectar Defensas** (Desactivar Windows Defender, Firewall, logging)
  - **T1070: Remoción de Indicador** (Borrar logs, limpiar historial)
  - **T1036: Enmascaramiento** (Malware nombrado svchost.exe, se ve legítimo)
  - **T1556: Modificar Proceso de Autenticación** (Bypass MFA, instalar autenticador falso)

**Detección SOC:**
```
Event 1102: Registro de Auditoría Borrado
ClearedBy: svc_admin
Time: 2024-01-15 03:05:00

→ ESCALACIÓN INMEDIATA
→ Mapea a T1070.001 (Borrar Event Logs de Windows)

Event 4104: Registro de Bloque de Script de PowerShell
ScriptBlockText: Set-MpPreference -DisableRealtimeMonitoring $true

→ Mapea a T1562.001 (Desactivar o Modificar Herramientas)

Event 4688: Creación de Proceso
Image: powershell.exe
CommandLine: powershell.exe -enc JABzAGM= [CODIFICADO]

→ Mapea a T1027.010 (Ofuscación de Comandos)
```

**Crítico:** T1070 (Remoción de Indicador) es la MAYOR bandera roja. Si ves Event 1102 (log borrado), asume brecha activa y escala inmediatamente.

---

#### **ACCESO A CREDENCIALES (Táctica 8)**

**Táctica 8: ACCESO A CREDENCIALES**
- Atacante roba contraseñas, tokens, o credenciales hash
- Técnicas Principales:
  - **T1110: Fuerza Bruta** (Adivina contraseñas offline u online)
  - **T1056: Captura de Entrada** (Keylogger, herramienta de screenshot)
  - **T1187: Autenticación Forzada** (Engañarte para dar credenciales)
  - **T1111: Canales Multi-Etapa** (Phishing de credenciales desde página de login falsa)
  - **T1621: Generación de Solicitud de Autenticación Multi-Factor** (Ataque de fatiga MFA)

**Detección SOC:**
```
Event 4625: Fallo de Inicio de Sesión (repetido)
TargetUserName: admin
FailureReason: Contraseña inválida
Count: 200 en 10 minutos

→ Mapea a T1110 (Fuerza Bruta)

Event 4688: Creación de Proceso
Image: mimikatz.exe (herramienta de dumper de credenciales)

→ Mapea a T1003 (Dumping de Credenciales del SO)

Event 5140: Acceso a Recurso Compartido de Red
ShareName: \\server\C$ (acceso a share admin = sospechoso)

→ Mapea a T1550.002 (Pass-the-Hash)
```

**Por qué importa:** Una vez que atacante tiene credenciales, Movimiento Lateral se vuelve fácil.

---

#### **DESCUBRIMIENTO (Táctica 9)**

**Táctica 9: DESCUBRIMIENTO**
- Atacante mapea la red, encuentra sistemas, identifica objetivos
- Técnicas Principales:
  - **T1087: Descubrimiento de Cuenta** (Enumera cuentas de usuario: Get-AdUser)
  - **T1010: Enumera Usuarios Locales y de Red** (net user, whoami)
  - **T1580: Descubrimiento de Infraestructura Cloud** (Encontrar recursos cloud)
  - **T1538: Descubrimiento de Servicio Cloud** (Listar buckets, bases de datos)
  - **T1217: Bookmarks del Navegador** (Encontrar herramientas/URLs internas)

**Detección SOC:**
```
Event 4688: Creación de Proceso (Herramientas de Reconocimiento)
Image: powershell.exe
CommandLine: Get-AdUser -Filter * | Select Name

→ Mapea a T1087 (Descubrimiento de Cuenta)

Event 4688: Creación de Proceso
CommandLine: net view \\domaincontroller

→ Mapea a T1010 (Enumera Recursos de Red)

Event 5140: Acceso de Objeto de Recurso Compartido de Red
ShareName: \\server\C$, \\server\Admin$
IntegrityLevel: Acceso desde cuenta admin

→ Mapea a T1135 (Descubrimiento de Recurso Compartido de Red)
```

**Por qué importa:** Descubrimiento te dice qué datos existen y dónde. Los atacantes SIEMPRE hacen esto antes de robar.

---

#### **MOVIMIENTO LATERAL (Táctica 10)**

**Táctica 10: MOVIMIENTO LATERAL**
- Atacante se mueve de una máquina a otra en la red
- Técnicas Principales:
  - **T1021: Servicios Remotos** (RDP, SSH, WinRM)
  - **T1550: Usar Material de Autenticación Alterno** (Pass-the-Hash, Pass-the-Ticket, robo de token)
  - **T1570: Transferencia de Herramienta Lateral** (Copiar herramientas entre máquinas)
  - **T1570: Transferencia de Herramienta Lateral** (Herramientas se propagan lateralmente)
  - **T1091: Replicación a través de Medios Removibles** (Gusano USB)

**Detección SOC:**
```
Event 4625: Fallo de Inicio de Sesión (IP externa)
+ Event 4624: Inicio de Sesión Exitoso (misma IP externa)
SourceIp: 10.0.1.50 (máquina interna—atacante moviéndose lateralmente)

→ Mapea a T1021.001 (RDP)

Event 4769: Ticket de Servicio Kerberos Solicitado
ServiceName: (cuenta de servicio inusual)
UserName: admin

→ Mapea a T1550.003 (Pass-the-Ticket / Kerberoasting)

Event 5140: Acceso a Recurso Compartido de Red
ShareName: \\other_server\Finance
SourceMachine: endpoint_comprometido

→ Mapea a T1570 (Movimiento Lateral vía Recurso Compartido de Red)
```

**Por qué importa:** Esto es DÓNDE se propagan las brechas. Detén movimiento lateral y contiene la brecha.

---

#### **RECOLECCIÓN (Táctica 11)**

**Táctica 11: RECOLECCIÓN**
- Atacante reúne datos objetivo (por qué vinieron)
- Técnicas Principales:
  - **T1123: Captura de Audio** (Grabar conversaciones)
  - **T1119: Exfiltración Automatizada** (Configurar recolección automática de datos)
  - **T1115: Datos de Portapapeles** (Robar contenidos del portapapeles)
  - **T1005: Datos del Sistema Local** (Copiar archivos del disco)
  - **T1114: Recolección de Email** (Exportar buzón, robar emails)

**Detección SOC:**
```
Event 5140: Acceso a Recurso Compartido de Red
ShareName: \\server\Finance, \\server\HR, \\server\Payroll
AccessMask: Lectura
SourceMachine: endpoint_comprometido
TimeOfDay: 03:00 AM (fuera de horario)

→ Mapea a T1005 (Datos del Sistema Local)

Log de Firewall: Conexión Saliente Grande
DestinationIP: servidor_atacante
BytesTransferred: 5 GB
Protocol: FTP/HTTPS

→ No es Recolección en sí, pero EVIDENCIA de que recolección ocurrió
```

**Por qué importa:** Este es el objetivo del atacante. Están robando tus joyas de la corona aquí.

---

#### **COMANDO & CONTROL (Táctica 12)**

**Táctica 12: COMANDO & CONTROL**
- Atacante se comunica con malware para darle comandos
- Técnicas Principales:
  - **T1071: Protocolo de Capa de Aplicación** (HTTP/HTTPS C&C)
  - **T1008: Canales Fallback** (Si C&C primario está caído, usa respaldo)
  - **T1001: Ofuscación de Datos** (Encriptar tráfico C&C)
  - **T1095: Protocolo de Capa No-Aplicación** (Protocolo personalizado, no HTTP)
  - **T1571: Puerto No-Estándar** (C&C en puertos inusuales)

**Detección SOC:**
```
Event 5156: Conexión de Red
DestinationIP: IP_sospechosa
DestinationPort: 8080 (inusual)
Protocol: HTTP
ProcessImage: malware.exe
Frequency: Cada 5 minutos (patrón beacon)

→ Mapea a T1071.001 (Protocolo de Capa de Aplicación - HTTP)

Log DNS:
Query: malware.attacker.com
Frequency: Cada 5 minutos
SourceIP: endpoint_comprometido

→ Mapea a T1071.004 (DNS)
```

**Por qué importa:** Tráfico C&C es evidencia de que malware está ACTIVO. Detén esto y has neutralizado el ataque.

---

#### **EXFILTRACIÓN (Táctica 13)**

**Táctica 13: EXFILTRACIÓN**
- Atacante transfiere datos robados fuera de la red
- Técnicas Principales:
  - **T1048: Exfiltración sobre Protocolo Alterno** (No HTTP: FTP, SFTP, DNS)
  - **T1041: Exfiltración sobre Canal C2** (Usa conexión C&C existente)
  - **T1020: Exfiltración Automatizada** (Malware automáticamente envía datos)
  - **T1030: Límites de Tamaño de Transferencia de Datos** (Trozo archivos grandes en pedazos más pequeños)
  - **T1537: Transferir Datos a Cuenta de Cloud** (Almacenamiento cloud: AWS S3, Azure Blob)

**Detección SOC:**
```
Log de Firewall: Transferencia Saliente Grande
Source: endpoint_comprometido
Destination: servidor_atacante O almacenamiento_cloud
Direction: Saliente
Volume: 10 GB+
Protocol: HTTPS/FTP

→ Mapea a T1041 o T1048

Log DNS: Intento de DNS Tunneling
Query: [datos_grandes_codificados].attacker.com
Frequency: Continuo

→ Mapea a T1048.003 (Exfiltración sobre Protocolo No-C2 sin Encripción/Ofuscado)
```

**Por qué importa:** Si los datos salen de la red, la brecha está COMPLETA. Detén fases anteriores para prevenir esto.

---

#### **IMPACTO (Táctica 14)**

**Táctica 14: IMPACTO**
- Atacante daña, destruye o interrumpe sistemas
- Técnicas Principales:
  - **T1486: Encriptar Datos Sensibles** (Ransomware)
  - **T1561: Wipe de Disco** (Destruir datos en disco)
  - **T1491: Desfiguración** (Cambiar apariencia de sitio web)
  - **T1531: Remoción de Acceso a Cuenta** (Bloquear usuarios de cuentas)
  - **T1529: Detención de Servicio** (Apagar servicios críticos)

**Detección SOC:**
```
Event 4688: Creación de Proceso
Image: ransomware.exe
CommandLine: ransomware.exe /encrypt /key:stolen_key

→ Mapea a T1486 (Encriptar Datos Sensibles)

Cambios en Sistema de Archivos: Archivos renombrados a .encrypted
Created: ransom_note.txt
Todos los archivos en C:\Users\* modificados

Event 5145: Acceso de Objeto de Recurso Compartido de Red
ShareName: \\server\*
AccessType: Escritura (cambiando archivos)

→ Mapea a T1561 (Wipe de Disco) o T1486 (Encriptación)
```

**Por qué importa:** Si alcanzas esta táctica, la contención falló. La prevención debe ocurrir MUCHO antes.

---

### **Parte 3: Mapeo de Eventos de Windows a Tácticas MITRE**

**Esta es la HABILIDAD CRÍTICA que te hace un analista top:**

Cada Evento de Windows puede (y debe) ser mapeado a una Táctica y Técnica MITRE.

```
LÍNEA DE TIEMPO COMPLETA DE ATAQUE CON MAPEO MITRE:

HORA | EVENTO | ID WINDOWS | TÁCTICA | TÉCNICA | ACCIÓN
-----|--------|------------|---------|---------|--------
23:15| RDP fuerza bruta comienza | 4625 | Acceso Inicial | T1133 | ALERTA: Bloquear IP
23:47| 200 intentos fallidos | 4625 x200 | Acceso Inicial | T1110 | ESCALAR
00:05| RDP exitoso | 4624 | Acceso Inicial | T1133 ✓ | AISLAR MÁQUINA
00:12| Atacante ejecuta cmd | 4688 | Ejecución | T1059.003 | RASTREAR PROCESO
00:15| PowerShell codificado | 4688 | Ejecución+Evasión | T1059.001+T1027 | MATAR PROCESO
00:18| Servicio instalado | 7045 | Persistencia | T1543.003 | ELIMINAR SERVICIO
00:20| Cuenta backdoor | 4720 | Persistencia | T1136.001 | DESACTIVAR CUENTA
00:25| Acceso a share admin | 5140 | Movimiento Lateral | T1021.002 | BLOQUEAR ACCESO
03:00| Ransomware se ejecuta | 4688 | Ejecución+Impacto | T1486 | CONTENCIÓN FALLA
03:05| Logs borrados | 1102 | Evasión de Defensa | T1070.001 | ASUMIR BRECHA
```

---

## ⚙️ Lo Que DEBES Memorizar

### **Las 14 Tácticas (En Orden de Ataque)**

**Mnemotécnica: "RRIEPD ECLCI"** (suena como "Reap-ed Eckly")

Mejor mnemotécnica en **MIX ESPAÑOL/INGLÉS:**

**"RAPI-PED-ECCE"**
- **R**econocimiento
- **A**cceso (Acceso Inicial)
- **P**ersistencia
- **I**nfiltración (Ejecución en español = "ejecución", pero piensa "infiltra y ejecuta")
---
- **P**rivilegio (Escalada de Privilegios = "escalada de privilegios")
- **E**vasión (Evasión de Defensa)
- **D**escubrimiento (Descubrimiento)
---
- **E**ncuentra (Movimiento Lateral = "encuentra" = encontrar/encontrase con otros sistemas)
- **C**redenciales (Acceso a Credenciales)
- **C**omando (Comando & Control)
- **E**xfiltración
- **I**mpacto (Impacto)

---

**Mejor: Orden Secuencial Mnemotécnica:**

```
PRIMERA FASE (Antes de Brecha):
1. Reconocimiento (👁️ Espiar)
2. Desarrollo de Recursos (🔧 Preparar)

SEGUNDA FASE (Entrar):
3. Acceso Inicial (🚪 Entrar)
4. Ejecución (▶️ Ejecutar)
5. Persistencia (🔒 Quedarse)

TERCERA FASE (Ir Profundo):
6. Escalada de Privilegios (📈 Subir)
7. Evasión de Defensa (🎭 Esconder)
8. Acceso a Credenciales (🔑 Robar Llaves)
9. Descubrimiento (🗺️ Mapear)
10. Movimiento Lateral (🚶 Moverse)

FASE FINAL (Salir):
11. Recolección (🎁 Reunir)
12. Comando & Control (📞 Controlar)
13. Exfiltración (📤 Exportar)
14. Impacto (💥 Dañar)
```

---

### **Top 30 Técnicas Que DEBES Conocer**

Estas son las técnicas que verás en 90% de brechas reales:

| Táctica | ID Técnica | Nombre Técnica | Herramienta Común | Evento Windows |
|---------|------------|----------------|------------------|----------------|
| Acceso Inicial | T1566 | Phishing | Adjunto de email | 4688 (malware runs) |
| Acceso Inicial | T1133 | Servicios Remotos Externos | RDP fuerza bruta | 4625→4624 |
| Ejecución | T1059.001 | PowerShell | Scripts codificados | 4688 |
| Ejecución | T1059.003 | Cmd.exe | Comandos del sistema | 4688 |
| Ejecución | T1106 | API Nativa | Llamadas API de Windows | 4688 (indirecto) |
| Ejecución | T1053.005 | Tarea Programada | Task Scheduler | 4698 |
| Persistencia | T1547 | Autostart de Arranque o Logon | Registry + .lnk | 4104 (Registry) |
| Persistencia | T1543.003 | Servicio de Windows | Svc malicioso | 7045 |
| Persistencia | T1053.005 | Tarea Programada | Tarea se ejecuta al arranque | 4698 |
| Persistencia | T1136.001 | Crear Cuenta Local | Usuario backdoor | 4720 |
| Escalada Privil | T1548 | Abusar Control de Elevación | UAC bypass | 4688 |
| Escalada Privil | T1134 | Manipulación de Token | Robo de token | 4688 (indirecto) |
| Evasión Defensa | T1027 | Ofuscación | Codificar comandos | 4104 (Script block) |
| Evasión Defensa | T1562.001 | Desactivar Defender | Cambio de Registry | 4104 |
| Evasión Defensa | T1070.001 | Borrar Event Logs Windows | Deletion de log | 1102 |
| Acceso Credenc | T1110 | Fuerza Bruta | Adivinanza de contraseña | 4625 (intentos fallidos) |
| Acceso Credenc | T1003 | Dumping de Credenciales SO | Mimikatz | 4688 (mimikatz.exe) |
| Acceso Credenc | T1556 | Modificar Autenticación | Desactivar MFA | 4104 (Cambio de Policy) |
| Descubrimiento | T1087 | Descubrimiento de Cuenta | Get-ADUser | 4688 (PowerShell) |
| Descubrimiento | T1010 | Enumera Usuarios Locales | whoami, net user | 4688 |
| Descubrimiento | T1135 | Descubrimiento de Recurso Compartido | net view, puerto 135 | 4688 |
| Movim Lateral | T1021.001 | RDP | Windows RDP | 4625→4624 (remoto) |
| Movim Lateral | T1021.006 | Recurso Compartido Admin Windows | C$, Admin$, IPC$ | 5140 |
| Movim Lateral | T1550.002 | Pass-the-Hash | Herramientas PtH | 4624 (cuenta inusual) |
| Movim Lateral | T1558.003 | Kerberoasting | hashcat, crack | 4769 (tickets inusuales) |
| Recolección | T1005 | Datos del Sistema Local | Copiar archivos | 5140 (acceso a recurso) |
| Recolección | T1114 | Recolección de Email | Exportar buzón | 5140 (acceso a buzón) |
| C&Control | T1071.001 | HTTP | Beacon sobre HTTP | 5156 (red) |
| Exfiltración | T1041 | Exfiltración sobre C2 | Robar vía C&C | 5156 (transferencia grande) |
| Impacto | T1486 | Encriptar Datos Sensibles | Ransomware | 4688 (exe runs) |

---

## 📚 Lo Que DEBES Entender

### **Entendimiento 1: No Todos Los Ataques Usan Las 14 Tácticas**

Diferentes tipos de atacantes usan tácticas diferentes:

```
GRUPO RANSOMWARE (FIN7):
Reconocimiento → Desarrollo de Recursos → Acceso Inicial → Ejecución →
Persistencia → Escalada de Privilegios → Acceso a Credenciales →
Movimiento Lateral → Recolección → C&C → Exfiltración → Impacto
(Las 14, porque quieren máximo daño)

ESPÍA (APT-28/Rusia):
Reconocimiento → Desarrollo de Recursos → Acceso Inicial → Ejecución →
Persistencia → Escalada de Privilegios → Descubrimiento → Acceso a Credenciales →
Movimiento Lateral → Recolección → C&C
(Saltan Exfiltración+Impacto—no quieren ser notados)

OPORTUNISTA (Script Kiddie):
Acceso Inicial → Ejecución → Impacto
(Tácticas mínimas—solo quieren causar caos rápido)

MINERO DE CRIPTOMONEDAS:
Acceso Inicial → Ejecución → Persistencia → C&C
(Saltan Movimiento Lateral—no necesitan moverse, solo CPU)
```

**Para SOC:** Conoce qué tácticas priorizar basado en perfil de actor de amenaza.

---

### **Entendimiento 2: Cadenas de Tácticas MITRE Muestran Progresión de Ataque**

Un ataque típico sigue este patrón:

```
Acceso Inicial (¿Cómo entraron?)
         ↓
    Ejecución (¿Qué ejecutaron?)
         ↓
    Persistencia (¿Cómo se quedaron?)
         ↓
    Escalada de Privilegios (¿Subieron?)
         ↓
    Descubrimiento/Movimiento Lateral (¿Exploraron/se movieron?)
         ↓
    Recolección (¿Qué robaron?)
         ↓
    Exfiltración (¿Sacaron datos?)
         ↓
    Impacto (¿Causaron daño?)
```

**Para detección:** Si paras en Paso 1, no hay brecha. Si paras en Paso 3, contenida. Si alcanzas Paso 7, perdiste.

---

### **Entendimiento 3: Evasión de Defensa es Continua**

Evasión de Defensa (T07) NO es un paso único—es **a lo largo de TODO el ataque**:

```
Reconocimiento (T01)
└─ Evasión de Defensa: VPN, identidad falsificada
    
Acceso Inicial (T03)
└─ Evasión de Defensa: Phishing se ve legítimo
    
Ejecución (T04)
└─ Evasión de Defensa: Codificar PowerShell
    
Persistencia (T05)
└─ Evasión de Defensa: Nombre de servicio legítimo (WindowsUpdate)
    
Movimiento Lateral (T10)
└─ Evasión de Defensa: Usar herramientas admin legítimas (PsExec, RDP)
    
Recolección (T11)
└─ Evasión de Defensa: Acceder fuera de horario, usar herramientas legítimas
```

**Para detección:** Cada táctica puede tener Evasión de Defensa mezclada. Un comando PowerShell "legítimo" podría ser malicioso.

---

### **Entendimiento 4: Procedimientos Muestran Patrones de Grupo**

Diferentes grupos APT tienen firmas:

```
APT-28 (Rusia/GRU):
- Siempre usa T1059.001 (PowerShell con ofuscación)
- Siempre usa T1550.003 (Pass-the-Ticket para movimiento lateral)
- Firma: Codificación Base64 + reconocimiento Get-AdUser

Lazarus (Corea del Norte):
- Siempre usa malware personalizado (no off-the-shelf)
- Firma: T1059 con lenguaje de scripting personalizado
- Siempre incluye componente destructivo (T1561 Disk Wipe)

FIN7 (Desconocido/Financiero):
- Siempre usa T1021.001 (RDP para persistencia)
- Usa herramientas legítimas (LOLBins)
- Firma: Infección multi-etapa (phishing → downloader → payload)
```

**Para threat hunting:** Si ves firma de APT-28, podrías estar apuntando a APT-28. Ajusta estrategia de hunting.

---

## 🚨 Aplicación Práctica: Investigación Real de Ataque

### **Escenario: Recibes Alerta a las 2 PM el Viernes**

**Alerta:** "Ejecución sospechosa de PowerShell detectada en MARKETING-PC-05"

**Eventos Crudos:**

```
14:22 - Event 4688: ejecución de powershell.exe
        CommandLine: powershell.exe -enc JABkAG8AdwBuAGwAbw...
        Parent: explorer.exe
        
14:23 - Event 5156: Conexión de red
        DestinationIP: 192.0.2.100
        DestinationPort: 8080
        Process: powershell.exe
        
14:24 - Event 7045: Servicio instalado
        ServiceName: WindowsDefenderService (¡nombre falso!)
        ImagePath: C:\ProgramData\windowsdefender.exe
        
14:25 - Event 5140: Acceso a recurso compartido de red
        ShareName: \\FINANCE-SERVER\Payroll
        UserName: marketing_user
        
14:26 - Event 4625: Intento fallido de RDP
        TargetServer: FINANCE-SERVER
        TargetUserName: admin
        SourceIP: 192.168.1.50 (interna—atacante moviéndose lateralmente)
```

---

### **Tu Análisis con MITRE:**

```
EVENTO → TÁCTICA MITRE → TÉCNICA MITRE → SEVERIDAD → ACCIÓN

14:22 PowerShell
└─ EJECUCIÓN (T04)
   └─ T1059.001 (PowerShell)
   └─ Severidad: ALTA (ejecución de código)
   └─ Acción: Investigar de dónde vino comando

14:23 Conexión de red
└─ COMANDO & CONTROL (T12)
   └─ T1071.001 (HTTP)
   └─ Severidad: CRÍTICA (malware comunicándose a casa)
   └─ Acción: Bloquear IP 192.0.2.100, revisar si malware corre

14:24 Servicio instalado
└─ PERSISTENCIA (T05)
   └─ T1543.003 (Servicio de Windows)
   └─ Severidad: CRÍTICA (atacante quedándose en sistema)
   └─ Acción: Eliminar servicio inmediatamente

14:25 Acceso a recurso compartido
└─ RECOLECCIÓN (T11)
   └─ T1005 (Datos del Sistema Local)
   └─ Severidad: CRÍTICA (apuntando a datos sensibles de nómina)
   └─ Acción: Revisar si datos fueron copiados

14:26 Intento fallido de RDP
└─ MOVIMIENTO LATERAL (T10)
   └─ T1021.001 (RDP)
   └─ Severidad: CRÍTICA (intentando propagarse)
   └─ Acción: Revisar logs FINANCE-SERVER por compromiso
```

---

## ❌ Errores Comunes Que Cometen los Estudiantes

### **Error 1: Memorizar Números de Técnica en Lugar de Nombres de Técnica**

**Incorrecto:**
> "Este evento mapea a T1059"

**Correcto:**
> "Esta ejecución de PowerShell mapea a T1059 (Intérprete de Comandos y Scripting), específicamente T1059.001 (PowerShell). El atacante intenta descargar un payload de segunda etapa."

**Por qué importa:** 
- Número de técnica es solo un ID de referencia
- Nombre de técnica te dice QUÉ sucedió
- Sub-técnica te dice la VARIANTE
- Descripción te dice POR QUÉ importa

---

### **Error 2: Pensar Que Cada Evento Mapea a Solo UNA Táctica**

**Incorrecto:**
```
Event 4688 PowerShell = Solo EJECUCIÓN
```

**Correcto:**
```
Event 4688 PowerShell -enc JAB...
├─ EJECUCIÓN (T1059.001) — se está ejecutando PowerShell
├─ EVASIÓN DE DEFENSA (T1027) — está codificado (ofuscado)
├─ MOVIMIENTO LATERAL (T1021) — si se ejecuta remotamente
└─ DESCUBRIMIENTO (T1087) — si ejecuta Get-ADUser

¡Un evento, múltiples tácticas!
```

**Por qué importa:**
- Pensamiento de táctica única = perdiendo cuadro completo de ataque
- Análisis de táctica múltiple = entiendes estrategia del atacante

---

### **Error 3: No Usar Procedimientos para Atribuir Ataques**

**Incorrecto:**
> "Este ataque usó PowerShell y fuerza bruta, así que podría ser cualquiera"

**Correcto:**
> "Este ataque usó:
> - T1059.001 con codificación Base64 (firma de APT-28)
> - T1550.003 Pass-the-Ticket (preferencia de APT-28)
> - Reconocimiento vía Get-ADUser (típico de APT-28)
> Esto coincide con procedimientos documentados de APT-28. Probabilidad: 80%"

**Por qué importa:**
- Conocer identidad del atacante ayuda predecir próximos movimientos
- Atribución informa nivel de amenaza y prioridad de respuesta
- Grupo de ransomware ≠ estado-nación ≠ hacker solitario (respuestas diferentes)

---

### **Error 4: Solo Ver Técnicas de "Ataque", Perdiendo Operaciones Normales**

**Incorrecto:**
> "Ejecución de PowerShell siempre es sospechosa"
> (Alerta en todos Event 4688 PowerShell)
> (Resultado: 10,000 falsos positivos por día)

**Correcto:**
> "Ejecución de PowerShell por admin de TI a las 9 AM = normal
> Ejecución de PowerShell por usuario de marketing a las 2 AM con comando codificado = sospechoso
> Ejecución de PowerShell con ofuscación (argumento -enc) = siempre sospechoso"

**Por qué importa:**
- Contexto importa (quién, cuándo, qué, dónde)
- No todo uso de técnica es malicioso
- Basándose en comportamiento normal primero, luego alertar en desviaciones

---

## 🧪 Escenario de Práctica: Mapea Este Ataque a MITRE

**Escenario:** Investigas una brecha que ocurrió durante 2 días.

**Línea de Tiempo:**

```
Día 1, 14:00 - Atacante envía email de phishing a 50 usuarios
Día 1, 14:15 - Un usuario hace click en link, malware se descarga
Día 1, 14:30 - Malware establece persistencia (instala servicio)
Día 1, 15:00 - Malware se conecta a servidor del atacante (C&C)
Día 1, 16:00 - Atacante desactiva Windows Defender
Día 1, 17:00 - Atacante crea cuenta backdoor
Día 1, 20:00 - Atacante dumpa credenciales de LSASS
Día 2, 08:00 - Atacante se mueve lateralmente a servidor de base de datos
Día 2, 09:00 - Atacante accede a base de datos financiera
Día 2, 10:00 - Atacante exporta 1 GB de datos
Día 2, 11:00 - Atacante borra Event Logs para esconder rastros
```

---

### **Tu Tarea:**

Mapea CADA evento a:
1. **Táctica MITRE** (número)
2. **Técnica MITRE** (ID + nombre)
3. **Severidad** (Baja/Media/Alta/Crítica)
4. **Punto de Detección** (Si usas logs de Windows, ¿cuál Event ID?)

**Formato de ejemplo:**

```
Día 1, 14:00 - Email de Phishing
├─ Táctica: #3 Acceso Inicial
├─ Técnica: T1566 (Phishing)
├─ Severidad: ALTA
└─ Detección: Email gateway logs (no Windows Event)
```

**¿Puedes intentar mapear el resto?** (Proporcionaré soluciones después)

---

## 🎯 Preguntas de Entrevista Que Podrías Recibir

### **Preguntas Fáciles (L1)**

**P: "Explica la diferencia entre una Táctica y Técnica MITRE."**

**Respuesta Esperada:**
> "Una Táctica es el objetivo estratégico (ej: Ejecución = ejecutar código), mientras que una Técnica es el método específico para lograr esa táctica (ej: T1059 PowerShell). Múltiples Técnicas pueden lograr la misma Táctica."

---

**P: "Nombra 3 técnicas usadas en Movimiento Lateral."**

**Respuesta Esperada:**
> "T1021.001 Servicios Remotos (RDP), T1550.002 Pass-the-Hash (usando hashes robados para autenticarse), y T1558.003 Kerberoasting (robando y quebrando tickets Kerberos)."

---

### **Preguntas Medias (L2)**

**P: "Ves Event 4688 mostrando ejecución de PowerShell con argumento -enc. Mapea esto a MITRE y explica por qué es sospechoso."**

**Respuesta Esperada:**
> "Esto mapea a:
> - Táctica: Ejecución (T04), Técnica: T1059.001 (PowerShell)
> - También Evasión de Defensa (T07), Técnica: T1027 (Ofuscación)
> 
> Es sospechoso porque:
> 1. Scripts PowerShell legítimos típicamente no están codificados en Base64
> 2. La codificación indica que atacante está escondiendo contenidos de comando
> 3. El flag -enc NUNCA es usado en administración normal
> 4. Este es patrón de ejecución de malware clásico"

---

**P: "Estás buscando ataques Pass-the-Hash. ¿Cuál es esta Técnica MITRE, qué Eventos buscarías, y qué indicaría éxito?"**

**Respuesta Esperada:**
> "Esta es T1550.002 (Usar Material de Autenticación Alterno - Pass-the-Hash), que es parte de Movimiento Lateral (T10).
>
> Lo que buscaría:
> - Event 4625: Intentos fallidos inusuales de logon con IPs extrañas
> - Event 4624: Logon exitoso sin script de logon correspondiente (uso offline)
> - Event 4688: Herramientas de dumping de credenciales (mimikatz, hashdump)
> - Event 5140: Acceso inusual a recurso compartido de red desde usuario de bajo-privilegio
>
> Indicadores de éxito:
> - Atacante accediendo a shares con hash de admin
> - Movimiento lateral a controlador de dominio
> - Escalada a cuenta Domain Admin"

---

### **Preguntas Difíciles (L3)**

**P: "Diseña una regla de detección para un ataque de ransomware usando MITRE. Comienza desde Acceso Inicial, encadena hasta Impacto, y explica qué IOCs alertarías y qué falsos positivos esperarías."**

**Respuesta Esperada:**
> "Estrategia de detección de ransomware usando cadena MITRE:
>
> Etapa 1: ACCESO INICIAL (T1566 - Phishing)
> - Alerta: Email con adjunto .exe o macro sospechoso
> - IOC: Dominio malicioso, hash de archivo sospechoso
> - Falso Positivo: Emails comerciales legítimos con PDFs
> - Mitigación: Escaneo de puerta de email, capacitación de usuarios
>
> Etapa 2: EJECUCIÓN (T1059.003 - cmd.exe, T1204 - Ejecución de Usuario)
> - Alerta: Usuario lanza .exe de adjunto
> - IOC: Ejecutable desconocido en directorio temp
> - Falso Positivo: Instaladores de software legítimo
> - Mitigación: Sandboxing, análisis de comportamiento
>
> Etapa 3: PERSISTENCIA (T1547 - Autostart, T1543 - Servicio)
> - Alerta: Nuevo servicio instalado con nombre inusual
> - IOC: Servicio apuntando a directorio temp o AppData
> - Falso Positivo: Actualizaciones de software legítimas
> - Mitigación: Whitelist de servicios conocidos, alerta en nuevos
>
> Etapa 4: EVASIÓN DE DEFENSA (T1562 - Desactivar Defensas)
> - Alerta: Windows Defender desactivado
> - IOC: Event 1102 (borrado de log), modificación de Registry
> - Falso Positivo: Gestión intencional de admin
> - Mitigación: Alerta en cambios fuera de horario, requiere aprobación
>
> Etapa 5: DESCUBRIMIENTO (T1087 - Descubrimiento de Cuenta)
> - Alerta: Comandos Get-ADUser, net user
> - IOC: Comandos de reconocimiento PowerShell
> - Falso Positivo: Scripts de admin legítimos
> - Mitigación: Basándose en actividad normal de admin, alertar en anomalías
>
> Etapa 6: MOVIMIENTO LATERAL (T1021.001 - RDP)
> - Alerta: Conexiones RDP inusuales entre máquinas
> - IOC: RDP lateral a servidores de archivo de endpoints
> - Falso Positivo: Administración TI legítima
> - Mitigación: Restringir RDP con EDR, monitorear patrones inusuales
>
> Etapa 7: RECOLECCIÓN (T1005 - Recolección de Datos Locales)
> - Alerta: Acceso súbito a archivos en recursos compartidos
> - IOC: Gran volumen de lecturas de recursos compartidos financieros/RRHH
> - Falso Positivo: Procesos de backup legítimos
> - Mitigación: Monitorear acceso fuera de horario, basándose en acceso a archivo
>
> Etapa 8: IMPACTO (T1486 - Encriptar Datos Sensibles)
> - Alerta: Encriptación masiva de archivo, extensiones de archivo cambiando
> - IOC: Proceso de encriptación desconocido, creación de nota de rescate
> - Falso Positivo: Operaciones de backup legítimas
> - Mitigación: Monitoreo de integridad de archivo, planes de recuperación
>
> Este enfoque de capas atrapa ransomware en MÚLTIPLES etapas en lugar de confiar en una firma única."

---

## 🔗 Cómo Esto Se Conecta a Todo Lo Demás

### **MITRE → Windows (Event Logs)**
- Técnicas MITRE → Event IDs de Windows
- T1059 (PowerShell) → Event 4688
- T1543 (Servicio) → Event 7045
- T1070 (Borrado de log) → Event 1102
- Domina Windows logs Y MITRE, tienes cobertura de detección completa

### **MITRE → Active Directory**
- T1087 (Descubrimiento de Cuenta) → Reconocimiento de Get-ADUser
- T1550.003 (Pass-the-Ticket) → Ataque Kerberos
- T1021.001 (RDP) → Movimiento lateral en dominio
- T1134 (Manipulación de Token) → Robo de credenciales en AD
- AD es donde suceden los ataques más valiosos—MITRE te ayuda a defenderlo

### **MITRE → SIEM**
- Construye reglas de detección usando técnicas MITRE
- Correlaciona múltiples eventos usando tácticas MITRE
- Alerta cuando cadena de ataque progresa a través de tácticas
- SIEM SIN MITRE = perdiendo bosque por los árboles

### **MITRE → Inteligencia de Amenazas**
- Mapea actores de amenaza a técnicas que usan
- Construye perfiles: "APT-28 siempre usa X, Y, Z técnicas"
- Predice próximos movimientos basados en procedimientos de grupo
- Prioriza hunting basado en amenazas conocidas

### **MITRE → Respuesta a Incidentes**
- Usa MITRE para alcance de brecha: "¿Qué tan lejos llegó atacante? ¿Acceso Inicial o Exfiltración?"
- Usa MITRE para remediación: "Para cada táctica, ¿qué necesitamos arreglar?"
- Usa MITRE para comunicar: "Los stakeholders del negocio entienden 'llegaron a Movimiento Lateral' mejor que 'T1021'"

---

## 💾 TL;DR Para Gente Ocupada

### **Las 14 Tácticas (Referencia Rápida)**

| # | Táctica | Objetivo | Técnica Principal | Evento Windows |
|---|---------|----------|------------------|----------------|
| 1 | Reconocimiento | Reunir info | OSINT | Ninguno |
| 2 | Desarrollo Recursos | Construir herramientas | Registro de dominio | Firewall |
| 3 | Acceso Inicial | Entrar sistema | Phishing (T1566) | 4688 |
| 4 | Ejecución | Ejecutar código | PowerShell (T1059) | 4688 |
| 5 | Persistencia | Quedarse en sistema | Servicio (T1543) | 7045, 4698 |
| 6 | Escal Privilegios | Obtener admin | UAC bypass (T1548) | 4688 |
| 7 | Evasión Defensa | Esconder actividad | Ofuscación (T1027) | 1102 |
| 8 | Acceso Credenciales | Robar llaves | Fuerza Bruta (T1110) | 4625 |
| 9 | Descubrimiento | Mapear red | Enumerar cuenta (T1087) | 4688 |
| 10 | Mov Lateral | Moverse alrededor | RDP (T1021) | 4625, 5140 |
| 11 | Recolección | Reunir datos | Acceso archivo (T1005) | 5140 |
| 12 | C&Control | Controlar malware | HTTP (T1071) | 5156 |
| 13 | Exfiltración | Robar afuera | Sobre C2 (T1041) | Firewall |
| 14 | Impacto | Dañar | Ransomware (T1486) | 4688 |

### **Lo Que Te Hace Peligroso:**

1. **Conoces las 14 tácticas en orden** ✅
2. **Puedes mapear eventos de Windows a MITRE** ✅
3. **Entiendes que un evento = múltiples tácticas** ✅
4. **Conoces los top 30 técnicas** ✅
5. **Puedes atribuir ataques a grupos APT** ✅

---

## 📌 Realidad de Producción: Cómo Funciona Realmente

### **SOC Real: Construyendo Detección Basada en MITRE**

**Escenario:** Tu SOC necesita detectar ransomware antes de encriptación.

**Enfoque Basado en MITRE:**

```
Alerta en: Acceso Inicial (T1566 - Phishing)
├─ Regla: Email con adjunto .exe o macro
├─ Acción: Bloquear adjunto, poner email en cuarentena

Alerta en: Ejecución (T1204 - Ejecución de Usuario)
├─ Regla: Usuario lanza .exe de adjunto
├─ Acción: Matar proceso, alerta analista

Alerta en: Persistencia (T1547 - Autostart)
├─ Regla: Modificación de Registry agregando startup
├─ Acción: Bloquear persistencia, alerta

Alerta en: Evasión de Defensa (T1562 - Desactivar Defensas)
├─ Regla: PowerShell desactiva Defender
├─ Acción: ESCALACIÓN INMEDIATA

Alerta en: Movimiento Lateral (T1021 - RDP)
├─ Regla: RDP de estación de trabajo a estación de trabajo
├─ Acción: Bloquear conexión, aislar máquinas

Alerta en: Recolección (T1005 - Datos Locales)
├─ Regla: Acceso grande de archivo de endpoint
├─ Acción: Alerta, investigar datos siendo robados

Alerta en: Impacto (T1486 - Encriptación)
├─ Regla: Modificación masiva de archivo, extensión .encrypted
├─ Acción: CONTENCIÓN - aislar máquina inmediatamente
```

**Resultado:**
- Para en Etapa 1 → No hay brecha
- Para en Etapa 3 → Compromiso menor
- Para en Etapa 7 → Contenido antes de daño
- Pierde todas las etapas → Éxito de ransomware

---

### **Línea de Tiempo Real de Brecha (Por Qué Importa MITRE)**

**Incidente de ransomware, línea de tiempo real:**

```
Lunes 14:00 - Email llega (Reconocimiento completo)
Lunes 14:05 - Usuario hace click en link (Acceso Inicial completo)
Lunes 14:10 - Malware se ejecuta (Ejecución completa)
Lunes 14:15 - Servicio instalado (Persistencia completa) ← DEBERÍA ALERTAR AQUÍ
Lunes 15:00 - Defender desactivado (Evasión completa) ← VENTANA CERRÁNDOSE
Lunes 16:00 - Credenciales robadas (Acceso a Credenciales completo) ← PUNTO DE NO RETORNO
Lunes 17:00 - Acceso a servidor de BD (Movimiento Lateral completo) ← ¿CONTENIDO?
Martes 08:00 - Datos copiados (Recolección completa) ← DEMASIADO TARDE
Martes 09:00 - Datos exfiltrados (Exfiltración completa) ← JUEGO TERMINADO
Martes 10:00 - Archivos encriptados (Impacto completo) ← PÉRDIDA TOTAL

Post-incidente: Logs borrados (Evasión continuada) ← Encubrimiento
```

**Si SOC tuviera detección basada en MITRE:**
- Alerta en Lunes 14:15 (Persistencia)
- Investigación inmediata
- Brecha contenida en 1 hora

**Si SOC no tuviera:**
- Sin alertas hasta Martes 10:00 (Impacto)
- Brecha ya completa
- 24+ horas de daño

---

## 📚 Lectura Adicional

### **Recursos Oficiales**
- [Framework MITRE ATT&CK](https://attack.mitre.org) - Sitio oficial, márcalo
- [Grupos MITRE](https://attack.mitre.org/groups/) - Grupos APT reales con procedimientos
- [Software MITRE](https://attack.mitre.org/software/) - Malware mapeado a técnicas

### **Recursos de Aprendizaje**
- MITRE ATT&CK para Detección de Amenazas (canal YouTube)
- SpecterOps: From ATT&CK to Defense (write-ups)
- ThreatHunting.net: Hunts basados en MITRE
- DetectLabs: Ejercicios MITRE basados en escenarios

### **Libros**
- *The Art of Memory Forensics* - Mapea técnicas a artefactos de memoria
- *Incident Response & Computer Forensics* - Mapea MITRE a investigación
- *Operator Handbook* - Perspectiva del atacante (entiende procedimientos)

### **Herramientas**
- **Mitre ATT&CK Navigator** - Visualiza técnicas por grupo/detección
- **Cyber Kill Chain Mapper** - Mapea técnicas a controles defensivos
- **ThreatHunting Workbench** - Encuentra brechas en detección usando MITRE

---

## 🎓 Pensamiento Final

MITRE ATT&CK es tu **Piedra de Rosetta para ciberseguridad**. Traduce:

- Logs crudos → Tácticas de ataque
- Eventos aislados → Cadenas de ataque  
- Malware desconocido → Grupos APT conocidos
- Alertas aleatorias → Comprensión estratégica

Domina MITRE, y pasas de "analizar eventos" a "entender ataques"—que es la diferencia entre analista junior y threat hunter senior.

**Tú puedes hacerlo.** 🎯

---

**Versión del Documento:** 1.0  
**Última Actualización:** Enero 2024  
**Creado para:** Capacitación de Seguridad Blue Team  
**Tema Siguiente:** Seguridad de Active Directory
