# 🔥 DETECTION ENGINEERING: Guía Completa para Blue Team
## Diseñar, Validar, Medir y Optimizar Reglas de Detección

---

## 📖 ¿Qué es Detection Engineering?

**En 30 segundos:**
Detection Engineering es el **proceso de ingeniería que diseña, valida, mide y optimiza reglas de detección basadas en ataques reales**. No es solo escribir reglas SIEM — es un **ciclo iterativo de mejoría continua** que transforma logs en alertas útiles mediante reglas que balancean precisión (pocos falsos positivos) y recall (pocas detecciones perdidas).

**Analogía:**
```
Escribir una regla = Construir una puerta
Detection Engineering = Diseñar, probar, medir, ajustar y mejorar esa puerta

Una puerta mal diseñada:
├─ Se queda abierta (bajo Recall = no detecta ataques)
├─ Se traba (alto FP = demasiadas falsas alarmas)
└─ Necesita constante mantenimiento (sin mediciones)

Una puerta bien engineered:
├─ Se abre/cierra correctamente (buen Precision + Recall)
├─ Dura años (validada)
└─ Mejora basado en feedback (iteración)
```

---

## 🎯 ¿Por Qué Un Blue Team Necesita Detection Engineering?

### En La Realidad del Trabajo

| Escenario | Sin Detection Engineering | Con Detection Engineering |
|-----------|---------------------------|---------------------------|
| Ataque nuevo ocurre | "¿Cómo lo detectamos?" | Regla diseñada, validada, en producción en 24 horas |
| 1000 alertas/día de una regla | Analistas abrumados, ignoran alertas | Regla tuneada: 50 alertas/día, 95% precision |
| Atacante cambia técnica | "Nuestra regla no funciona más" | Threat hunt → nueva regla diseñada y validada |
| Falso positivo detectado | Nada (continúa generando ruido) | Se investiga, regla se ajusta, FP se elimina |
| Compliance (auditoría) | "¿Detectamos este ataque?" | Dashboard mostrando: 47 detecciones, 45 validadas, 2 FP |

### Preguntas de Entrevista

1. **"¿Cuál es la diferencia entre escribir una regla SIEM y Detection Engineering?"**
   - Esperado: "Una regla es una línea de código. Detection Engineering es el proceso de diseñarla, validarla contra logs históricos, medir precision/recall, tunarla, y mantenerla actualizada basado en incidentes reales."

2. **"¿Qué es más importante: detectar todos los ataques o minimizar falsos positivos?"**
   - Esperado: "Ambos. Pero el balance depende del contexto. Para admin accounts = recall alto (detectar casi todo). Para usuarios normales = precision alto (minimizar FP)."

3. **"¿Cómo validarías una regla nueva antes de ponerla en producción?"**
   - Esperado: "Contra logs históricos (¿detecta los incidentes conocidos?), medir precision/recall, ver si genera FP normales, tunar, y después validar nuevamente."

4. **"¿Cuál es el mejor indicador de que una regla es 'buena'?"**
   - Esperado: "No es solo precision o recall. Es el balance (F1-Score), baja en volumen (pocas alertas), y que los analistas piensan que la alerta es valiosa."

---

## 🔍 El Concepto Desglosado

### Parte 1: El Ciclo de Vida Completo de una Regla

