# 🔐 SIEM: Guía Completa para Blue Team
## Security Information and Event Management

---

## 📖 ¿Qué es SIEM?

**En 30 segundos:**
Un SIEM es la **herramienta central que recolecta, normaliza, correlaciona y analiza logs** de todas tus fuentes de seguridad (firewalls, antivirus, AD, Windows, etc.) para **detectar alertas, patrones sospechosos y potenciales incidentes de seguridad**.

**No es solo visualización.** Es:
- ✅ **Agregación**: Centraliza logs de cientos de fuentes
- ✅ **Normalización**: Traduce diferentes formatos a uno estándar
- ✅ **Correlación**: Relaciona eventos de múltiples fuentes
- ✅ **Detección**: Aplica reglas para generar alertas
- ✅ **Respuesta**: Proporciona contexto para investigación

---

## 🎯 ¿Por Qué Un SOC/Blue Team Necesita SIEM?

### **En La Realidad del Trabajo**

| Escenario | Sin SIEM | Con SIEM |
|-----------|----------|----------|
| **Un atacante intenta login fallido en AD, luego accede a servidor bloqueado por firewall** | Ves 2 eventos separados en 2 herramientas diferentes | SIEM correlaciona: "¿Misma IP en ambos? → Alerta Critical" |
| **Compliance (GDPR, PCI-DSS)** | ¿Dónde están todos los logs? ¿Retención? ¿Auditoría? | SIEM centraliza, retiene y reporta automáticamente |
| **Investigación de incidente** | Saltar entre 5 herramientas diferentes | Toda la evidencia en un lugar, con contexto |
| **Detección proactiva** | Solo ves lo que cada herramienta alerta | SIEM detecta patrones que ninguna herramienta individual vería |

### **Preguntas de Entrevista que Te Harán**

1. **"¿Por qué es importante centralizar logs?"**
   - Respuesta esperada: Visibilidad única, detección de correlaciones, compliance, investigación más rápida

2. **"¿Cuál es la diferencia entre un firewall que genera alertas y un SIEM?"**
   - Respuesta esperada: Firewall solo ve su tráfico; SIEM ve el contexto completo (AD login + firewall block + antivirus threat = ataque coordinado)

3. **"¿Qué pasa si un log no llega al SIEM?"**
   - Respuesta esperada: Ese evento no se detecta, no se correlaciona, no hay auditoría. Es un punto ciego crítico.

---

## 🔍 El Concepto Desglosado

### **Part 1: Logs vs Alertas (La Diferencia Fundamental)**

```
📝 LOG = Evento sin procesar que sucedió
   Ejemplo: "user_login from 192.168.1.100 at 14:30:45"

🚨 ALERT = Log procesado por regla que indica peligro
   Ejemplo: "CRITICAL: Failed login attempts detected (50x in 5 min from same IP)"
```

**Clave:** No todos los logs son alertas. Un log es "información". Una alerta es "algo requiere atención".

### **Part 2: El Flujo Real en Tu SOC**

```
┌──────────────────────────────────────────────────────────────┐
│                    TUS FUENTES DE LOGS                       │
├─────────────────┬─────────────────┬─────────────────┬────────┤
│   FIREWALL      │   ANTIVIRUS     │   ACTIVE DIR    │ WINDOWS│
│  (logs)         │   (logs)        │   (logs)        │ (logs) │
└────────┬────────┴────────┬────────┴────────┬────────┴────┬───┘
         │                 │                 │             │
         └─────────────────┴─────────────────┴─────────────┘
                           │
                    (Protocolo: Syslog)
                           │
                    ┌──────▼──────┐
                    │    SIEM     │
                    ├─────────────┤
                    │ 1. RECIBE   │ Centraliza todos los logs
                    │ 2. NORMALIZA│ Traduce a formato estándar
                    │ 3. ENRIQUECE│ Añade contexto (GeoIP, threat intel)
                    │ 4. CORRELACIONA│ Relaciona eventos de múltiples fuentes
                    │ 5. APLICA REGLAS│ Detecta patrones sospechosos
                    │ 6. GENERA ALERTAS│ Crea incidentes
                    └──────┬──────┘
                           │
                    ┌──────▼──────────────────┐
                    │   DASHBOARD DEL SOC     │
                    │  ✅ Alertas por severidad│
                    │  ✅ Timeline de eventos │
                    │  ✅ Correlaciones      │
                    │  ✅ Contexto completo  │
                    └────────────────────────┘
```