```
┌─────────────────────────────────────────────────────────────┐
│  FASE 1: DESCUBRIMIENTO                                     │
│  (Threat Hunter o Analyst detecta patrón nuevo)             │
├─────────────────────────────────────────────────────────────┤
│  Entrada: Incidente real o patrón de ataque documentado     │
│  Ejemplo: "Vimos 50 conexiones a puerto 4444 de una IP rara"│
│  Salida: Necesidad documentada de detección                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 2: DISEÑO                                             │
│  (Detection Engineer diseña la regla)                       │
├─────────────────────────────────────────────────────────────┤
│  Preguntas: ¿Qué logs necesito? ¿Qué campos?              │
│             ¿Qué valores indican ataque?                   │
│             ¿Cuántos eventos = alerta?                     │
│             ¿En qué timeframe?                             │
│  Salida: Sigma rule o KQL rule escrita                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 3: VALIDACIÓN                                         │
│  (Prueba contra logs históricos)                            │
├─────────────────────────────────────────────────────────────┤
│  Pregunta 1: ¿Detecta los incidentes históricos conocidos?│
│  Pregunta 2: ¿Genera falsos positivos?                      │
│  Pregunta 3: ¿Los FP son "aceptables"?                     │
│  Salida: Métricas de TP, FP, Precision, Recall            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 4: TUNING (si es necesario)                          │
│  (Ajustar la regla basado en validación)                   │
├─────────────────────────────────────────────────────────────┤
│  Opción A: ¿Precision baja? (muchos FP)                    │
│           → Hacer regla más específica                      │
│  Opción B: ¿Recall baja? (pierdes ataques)                 │
│           → Hacer regla más amplia                         │
│  Salida: Regla ajustada, métricas nuevas                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 5: PRODUCCIÓN                                         │
│  (Desplegar en el SIEM)                                    │
├─────────────────────────────────────────────────────────────┤
│  Acciones:                                                  │
│  ├─ Desplegar regla                                         │
│  ├─ Monitorear volumen de alertas (¿esperado?)            │
│  ├─ Hacer baselines (cuántas alertas/día es normal)       │
│  └─ Documentar: qué detecta, por qué, cómo responder      │
│  Salida: Regla viva en el SIEM                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 6: MONITOREO Y MEJORA CONTINUA                       │
│  (Análisis de alertas reales)                              │
├─────────────────────────────────────────────────────────────┤
│  Cada semana:                                               │
│  ├─ Analizar FP genuinos                                    │
│  ├─ ¿Puedo excluir este FP?                               │
│  ├─ ¿Hay ataques que no detecté?                          │
│  └─ ¿Técnicas nuevas de ataque que necesito cover?        │
│  Salida: Regla mejorada, feedback loop cerrado            │
└─────────────────────────────────────────────────────────────┘
```

### Parte 2: Las Cuatro Métricas Fundamentales

Todo en Detection Engineering se reduce a entender estas 4 métricas:

```
TP (True Positive)   = Alerta correcta. Atacante real detectado.
FP (False Positive)  = Alerta falsa. No había ataque.
TN (True Negative)   = No hubo alerta Y no había ataque. ✓ Correcto
FN (False Negative)  = No hubo alerta PERO había ataque. ✗ Peor error

Visualización:
                   EXISTE ATAQUE    NO EXISTE ATAQUE
ALERTA DISPARA  →    TP (bueno)         FP (malo)
NO ALERTA       →    FN (peor)          TN (bueno)
```

**Ejemplo concreto:**

```
Incidente: 50 intentos de login fallidos en 5 minutos, misma IP

Si tu regla generó alerta: TP (correcto, era ataque)
Si tu regla no generó alerta: FN (peor error, no detectaste ataque)
Si tu regla generó alerta pero era admin reseteando contraseña: FP
Si la noche transcurrió normal y sin alertas: TN
```

### Parte 3: Las Dos Métricas Que Importan

De TP, FP, TN, FN se derivan dos métricas críticas:

**PRECISION** = ¿Cuántas de mis alertas son reales?
```
Precision = TP / (TP + FP)

Ejemplo:
- 100 alertas generadas
- 90 eran ataques reales (TP)
- 10 eran falsas (FP)
- Precision = 90 / (90 + 10) = 90%

Significado: De 100 alertas, 90 son reales.
Si precision es 50%, es decir, la mitad son falsas = RUIDO
```

**RECALL** = ¿Cuántos ataques reales detecto?
```
Recall = TP / (TP + FN)

Ejemplo:
- En los últimos 30 días, hubieron 100 ataques reales
- Mi regla detectó 95 (TP)
- Se me escaparon 5 (FN)
- Recall = 95 / (95 + 5) = 95%

Significado: Detecto 95 de cada 100 ataques.
Si recall es 50%, me escapan la mitad = PELIGROSO
```

**Balance: F1-Score**
```
F1-Score = 2 * (Precision * Recall) / (Precision + Recall)

Objetivo:
- Precision > 80% (no quiero 80% de ruido)
- Recall > 90% (no quiero perder 90% de ataques)
- F1-Score > 0.85 (buen balance)
```

---

## ⚙️ Cómo Escribir una Regla Desde Cero

### Paso 1: Entender el Ataque

**Escenario:** Un atacante intenta acceso lateral a un servidor SQL.