### **Part 3: Las Reglas - El Corazón del SIEM**

Las reglas son el **motor de detección** del SIEM. Sin reglas, SIEM solo acumula logs.

**¿De dónde vienen las reglas?**
- ✅ **Threat Hunters**: Descubren patrones nuevos de ataque
- ✅ **Incident Response**: "Esto pasó, cómo detectarlo en el futuro"
- ✅ **Playbooks**: Procedimientos documentados se convierten en reglas
- ✅ **Estándares**: MITRE ATT&CK, CIS Controls, etc.
- ✅ **Threat Intelligence**: "Esta IP es maliciosa, bloquea si aparece"

**¿Quién las implementa?**
- **Detection Engineers** o **Security Engineers** escriben la regla en Sigma/KQL/SPL
- **SOC Analysts** validan que funcione
- **SIEM Admins** las despliegan en producción

---

## ⚙️ Cómo Escribir Una Regla SIEM (Sigma)

### **¿Por Qué Sigma?**

Sigma es un **estándar abierto** para escribir reglas de detección que funciona en cualquier SIEM:
- ✅ Splunk, ELK, Microsoft Sentinel, Sumo Logic, etc.
- ✅ Portable entre herramientas
- ✅ Fácil de entender y colaborar
- ✅ Usado por la industria

### **Estructura Básica de una Regla Sigma**

```yaml
title: Detección de Múltiples Intentos de Login Fallidos
id: c91f8a6a-5c45-11eb-ae93-0242ac120002
status: experimental
description: Detecta 5+ intentos de login fallidos en 5 minutos (posible ataque de fuerza bruta)
author: Blue Team SOC
date: 2024-01-15

# PARTE 1: FUENTE DE LOGS
logsource:
  product: windows
  service: security
  
# PARTE 2: DETECCIÓN (¿Qué buscamos?)
detection:
  selection:
    EventID: 4625  # Windows Event ID para login fallido
    LogonType: 3   # Tipo: Red (no local)
    Status: 'C0000064'  # Error: Usuario desconocido/contraseña mala
  
  # PARTE 3: TIMEFRAME (¿En cuánto tiempo?)
  timeframe: 5m
  
  # PARTE 4: CONDICIÓN
  condition: selection | count(TargetUserName) by SourceIp >= 5

# PARTE 5: CONTEXTO
falsepositives:
  - Usuarios legítimos olvidando contraseñas
  - Procesos automatizados con credenciales vencidas
  
level: medium
tags:
  - attack.credential_access  # MITRE ATT&CK
  - attack.t1110_001  # Brute Force: Password Guessing
```

### **Desglose Línea por Línea**

| Sección | ¿Qué es? | Tu SOC |
|---------|---------|--------|
| **title** | Nombre descriptivo de la regla | "Detección de Brute Force en AD" |
| **id** | UUID único | Para tracking y versioning |
| **logsource** | ¿De dónde vienen los logs? | product: windows, service: security |
| **selection** | Qué logs coinciden con nuestra búsqueda | EventID: 4625 (login fallido) |
| **timeframe** | Ventana de tiempo para correlación | 5m (5 minutos) |
| **condition** | Lógica: cuándo generar alerta | >= 5 intentos = Alerta |
| **level** | Severidad (low/medium/high/critical) | medium |
| **tags** | MITRE ATT&CK techniques | attack.t1110_001 |

### **Ejemplo Real #1: Detección de Acceso a Firewall desde IP Bloqueada**

```yaml
title: Firewall Block + Posterior Access Attempt
id: fw-detect-001
logsource:
  product: firewall
  service: fortinet

detection:
  blocked_traffic:
    EventType: BLOCKED
    DestinationIP: 10.0.0.0/8  # Red interna
    
  suspicious_source:
    SourceIP|filter: threat_intel_db  # IP en base de datos maliciosa
    
  timeframe: 10m
  condition: blocked_traffic AND suspicious_source

level: high
tags:
  - attack.reconnaissance
  - attack.t1592  # Gather Victim Host Information
```

### **Ejemplo Real #2: Detección en Active Directory**

```yaml
title: Múltiples Cambios de Contraseña - Posible Compromiso
id: ad-detect-002
logsource:
  product: windows
  service: security

detection:
  password_change:
    EventID: 4723  # Password change request
    
  admin_user:
    TargetUserName|contains:
      - 'admin'
      - 'domain admin'
      - 'service account'
  
  timeframe: 1h
  condition: password_change AND admin_user

level: critical
tags:
  - attack.persistence
  - attack.t1098  # Account Manipulation
```