**¿Qué logs vamos a ver?**
```
Firewall: Intento de conexión a puerto 1433 (SQL Server)
         Origen: IP interna (workstation)
         Destino: BBDD-SERVER (servidor BD)
         Action: BLOCKED (porque no está autorizado)

Windows (BBDD-SERVER): EventID 4625 (Login fallido)
                      Usuario: attacker_creds
                      Status: 0xC0000064 (mala contraseña)

¿Por qué es importante?: Login fallido a BD en horario anormal = suspicious
```

### Paso 2: Diseñar la Regla (Sigma)

```yaml
title: Ataque Lateral - Intentos de Login a SQL Server
id: sql-lateral-movement-001
status: experimental
description: Detecta múltiples intentos fallidos contra SQL Server (puerto 1433)
author: Detection Engineering Team
date: 2024-01-15

logsource:
  product: firewall
  service: fortinet

# PARTE 1: QUÉ BUSCAMOS
detection:
  # Condición A: Intentos a puerto SQL (1433)
  sql_access_attempt:
    EventType: DENIED
    DestinationPort: 1433
    DestinationIP: 10.0.9.15  # BBDD-SERVER
  
  # Condición B: Desde workstations (no admin)
  from_workstation:
    SourceIP|startswith:
      - '10.0.100.'   # Workstation subnet
      - '10.0.101.'
  
  # Condición C: No es administrador conocido
  not_admin:
    SourceIP|exclude:
      - '10.0.50.10'   # Admin workstation
      - '10.0.50.11'
  
  # PARTE 2: TIMEFRAME
  timeframe: 5m
  
  # PARTE 3: CONDICIÓN DE ALERTA
  condition: sql_access_attempt AND from_workstation AND not_admin

# INFORMACIÓN ADICIONAL
falsepositives:
  - SQL administrators accessing from unusual workstations
  - Legitimate application trying to connect to SQL

level: high
tags:
  - attack.lateral_movement
  - attack.t1570
  - BBDD
```

### Paso 3: Validar Contra Logs Históricos

**Pregunta:** ¿Cuáles fueron los incidentes de lateral movement históricos?

```
Histórico (últimos 90 días):
┌─ Incidente #1 (2024-01-10):
│  └─ Atacante intentó conectar a BBDD-SERVER desde WKS-0055
│     → Logs: Firewall DENIED, EventID 4625 en BD
│     → ¿Mi regla lo detectaría? SÍ ✓
│
├─ Incidente #2 (2023-12-28):
│  └─ Atacante intentó conexión a puerto 1433
│     → Logs: Firewall DENIED, SQL auth failed
│     → ¿Mi regla lo detectaría? SÍ ✓
│
└─ Incidente #3 (2023-12-15):
   └─ Admin legítimo desde laptop nueva (no en whitelist)
      → Logs: Firewall DENIED (porque IP no autorizada)
      → ¿Mi regla lo detectaría? SÍ, pero es FP ✗
```

### Paso 4: Ejecutar la Regla en Logs Históricos

**Simulación (test retrospectivo):**

```
Logs históricos: 90 días

Regla ejecutada contra todos los logs:
├─ TP (verdaderos positivos): 2 incidentes detectados
├─ FP (falsos positivos): 1 admin legítimo
├─ TN (verdaderos negativos): 259,999 eventos normales
└─ FN (falsos negativos): 0 incidentes no detectados

Métricas:
├─ Precision: 2 / (2 + 1) = 66.6% ← BAJA (mucho ruido)
├─ Recall: 2 / (2 + 0) = 100% ← BUENA (detecta todo)
└─ F1-Score: 0.8 ← ACEPTABLE pero precision es baja
```

### Paso 5: Tuning (Ajustar la Regla)

**Problema identificado:** Precision baja (66.6%), especialmente porque admin legítimo genera FP.

**Solución:** Mejorar la exclusión de admins legítimos

```yaml
# VERSIÓN AJUSTADA

detection:
  sql_access_attempt:
    EventType: DENIED
    DestinationPort: 1433
    DestinationIP: 10.0.9.15

  from_workstation:
    SourceIP|startswith:
      - '10.0.100.'
      - '10.0.101.'

  # VERSIÓN MEJORADA: Exclusión más granular
  known_legitimate:
    SourceIP|exclude:
      - '10.0.50.10'     # Admin workstation 1
      - '10.0.50.11'     # Admin workstation 2
      - '10.0.102.105'   # CIO laptop (nuevo)
    # Alternativamente, buscar "admin" en User field si está disponible:
    # User|exclude:
    #   - '*admin*'
    #   - 'CORP\sa_*'    # Service accounts

  timeframe: 5m
  condition: sql_access_attempt AND from_workstation AND not known_legitimate

level: high
```

**Validar de nuevo:**

```
Nueva ejecución contra logs históricos:
├─ TP: 2 incidentes detectados
├─ FP: 0 (admin ahora está en exclusión)
├─ Precision: 2 / (2 + 0) = 100% ← EXCELENTE
├─ Recall: 2 / (2 + 0) = 100% ← EXCELENTE
└─ F1-Score: 1.0 ← PERFECTO
```

### Paso 6: Desplegar en Producción

```
1. Cambio aprobado por equipo de seguridad
2. Documentación:
   ├─ Qué detecta: Intentos de login lateral a SQL Server
   ├─ Por qué importa: SQL Server contiene datos sensibles
   ├─ Cómo responder: Investigar origen, bloquear IP si es externo
   └─ Métricas: Precision 100%, Recall 100%
3. Desplegar en SIEM
4. Monitorear alertas en vivo durante 7 días
5. Ajustar si es necesario
```

---

## 📊 Ejemplos de Reglas Reales (Sigma)

### Ejemplo 1: Brute Force contra Active Directory (Windows)

```yaml
title: AD Brute Force - Multiple Failed Logins
id: ad-brute-force-001
logsource:
  product: windows
  service: security

detection:
  failed_login:
    EventID: 4625
    LogonType: 3
    Status: 'C0000064'  # Wrong password
  
  timeframe: 5m
  condition: failed_login | count(TargetUserName) by SourceIp >= 5

level: high
tags:
  - attack.credential_access
  - attack.t1110_001

# Métricas esperadas:
# Precision: 85% (algunos false positives de users olvidando contraseña)
# Recall: 95% (detecta casi todos los brute force)
```

**Log crudo que dispara esta regla:**

```text
<134>Jan 15 09:14:01 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:03 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:05 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
[+3 más en 5 minutos, misma IP]
→ ALERTA DISPARADA
```

---

### Ejemplo 2: Descarga de Credential Dumper (Antivirus)

```yaml
title: Credential Dumping Tool Downloaded
id: cred-dump-download-001
logsource:
  product: antivirus
  service: webfilter

detection:
  suspicious_tools:
    FileName|endswith:
      - 'mimikatz.exe'
      - 'lsass_dump.exe'
      - 'hash_dumper.exe'
      - 'passwords.txt'
      - 'ntds_extract.exe'
    Action: BLOCKED  # El AV lo detectó
  
  # O alternativamente, desde Windows (descarga):
  # EventID: 4688  # Process creation
  # CommandLine|contains|all:
  #   - 'powershell'
  #   - 'mimikatz'

  condition: suspicious_tools

level: critical
tags:
  - attack.credential_access
  - attack.t1003
```

**Log crudo:**

```text
<134>Jan 15 14:32:22 AV-SRV01 CEF:0|Symantec|Endpoint Protection|14.3|download|File Blocked|8|
  src=192.168.1.55 fname=mimikatz.exe threatLevel=critical
  act=quarantine fileHash=SHA256:9f8a...c221
```

---

### Ejemplo 3: PowerShell Encoding (Windows)

```yaml
title: Suspicious PowerShell with Encoding Parameter
id: ps-encoded-command-001
logsource:
  product: windows
  service: security

detection:
  powershell_encoded:
    EventID: 4688  # Process creation
    Image|endswith: 'powershell.exe'
    CommandLine|contains|all:
      - '-Enc'
      - '-EncodedCommand'
  
  # Exclusión de admins legítimos
  not_it_team:
    User|exclude:
      - 'CORP\IT_Team*'
      - 'CORP\Admin*'
      - 'CORP\sa_*'

  condition: powershell_encoded AND not_it_team

level: medium
tags:
  - attack.execution
  - attack.t1059_001
```

**Log crudo:**

```text
<134>Jan 15 14:35:15 WKS-0231 MSWinEventLog 1 Security 209981 ... 4688 ...
  User: CORP\jsmith
  Process Name: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  Command Line: powershell.exe -nop -w hidden -enc JABzAGgAZWBs...
  ParentImage: C:\Program Files\Outlook\OUTLOOK.EXE
```

**¿Por qué es suspicious?** PowerShell + encoding típicamente indica malware.

---

### Ejemplo 4: Conexión C2 (Firewall)