### **Ejemplo Real #3: Correlación Antivirus + Windows**

```yaml
title: Antivirus Threat + Subsequent Windows Process Creation
id: av-detect-003
logsource:
  product: antivirus,windows
  service: defense

detection:
  av_threat:
    ThreatDetected: true
    ThreatLevel: high
    
  suspicious_process:
    EventID: 4688  # Process creation
    CommandLine|contains:
      - 'powershell'
      - 'cmd.exe'
      - 'whoami'
  
  timeframe: 5m
  condition: av_threat AND suspicious_process by SourceIP

level: critical
tags:
  - attack.execution
  - attack.t1059  # Command and Scripting Interpreter
```

---

## 🔗 Mapeo a MITRE ATT&CK (Para Reportes)

### **¿Por Qué Mapear?**

MITRE ATT&CK es un **estándar de referencia** que:
- ✅ **Comunica** qué tipo de ataque detectaste (a nivel ejecutivo/compliance)
- ✅ **Categoriza** técnicas de ataque en tu reporte
- ✅ **Permite comparación** con otros SOCs
- ✅ **Facilita compliance**: "¿Detectamos T1110?" → Sí, en 5 ocasiones

### **Estructura MITRE ATT&CK**

```
attack.tactic → attack.technique → attack.sub_technique

Ejemplo:
attack.credential_access → attack.t1110 → attack.t1110_001
           (Tática)               (Técnica)      (Subtécnica)
           ¿Para qué?        ¿Cómo lo hacen? ¿De qué forma exacta?
```

### **Los Tácticos Principales (Lo que Necesitas Saber)**

| Táctico | Descripción | Ejemplos |
|---------|------------|----------|
| **reconnaissance** | Recopilación de información antes del ataque | Escaneo de puertos, enumerar usuarios |
| **resource_development** | Preparación de infraestructura de ataque | Comprar dominios fake, crear C2 |
| **initial_access** | Primer punto de entrada | Phishing, exploit, credenciales comprometidas |
| **execution** | Ejecutar código malicioso | PowerShell, Command, Macros |
| **persistence** | Mantener acceso duradero | Crear cuentas, instalar backdoors |
| **privilege_escalation** | Obtener permisos mayores | UAC bypass, exploit kernel |
| **defense_evasion** | Evitar ser detectado | Obfuscación, desactivar antivirus |
| **credential_access** | Robar credenciales | Brute force, keylogger, dumping |
| **discovery** | Recopilar info del sistema/red | Enumerar shares, listar procesos |
| **lateral_movement** | Moverse en la red | Pass-the-hash, RDP, compartir archivos |
| **collection** | Recopilar datos de interés | Captura de pantalla, email scraping |
| **command_and_control** | Comunicar con atacante | C2 channels, DNS tunneling |
| **exfiltration** | Robar datos | Upload a servidor externo, email |
| **impact** | Causar daño | Borrar datos, ransomware, DDoS |

### **Cómo Mapear Tu Regla a MITRE**

**Paso 1: Identifica QUÉ está detectando tu regla**
```
Mi regla detecta: "5+ login fallidos en 5 min desde misma IP"
¿Qué es esto? = Intento de adivinar contraseña
```

**Paso 2: Busca la técnica en MITRE**
```
Voy a https://attack.mitre.org
Busco: "Brute Force"
Encuentro: T1110 - Brute Force
  └─ T1110_001 - Password Guessing
  └─ T1110_002 - Password Spraying
  └─ T1110_003 - Password Cracking
  └─ T1110_004 - Credential Stuffing
```

**Paso 3: Elige la subtécnica correcta**
```
¿Mi regla detecta intentos múltiples de MISMA contraseña a DIFERENTES usuarios?
→ T1110_002 (Password Spraying)

¿Mi regla detecta intentos múltiples de DIFERENTES contraseñas a MISMO usuario?
→ T1110_001 (Password Guessing)
```

**Paso 4: Añade a tu regla**
```yaml
tags:
  - attack.credential_access       # Táctico
  - attack.t1110                   # Técnica
  - attack.t1110_001              # Subtécnica
```

### **Mapeo Rápido: Ejemplos de Tu SOC**