```yaml
title: Suspicious Outbound Connection to Known C2 IP
id: c2-connection-001
logsource:
  product: firewall
  service: fortinet

detection:
  outbound_c2:
    EventType: ALLOWED  # Fue permitida (firewall no vio como amenaza)
    DestinationIP:
      - '198.51.100.50'  # Conocido C2 de threat intel
      - '203.0.113.45'   # Otro C2
    Protocol|contains:
      - 'TCP'
      - 'UDP'
    Action: ALLOW
  
  timeframe: 1h
  condition: outbound_c2

level: critical
tags:
  - attack.command_and_control
  - attack.t1071
```

**Log crudo:**

```text
<134>Jan 15 14:40:33 FW-EDGE-01 CEF:0|Fortinet|FortiGate|7.2|traffic|Traffic Log|3|
  src=192.168.1.100 spt=52301 dst=198.51.100.50 dpt=4444 proto=tcp
  act=allow reason=policy_allow policyid=5 srcintf=internal dstintf=wan1
  duration=3650 sentbyte=2048 rcvdbyte=5120
  # ← Conexión activa, datos siendo enviados/recibidos
```

---

## 🧪 Validación Completa: Antes y Después

### Caso: Mejorar una Regla Existente

**Regla ORIGINAL (con problemas):**

```yaml
title: Any Firewall Block
description: Alert on any traffic block
detection:
  selection:
    EventType: BLOCKED
  condition: selection
level: high
```

**Problema:** Genera 10,000+ alertas/día (casi inútil)

```
Validación de regla original:
├─ TP (verdaderos ataques bloqueados): 15
├─ FP (tráfico legítimo bloqueado): 9,985
├─ Precision: 15 / (15 + 9,985) = 0.15% ← HORRIBLE
├─ Recall: 15 / (15 + 0) = 100% ← Detecta todo pero con ruido
└─ F1-Score: 0.003 ← INÚTIL
```

**Regla MEJORADA v1 (después de tuning):**

```yaml
title: Firewall Block from External IP to Internal Server
detection:
  selection:
    EventType: BLOCKED
    SourceIP|startswith: '203.'  # External IPs only
    DestinationIP: '10.0.9.15'   # Critical DB server
    DestinationPort: 1433        # SQL Server
  condition: selection
level: high
```

**Validación:**

```
├─ TP: 8
├─ FP: 2 (ISP issues, DNS resolution attempts)
├─ Precision: 8 / (8 + 2) = 80% ← MEJOR
├─ Recall: 8 / (8 + 7) = 53% ← Bajo, se nos escapan algunos
└─ F1-Score: 0.64 ← Aceptable pero recall es bajo
```

**Regla MEJORADA v2 (después de más tuning):**

```yaml
title: Firewall Block - External to Critical Resource
detection:
  selection:
    EventType: BLOCKED
    SourceIP|startswith: '203.'
    DestinationIP:
      - '10.0.9.15'   # DB Server
      - '10.0.8.50'   # File Server
      - '10.0.7.100'  # Mail Server
    DestinationPort|in:
      - 1433   # SQL
      - 445    # SMB
      - 25     # SMTP
  timeframe: 1h
  condition: selection | count() by SourceIP >= 3  # 3+ blocks from same IP
level: high
```

**Validación final:**

```
├─ TP: 12
├─ FP: 1 (partner VPN with IP overlap)
├─ Precision: 12 / (12 + 1) = 92% ← EXCELENTE
├─ Recall: 12 / (12 + 2) = 86% ← BUENO
└─ F1-Score: 0.89 ← PROFESIONAL
```

---

## ❌ Errores Comunes en Detection Engineering

### Error #1: Escribir regla sin validar contra histórico

```
❌ MAL:
"Escribo la regla, la despliego, y espero a ver qué pasa"

✅ BIEN:
"Escribo la regla, la ejecuto contra 90 días de logs históricos,
calculo precision/recall, y solo entonces la despliego"

Razón: Sin validación previa, es como disparar al aire.
```

---

### Error #2: Confundir "baja precision" con "regla compleja"

```
❌ MAL:
"Mi regla tiene baja precision, debo hacerla más compleja"

✅ BIEN:
"Mi regla tiene baja precision (mucho ruido), debo hacerla MÁS ESPECÍFICA:
- Excluir sistemas conocidos
- Agregar condiciones adicionales
- Aumentar threshold de eventos"

Regla más compleja ≠ mejor precision.
Regla más específica = mejor precision.
```

---

### Error #3: Olvidar el contexto de negocio