| Lo que detectas | Táctico | Técnica | Subtécnica |
|-----------------|---------|---------|------------|
| Firewall bloquea conexión a IP maliciosa | reconnaissance | T1592 | Gather Victim Host Info |
| Múltiples intentos de login fallidos | credential_access | T1110 | T1110_001 (Password Guessing) |
| PowerShell ejecuta comando sospechoso | execution | T1059 | T1059_001 (PowerShell) |
| Usuario admin cambia contraseña a las 3 AM | persistence | T1098 | T1098_002 (Account Manipulation) |
| Archivo ejecutable descargado desde email | initial_access | T1566 | T1566_001 (Phishing - Attachment) |
| Servicio nuevo creado en Windows | persistence | T1543 | T1543_003 (Windows Service) |
| Usuario accede a share desde IP externa | lateral_movement | T1570 | Lateral Tool Transfer |
| Volcado de credenciales de SAM | credential_access | T1003 | T1003_002 (LSASS Memory) |

### **Cómo Reportar (Ejemplo)**

```
INCIDENTE #2024-001215
Severidad: HIGH
Tipo: Credential Access Attack

Descripción:
Se detectaron 47 intentos de login fallidos en 3 minutos contra usuario admin@corp.com
desde IP 203.0.113.45 (localización: China según GeoIP)

MITRE ATT&CK Mapping:
├─ Táctico: credential_access
├─ Técnica: T1110 - Brute Force
└─ Subtécnica: T1110_001 - Password Guessing

Fuentes de Detección:
✅ Regla: "Multiple Failed Logins" (SIEM)
✅ EventID: 4625 (Windows Security Log)
✅ Fuente: Active Directory

Acciones Tomadas:
1. IP bloqueada a nivel firewall
2. Cuenta monitoreada para cambios
3. Password reset para usuario afectado
4. Escalado a IR Team
```

---

## 🚨 Tu Metodología: Respuesta a Alertas

### **El Proceso: Evaluar → Investigar → Decidir**

```
┌─────────────────────────────────────────────────────────┐
│          ALERTA LLEGA AL SOC (Dashboard)               │
├─────────────────────────────────────────────────────────┤
│ PASO 1: EVALUAR                                         │
│  └─ Severidad: LOW / MEDIUM / HIGH / CRITICAL          │
│  └─ Contexto: ¿Es usuario legítimo? ¿Horario normal?  │
│  └─ Fuente: ¿De dónde viene? ¿Es confiable?           │
│  └─ Tiempo: ¿Cuándo sucedió?                           │
├─────────────────────────────────────────────────────────┤
│ PASO 2: INVESTIGAR (Si requiere atención)              │
│  └─ ¿Hay eventos previos similares?                    │
│  └─ ¿Otros sistemas lo confirman?                      │
│  └─ ¿Existe explicación legítima?                      │
├─────────────────────────────────────────────────────────┤
│ PASO 3: DECIDIR                                         │
│  ├─ ✅ INVESTIGAR: Tier 1/2 análisis + documentación  │
│  ├─ ⬆️ ESCALAR: Tier 3 / Incident Response Team      │
│  └─ ✋ CERRAR: False Positive (documentar por qué)     │
├─────────────────────────────────────────────────────────┤
│ PASO 4: ACTUAR                                          │
│  └─ Recopilar evidencia, contener, eradicar, verificar │
│  └─ Documentar todo en caso de auditoría               │
└─────────────────────────────────────────────────────────┘
```

### **Severidad: ¿Cuándo Investigar vs Cerrar?**

| Severidad | Ejemplos | Acción |
|-----------|----------|--------|
| **LOW** | 1 login fallido, acceso a share permitido, puerto cerrado | Revisar en reporte semanal, probablemente cerrar |
| **MEDIUM** | 5+ login fallidos en 10 min, cambio de permisos, ejecutable bloqueado | Investigar en 2-4 horas, correlacionar con eventos |
| **HIGH** | Múltiples login fallidos en 2 min + posterior acceso exitoso, descarga de credential dumper | Investigar AHORA, escalar si confirmas |
| **CRITICAL** | Admin modificado, servicio malicioso creado, ransomware detectado, volcado de AD | Escalar INMEDIATAMENTE a IR, iniciar respuesta |

### **Ejemplo: Investigando una Alerta MEDIUM**