```
❌ MAL:
"Todo PowerShell = alerta. Todo login fallido = alerta"

✅ BIEN:
"El contexto importa:
- ¿Quién? (admin vs user normal)
- ¿Cuándo? (3 AM vs 10 AM)
- ¿Dónde? (desde IP interna vs externa)
- ¿Por qué? (cambio de contraseña esperado?)"

Ejemplo: 5 intentos de login fallidos
├─ Si es admin a las 3 AM desde IP externa: CRÍTICO
├─ Si es user normal olvidando contraseña: IGNORAR
```

---

### Error #4: No documentar la regla

```
❌ MAL:
Regla creada, desplegada, pero:
├─ Nadie sabe qué detecta exactamente
├─ No hay playbook de respuesta
├─ Al mes siguiente, "¿por qué tenemos 100 alertas de esto?"

✅ BIEN:
Cada regla debe documentar:
├─ Qué detecta (descripción clara)
├─ Por qué es importante (MITRE ATT&CK)
├─ Precision/Recall esperados
├─ Cómo responder (playbook)
├─ Cuándo revisar (si metrics cambian)
└─ Quién mantiene la regla (owner)
```

---

## 🧪 Ejercicios Prácticos

### Ejercicio #1: Analizar una Regla Existente

**Regla:**

```yaml
title: Suspicious Admin Account Activity
detection:
  selection:
    User|contains: 'admin'
  condition: selection
level: high
```

**Pregunta:** ¿Cuál es el problema con esta regla?

**Respuesta esperada:**
```
Problemas:
1) TOO BROAD - "admin" está en muchos users: admin, admin_user, backup_admin, etc.
2) SIN TIMEFRAME - No agrupa eventos en tiempo
3) SIN THRESHOLD - Alertaría por CUALQUIER evento de un admin (1000+ alertas/día)
4) SIN CONTEXTO - No diferencia login normal de cambio de permisos
5) Precision: Probablemente <5%
6) Recall: Probablemente 100% (pero con ruido masivo)

Versión mejorada:
title: Admin Account Privileged Operation
detection:
  selection:
    EventID|in: [4728, 4729, 4730, 4723]  # Admin group changes, password changes
    User|contains: 'domain_admin'
  time_based:
    Hour: [22, 23, 0, 1, 2, 3, 4, 5]  # Horario anormal (10 PM - 5 AM)
  condition: selection AND time_based
level: high
```

---

### Ejercicio #2: Diseñar una Regla Nueva

**Escenario:** El equipo de Threat Intel reporta que atacantes usan msiexec.exe para instalar malware.

**Tarea:** Diseña una regla para detectar esto.

**Respuesta esperada:**

```yaml
title: Suspicious msiexec.exe Execution with Network Activity
id: msiexec-suspicious-001
logsource:
  product: windows
  service: sysmon  # o Endpoint Detection

detection:
  msiexec_exec:
    EventID: 1  # Process creation
    Image|endswith: 'msiexec.exe'
    
  # Exclusión de instalación legítima
  not_legitimate:
    CommandLine|exclude:
      - '*software update*'
      - '*patch*'
      - '*install*'  # palabras clave de instalación legítima
    ParentImage|exclude:
      - 'C:\Windows\System32\svchost.exe'
      - 'C:\Program Files\*'
  
  # Correlación: msiexec + network connection
  network_connection:
    EventID: 3  # Network connection
    Image|endswith: 'msiexec.exe'
    DestinationPort|exclude:
      - 80    # HTTP
      - 443   # HTTPS
    
  timeframe: 5m
  condition: msiexec_exec AND not_legitimate AND network_connection

level: high
tags:
  - attack.execution
  - attack.t1218_009

falsepositives:
  - Legitimate Windows updates installing via msiexec
  - Corporate software deployment systems
```

**Métricas esperadas:**
```
Precision: ~85% (algunos FP de updates, pero manejables)
Recall: ~90% (detecta la mayoría de msiexec malicioso)
F1-Score: 0.87 (bueno)
```

---

### Ejercicio #3: Medir una Regla Existente

**Datos históricos (último mes):**

```
Regla: "Conexiones a puerto 4444"

Ejecución retrospectiva:
├─ Alertas generadas: 120
├─ Ataques confirmados luego: 12
├─ Falsos positivos confirmados: 8
├─ Ataques no detectados: 3

Preguntas:
1) ¿Cuál es la Precision?
2) ¿Cuál es el Recall?
3) ¿Es una buena regla?
```