```
🚨 ALERTA: "Suspicious PowerShell Execution"
Severity: MEDIUM
Source: Windows Event ID 4688
Time: 14:32 UTC
User: jsmith@corp.com
Command: powershell.exe -Enc JABzAGgAZQBs...

PASO 1: EVALUAR
✓ Severidad: MEDIUM (PowerShell con encoding puede ser legítimo o malicioso)
✓ Usuario: jsmith - es developer conocido (REDUCIR RIESGO)
✓ Hora: 14:32 en horario laboral (LEGÍTIMO)
✓ Fuente: Workstation, no servidor (REDUCIR RIESGO)

PASO 2: INVESTIGAR
? ¿Hay events previos de este usuario ejecutando PowerShell? 
  → Revisar últimas 7 días: SÍ, hace 2 semanas ejecutó similar (LEGÍTIMO)
  
? ¿Otros sistemas ven actividad sospechosa desde este usuario?
  → Revisar firewall: NO anomalías
  → Revisar antivirus: NO amenazas
  
? ¿Qué hace ese comando en base64?
  → Decodificar: "Get-Process | Export-CSV report.csv" (LEGÍTIMO - reporte)

PASO 3: DECIDIR
✅ CERRAR - False Positive
Razón: Usuario documentado, actividad histórica similar, comando inofensivo

Documentación:
- Usuario es developer, ejecuta scripts regularmente
- Comando es para generar reporte de procesos
- No hay actividad sospechosa correlacionada
- Regla puede necesitar tuning para excluir developers conocidos
```

---

## ❌ Errores Comunes que Cometen Los Analistas

### **Error #1: Confundir "Alerta" con "Amenaza"**

```
❌ MAL:
"Tenemos 200 alertas hoy, debe ser ataque importante"

✅ BIEN:
"Tenemos 200 alertas, pero 195 son false positives (usuarios olvidando contraseñas).
Solo 5 requieren investigación. De esas, 3 son legítimas, 2 son potenciales threats."

Lección: Una alerta ≠ Ataque. Necesitas contexto.
```

### **Error #2: No Hacer Baseline de Actividad Normal**

```
❌ MAL:
"Usuario descargó 500 MB, ¡ALERTA!"
Realidad: Es ingeniero de datos, descarga 500 MB DIARIOS.

✅ BIEN:
Baseline: Usuario normalmente descarga 400-600 MB/día
Anomalía: Si descarga >2GB O en horario no laboral, ENTONCES alerta

Lección: Necesitas saber qué es "normal" para ese usuario/sistema.
```

### **Error #3: Reglas Demasiado Amplias (Ruido)**

```
❌ MAL:
Regla: "Cualquiera que ejecute PowerShell = Alerta"
Resultado: 1000+ alertas/día de usuarios legítimos
Analistas abrumados, empiezan a ignorar alertas

✅ BIEN:
Regla: "Usuario ejecuta PowerShell CON parámetro -enc (encoding) 
        Y la IP NO está en whitelist de developers"
Resultado: 5-10 alertas/día, mayoría legítimas pero investigables

Lección: Especificidad > Amplitud
```

### **Error #4: No Documentar False Positives**

```
❌ MAL:
Mismo analyst, mismo false positive, 10 veces al mes
Siempre cierra sin documentar razón

✅ BIEN:
Primer false positive: "Usuario X descargó PDF desde email"
Documentas: "Vendor ABC envía PDFs automáticos, false positive conocido"
Después: Añades a whitelist o tunas la regla
Resultado: Menos ruido, menos investigación innecesaria

Lección: Los FP son oportunidades para mejorar reglas.
```

---

## 🧪 Ejercicios Prácticos

### **Ejercicio #1: Escribir tu Primera Regla**

```
Escenario: En tu firewall ves que alguien intenta conectar a puerto 22 (SSH)
desde una IP externa a tu servidor de base de datos interno (no debe haber SSH externo).

Tarea: Escribe una regla Sigma para detectar esto.

Respuesta esperada:
- logsource: firewall
- selection: DestinationPort=22 AND DestinationIP=internal_db AND SourceIP=external
- condition: selection
- level: high (esto es suspicious, database no debe ser accessible por SSH desde internet)
- tags: attack.lateral_movement, attack.t1570
```

### **Ejercicio #2: Clasificar Eventos**

```
Evento 1: Admin hace login a las 3 AM desde IP en China
¿Qué severidad? ¿Qué acción?
→ CRITICAL/HIGH - Investigar AHORA
   ¿Es usuario en viaje? ¿Están de guardia? Contexto es clave.

Evento 2: Developer ejecuta PowerShell en workstation durante horario laboral
¿Qué severidad? ¿Qué acción?
→ LOW/MEDIUM - Revisar contexto
   ¿Está documentado? ¿Otros developers lo hacen? Probablemente cerrar.

Evento 3: 3 intentos de login fallidos en 30 minutos (usuario normal)
¿Qué severidad? ¿Qué acción?
→ LOW - Cerrar
   Dentro de lo normal, usuario olvidó contraseña probablemente.
```

### **Ejercicio #3: Mapear a MITRE**

```
Escenario: Tu SIEM detecta que un usuario cambió contraseña de su cuenta
y luego accedió con la nueva contraseña desde una IP nueva (nunca vista).

Tarea: Mapea esto a MITRE ATT&CK

Respuesta esperada:
- Acción: Cambio de contraseña + acceso desde IP nueva
- ¿Qué técnica? → T1098 (Account Manipulation) o T1078 (Valid Accounts)
- ¿Por qué? → Atacante compromete cuenta, cambia contraseña, mantiene acceso
- Reporte: "Detección de Account Persistence (T1098) en usuario jsmith"
```

---

## 🎯 Preguntas de Entrevista Que Te Harán

### **Level 1 (Entry-Level SOC Analyst)**

**P1: "¿Qué es SIEM?"**
```
Respuesta esperada:
"Herramienta que centraliza logs de múltiples fuentes, los normaliza y 
genera alertas basadas en reglas para detectar incidentes de seguridad."
```

**P2: "¿Cuál es la diferencia entre un log y una alerta?"**
```
Respuesta esperada:
"Un log es un evento que sucedió (información cruda).
Una alerta es cuando una regla SIEM detecta que ese log (o combinación de logs)
representa un comportamiento sospechoso o malicioso."
```

**P3: "¿Por qué es importante centralizar logs?"**
```
Respuesta esperada:
"Porque permite detectar patrones que ninguna herramienta individual vería.
Ejemplo: Login fallido (AD) + acceso a servidor bloqueado (firewall) = 
ataque coordinado que SIEM correlaciona."
```

### **Level 2 (Mid-Level SOC Analyst / Detection Engineer)**

**P1: "¿Cómo escribirías una regla para detectar brute force en Active Directory?"**
```
Respuesta esperada:
"Buscar EventID 4625 (login fallido) con mismo usuario pero múltiples IPs,
o misma IP con múltiples usuarios, en timeframe corto (5 min).
Si >= 5 intentos fallidos, generar alerta."

Bonus: "Mapearía a MITRE T1110 (Brute Force)"
```

**P2: "¿Cómo manejarías un false positive en tu SIEM?"**
```
Respuesta esperada:
"Primero, documentaría por qué es falso positivo (contexto legítimo).
Luego, consideraría:
- ¿Excluir este user/IP de la regla?
- ¿Ajustar threshold (5 intentos → 10 intentos)?
- ¿Añadir filtro (excluir developers conocidos)?
- Después, mediría si mejora sin perder detecciones reales."
```

**P3: "¿Cómo correlacionarías eventos de múltiples fuentes en SIEM?"**
```
Respuesta esperada:
"Usando campos comunes como:
- SourceIP: Si misma IP aparece en firewall + antivirus = sospechoso
- TargetUser: Si mismo usuario en AD + Windows = correlación
- Timestamp: Si eventos suceden en ventana de tiempo corta

Ejemplo: Login fallido (AD) + archivo malicioso (antivirus) + conexión bloqueada (firewall)
en menos de 5 minutos, misma IP → SIEM genera alerta correlacionada = HIGH"
```

### **Level 3 (Senior Analyst / Detection Engineer)**

**P1: "¿Cómo diseñarías un programa de detección desde cero?"**
```
Respuesta esperada:
"Seguiría este proceso:

1. MAPEAR AMENAZAS: ¿Cuál es el adversario? ¿Qué tácticas usa?
2. USAR MITRE ATT&CK: Documentar técnicas específicas que queremos detectar
3. CREAR REGLAS: Por cada técnica, escribir regla en Sigma
4. VALIDAR: Probar con logs históricos (¿detecta ataques conocidos?)
5. TUNING: Ajustar para reducir false positives
6. MEDIR: Trackear True Positives, False Positives, False Negatives
7. MEJORAR: Iterar basado en análisis de incidentes reales
"
```