**Respuesta esperada:**

```
1) Precision = TP / (TP + FP)
   = 12 / (12 + 8)
   = 12 / 20
   = 60%
   
   Interpretación: De las 120 alertas, solo 60% fueron reales.
   40% fue ruido.

2) Recall = TP / (TP + FN)
   = 12 / (12 + 3)
   = 12 / 15
   = 80%
   
   Interpretación: Detecté 80% de los ataques.
   20% se me escaparon.

3) ¿Es buena?
   - Precision de 60% es baja (mucho ruido)
   - Recall de 80% es aceptable pero podría mejorar
   - F1-Score = 2 * (0.6 * 0.8) / (0.6 + 0.8) = 0.686
   
   Conclusión: NECESITA TUNING
   
   Opción A: Aumentar precision (menos FP):
   ├─ Excluir IPs legítimas de puerto 4444
   ├─ Añadir condición: solo alertar si desde IP externa
   
   Opción B: Aumentar recall (detectar más ataques):
   ├─ Incluir puertos relacionados: 4445, 4446, etc.
   ├─ Añadir correlación con otros eventos
```

---

## 🎯 Preguntas de Entrevista por Nivel

### Level 1 (Entry-Level)

**P1: "¿Qué es Detection Engineering?"**
> "El proceso de diseñar, validar, medir y optimizar reglas de detección basadas en ataques reales. No es solo escribir reglas SIEM — es un ciclo iterativo de mejora continua."

**P2: "¿Cuál es la diferencia entre una regla SIEM y Detection Engineering?"**
> "Una regla SIEM es una línea de código que detecta un patrón. Detection Engineering es el proceso de validar esa regla contra logs históricos, medir su precision/recall, ajustarla, y mantenerla actualizada."

**P3: "¿Por qué los falsos positivos son un problema?"**
> "Porque generan ruido. Si 80% de mis alertas son falsas, los analistas empiezan a ignorar alertas, incluyendo las reales. Los FP deben minimizarse mediante tuning."

### Level 2 (Mid-Level)

**P1: "¿Cómo escribirías una regla para detectar Kerberoasting?"**
> "Buscaría EventID 4769 (Service Ticket solicitado) con encryptionType RC4 (fácil de crackear), en corto timeframe de múltiples Service Tickets. Excluiría solicitudes legítimas del sistema. Validaría contra logs históricos de incidentes conocidos, mediría precision/recall, y tunaría según FP."

**P2: "¿Cuál es más importante: Precision o Recall?"**
> "Ambos. Pero depende del contexto. Para Domain Admin accounts = Recall alto (detectar casi todo, aceptar algo de ruido). Para usuarios normales = Precision alto (minimizar FP, aceptar perder algunos ataques). El ideal es balance: Precision >80%, Recall >90%."

**P3: "¿Cómo validarías una regla antes de ponerla en producción?"**
> "1) Ejecuto la regla contra 90 días de logs históricos. 2) Mido TP, FP, TN, FN. 3) Calculo Precision y Recall. 4) Comparo con incidentes conocidos (¿detecta los que se vieron?). 5) Si precision < 80%, tuning. 6) Una vez validada, despliego con monitoreo."

### Level 3 (Senior / Detection Engineer)

**P1: "¿Cómo diseñarías un programa de Detection Engineering desde cero?"**
> "1) Threat Modeling: ¿Cuáles son nuestros principales adversarios? ¿Qué técnicas usan (MITRE ATT&CK)?
> 2) Priorización: ¿Cuáles son los ataques más probables y de mayor impacto?
> 3) Para cada técnica: Diseñar regla, validar contra histórico, medir.
> 4) Implementar: Desplegar en tiers (test → staging → production).
> 5) Monitoreo: Analizar FP, mejorar reglas semanalmente.
> 6) Evolucionar: Agregar nuevas técnicas basado en threat intel y nuevos ataques detectados."

**P2: "¿Cuál es el mayor desafío en Detection Engineering?"**
> "El balance entre Precision y Recall. Precision muy alta = pocos FP pero pierdo ataques. Recall muy alto = detecto casi todo pero con mucho ruido. La solución es: 1) Entender el contexto de negocio (riesgo aceptable), 2) Tuning iterativo, 3) Feedback de incidentes reales, 4) Automatizar tuning si es posible."