**P2: "¿Cómo evaluarías la efectividad de una regla SIEM?"**
```
Respuesta esperada:
"Usaría estas métricas:

Precision = TP / (TP + FP)  [¿Qué % de alertas son reales?]
Recall = TP / (TP + FN)     [¿Qué % de ataques detectamos?]
F1-Score = Promedio de precision y recall

Target ideal:
- Precision > 80%   (no queremos 80% de ruido)
- Recall > 90%      (no queremos perder 90% de ataques)

Si una regla tiene precision baja (muchos FP), debo hacerla más específica.
Si tiene recall baja (pocos TP), debo hacerla más amplia."
```

**P3: "¿Cómo explicarías el mapeo de MITRE ATT&CK a un ejecutivo?"**
```
Respuesta esperada:
"MITRE es un lenguaje común para hablar de ataques. Le explicaría:

'Detectamos 15 intentos de T1110 (Brute Force) este mes.
Significa: 15 veces alguien intentó adivinar contraseñas.
Bloqueamos 14 (93% efectividad).
1 nos preocupa porque tuvo éxito - escalé a IR Team.

Comparado con el mes anterior (8 detecciones), aumentó 87%.
Posible razón: Vulnerabilidad en servicio expuesto. Recomiendo parchear urgente.'

En términos de negocio: Cuántos ataques detectamos, cuántos bloqueamos, qué riesgo queda."
```

---

## 💾 TL;DR - Referencia Rápida

### **¿Qué es SIEM?**
Herramienta que centraliza logs → normaliza → correlaciona → genera alertas → proporciona contexto para investigación.

### **Componentes Clave**
- **Logs**: Eventos crudos de firewalls, AD, Windows, antivirus
- **Alertas**: Logs que coinciden con reglas SIEM
- **Reglas**: Lógica (escrita en Sigma) que define qué es sospechoso
- **Correlación**: Relacionar eventos de múltiples fuentes
- **MITRE ATT&CK**: Clasificación estándar de técnicas de ataque

### **Proceso de Análisis**
1. **Evaluar**: Severidad, contexto, hora, usuario, fuente
2. **Investigar**: Buscar eventos previos, correlaciones, explicaciones legítimas
3. **Decidir**: Investigar más, escalar a IR, o cerrar como FP
4. **Actuar**: Contener, eradicar, documentar

### **Estructura Básica de Regla**
```yaml
logsource: (producto y servicio)
detection:
  selection: (qué buscamos)
  timeframe: (ventana de tiempo)
  condition: (cuándo alertar)
level: (severidad)
tags: (MITRE ATT&CK)
```

### **MITRE Mapping Quick**
- **Táctico**: ¿Para qué? (credential_access, persistence, etc.)
- **Técnica**: ¿Cómo? (T1110 - Brute Force)
- **Subtécnica**: ¿De qué forma exacta? (T1110_001 - Password Guessing)

### **Métricas Importantes**
- **False Positives** (FP): Alertas falsas → ruido → necesita tuning
- **True Positives** (TP): Alertas correctas → detecciones reales
- **Precision**: ¿Qué % de mis alertas son reales? (Target: >80%)
- **Recall**: ¿Qué % de ataques detecto? (Target: >90%)

---

## 📊 Tuning de Reglas: El Arte de Balancear

### **El Problema del Balance**

```
TOO SPECIFIC (Pocas alertas)        TOO BROAD (Muchas alertas)
├─ Precision ALTA                   ├─ Precision BAJA (80% ruido)
├─ Recall BAJA (pierdes ataques)   ├─ Recall ALTA (detecta casi todo)
├─ Ejemplos:                        ├─ Ejemplos:
│  • Login fallido = 10 en 1 min    │  • PowerShell = Siempre alerta
│  • PowerShell + parámetro +IP rare│  • Cualquier cambio de contraseña
│                                   └─ 1000+ alertas/día = Inútil
└─ Resultado: 5 alertas/día = OK   
```

### **Cómo Tunear Una Regla**

**Paso 1: Medir baseline**
```
"Mi regla de brute force genera 50 alertas/día"
¿Cuántas son TP (verdaderos positivos)? = 3
¿Cuántas son FP (falsos positivos)? = 47
Precision: 3/(3+47) = 6% ← TERRIBLE
```