**P3: "¿Cómo medirías el éxito de tu programa de Detection Engineering?"**
> "1) Métricas de reglas: Precision, Recall, F1-Score de cada regla.
> 2) Métricas operacionales: ¿Cuántos ataques detectamos vs. que se nos escaparon?
> 3) Tiempo de respuesta: ¿Cuánto tarda desde alerta hasta investigación?
> 4) Impacto: ¿Cuántos incidentes fueron prevenidos o acortados?
> 5) Tendencias: ¿Las técnicas de ataque están evolucionando? ¿Nuestras reglas evolucionan con ellas?
> El verdadero éxito es cuando la organización dice: 'gracias a tus reglas, detectamos X ataque en 5 minutos vs. 5 días manualmente.'"

---

## 🔗 Conexiones a Otros Temas

- 🔥 **SIEM**: Plataforma donde se despliegan las reglas
- 📊 **MITRE ATT&CK**: Framework que guía qué técnicas detectar
- 🔐 **Active Directory**: Logs de AD son críticos para detection engineering
- 🌐 **Windows Security Logs**: EventIDs son la base de muchas reglas
- 🔍 **Threat Hunting**: Descubrimientos se convierten en reglas
- 🛡️ **Incident Response**: Cada incidente genera oportunidad para nueva regla
- 📈 **Threat Intelligence**: Alimenta qué reglas crear (malware hashes, C2 IPs, etc.)

---

## 💾 TL;DR - Referencia Rápida

### El Ciclo de Life de una Regla
```
Descubrimiento → Diseño → Validación → Tuning → Producción → Monitoreo
                                      ↑_________________↑
                                      (feedback loop)
```

### Las 4 Métricas
```
TP = Ataque real detectado (bueno)
FP = Falsa alarma (malo, genera ruido)
TN = No alarma + no ataque (bueno)
FN = Ataque no detectado (peor)
```

### Las 2 Métricas Que Importan
```
Precision = TP / (TP + FP) → % de alertas que son reales (Target: >80%)
Recall = TP / (TP + FN)   → % de ataques que detecto (Target: >90%)
```

### Estructura Básica de Regla (Sigma)
```yaml
title: Nombre descriptivo
logsource: (qué logs)
detection:
  selection: (qué buscamos)
  timeframe: (ventana de tiempo)
  condition: (cuándo alertar)
level: (severidad)
tags: (MITRE ATT&CK)
```

### Checklist de Tuning
```
¿Precision baja? → Hacer más específica
  - Excluir sistemas conocidos
  - Añadir condiciones
  - Aumentar threshold

¿Recall bajo? → Hacer más amplia
  - Reducir threshold
  - Incluir variaciones del ataque
  - Correlacionar con otros eventos
```

---

## 📌 Realidad en Producción

### Caso Real #1: Regla que salvó la empresa

```
Regla: "EventID 4720 + 4728 + 4723 en <5 minutos"
(Cuenta creada + agregada a admin + contraseña cambiada)

Resultado: Detectó compromiso de AD en 3 minutos
Sin la regla: El ataque hubiera durado horas o días

Impacto: Previno exfiltración de datos críticos
```

### Caso Real #2: Regla que generaba mucho ruido

```
Antes: "Cualquier login fallido = ALERT"
└─ 50,000 alertas/día, precision 2%

Después: "5+ login fallidos + mismo user + mismo IP + <5 min"
└─ 500 alertas/día, precision 85%

Lección: Especificidad > cantidad
```

---

## 📚 Próximos Pasos

1. **Aprende Sigma** (el estándar de reglas)
2. **Practica escribiendo reglas** contra logs reales
3. **Valida contra histórico** (no depliegues sin validar)
4. **Mide siempre** (Precision, Recall, F1-Score)
5. **Iteración continua** (cada incidente = oportunidad de mejorar)

---

## 📌 Recuerda

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Detection Engineering ≠ Escribir reglas                  │
│  Detection Engineering = Diseñar, validar, medir,         │
│                        optimizar y mantener reglas        │
│                                                            │
│  Una regla sin validación = Un ataque sin prevención      │
│  Una regla sin métricas = Conducir sin ver el camino      │
│  Una regla sin tuning = Trabajo inacabado                 │
│                                                            │
│  El éxito es cuando los analistas dicen:                  │
│  "Esta alerta siempre es valiosa, nunca la ignoro"        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

**Última actualización**: 2024
**Versión**: 1.0
**Para**: Blue Team Students - TripleTen Bootcamp