**Paso 2: Investigar FPs**
```
De los 47 FPs:
- 30: Usuarios olvidando contraseña (ESPERABLE)
- 12: Procesos automatizados con credenciales vencidas (ESPERABLE)
- 5: Usuarios desde IP variable (ESPERABLE - trabajadores remotos)
```

**Paso 3: Ajustar**
```
Opción A - Aumentar threshold:
  Antes: >= 5 intentos
  Después: >= 15 intentos
  Resultado: 20 alertas/día, 3 TP (mejor ratio)

Opción B - Excluir usuarios conocidos:
  Añadir whitelist de service accounts y automated processes
  
Opción C - Cambiar timeframe:
  Antes: 5 minutos
  Después: 1 minuto (más restrictivo)
  
Opción D - Añadir condicionales:
  Si login fallido AND (IP nunca vista ANTES Y hora no laboral)
  Resultado: Solo alertar por anomalías reales
```

**Paso 4: Re-medir**
```
Después de cambios:
Nueva precision: 12 / (12+8) = 60% ← MEJOR
Nuevas alertas/día: 20 (vs 50 antes)
Analistas no abrumados ✓
Detectamos ataques reales ✓
```

### **Checklist de Tuning**

```
¿Tu regla tiene demasiadas alertas?
☐ Aumentar threshold (5 eventos → 10 eventos)
☐ Reducir timeframe (5 min → 1 min)
☐ Añadir condiciones adicionales
☐ Excluir usuarios/IPs conocidas
☐ Requiere múltiples fuentes (no solo 1)

¿Tu regla deja pasar ataques (recall bajo)?
☐ Reducir threshold
☐ Aumentar timeframe
☐ Remover condiciones innecesarias
☐ Añadir variaciones del comportamiento
☐ Incluir diferentes tipos de eventos

¿No sabes qué hacer?
☐ Analizar últimos 100 TP y 100 FP
☐ ¿Qué diferencia a TP de FP?
☐ Añade esa diferencia como condición
```

---

## 🔗 Conexiones a Otros Temas

Esta guía se conecta con:

- **🛡️ Detection Engineering**: Las reglas SIEM son la base de todo programa de detección
- **📊 MITRE ATT&CK Framework**: Cómo clasificar y reportar ataques detectados
- **🔍 Incident Response**: SIEM genera alertas que IR team investiga
- **🔐 Active Directory Security**: AD logs son críticos en SIEM para detectar compromiso
- **🪟 Windows Security Logs**: EventID (4625, 4688, etc.) son fuentes importantes
- **🔥 Firewall & IDS/IPS**: Logs de red alimentan detección en SIEM
- **📈 Threat Hunting**: Descubrimientos de threat hunters se convierten en reglas SIEM

---

## 📚 Preguntas de Reflexión (Para Profundizar)

1. **"¿Qué pasa si una fuente de logs (ej: AD) no está conectada al SIEM?"**
   - Respuesta: Punto ciego crítico, no detectas ataques que solo aparecen en esos logs

2. **"¿Cómo sabes si una regla es 'buena'?"**
   - Respuesta: TP rate alto (detecta ataques), FP rate bajo (no es ruidosa)

3. **"¿Quién decide si una alerta se escala a IR?"**
   - Respuesta: Tier 1/2 analyst (tú) basado en severidad, contexto, y disponibilidad

4. **"¿Cómo documentas un incidente detectado por SIEM?"**
   - Respuesta: Alerta + Timeline + Correlaciones + MITRE mapping + Acciones tomadas

5. **"¿Qué es más importante: Detectar ataques o no generar falsas alarmas?"**
   - Respuesta: AMBAS son críticas. Balance es arte de tuning.

---

## 🎓 Próximos Pasos

1. **Practica escribiendo Sigma rules** para casos de tu SOC
2. **Mapea tus alertas a MITRE ATT&CK** en tus reportes
3. **Analiza tus false positives** y tuneá reglas
4. **Documenta todo** - será tu portafolio de detection engineer

---

## 📌 Recuerda

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  SIEM no es magia. Es lógica + datos + reglas.       │
│                                                        │
│  Buenos logs → Normalizadas → Reglas específicas     │
│  ↓                                                     │
│  Alertas útiles → Investigación rápida → Respuesta   │
│                                                        │
│  Tu trabajo es hacer esa cadena perfecta.            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

**Última actualización**: 2024
**Versión**: 1.0
**Para**: Blue Team Students - TripleTen Bootcamp
