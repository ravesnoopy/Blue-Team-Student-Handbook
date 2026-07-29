# 🔍 DIGITAL FORENSICS & INCIDENT RESPONSE (DFIR): Guía Completa para Blue Team

## Entender la Disciplina de Investigación

---

## 📖 ¿Qué es DFIR?

**En 60 segundos:**
DFIR = **Digital Forensics** + **Incident Response**

- **Digital Forensics**: Investigación post-ataque. "¿Qué pasó exactamente? ¿Dónde está la evidencia?"
- **Incident Response**: Durante/después del ataque. Contener, erradicar, recuperar sistemas.

**Distinción Clave:**
```
SOC/Detection Engineer → Detecta ataque en progreso → "¡ALERTA!"
Incident Responder → Recibe alerta → Toma decisiones en tiempo real → "Aislar servidor AHORA"
DFIR Analyst → Después de contención → Investigación profunda → "Así es lo que pasó"
```

**Vista de timeline:**
```
Atacante entra → SOC detecta → IR responde/contiene → DFIR investiga → Reporte + Prevención
      (0h)          (5min)           (30min)            (horas/días)    (recomendaciones)
```

---

## 🎯 ¿Por Qué Un Profesional de Blue Team Necesita DFIR?

### Escenarios Reales del Trabajo

| Escenario | Qué Sucede |
|-----------|-----------|
| Ransomware ataca empresa | IR lo contiene. DFIR descubre cómo entraron, qué encriptaron, cómo prevenir próxima vez |
| Se detecta data breach | IR aísla sistemas. DFIR responde: "¿Qué fue robado? ¿Cuándo? ¿Cuánto?" |
| Se sospecha insider threat | IR no sabe si es real. DFIR lo prueba con timestamps, logs de acceso, trails de email |
| Brote de malware | IR lo elimina. DFIR rastrea paciente cero: primera máquina infectada, ruta de movimiento lateral |
| Auditoría de compliance | "¿Detectaron este ataque?" DFIR produce timeline + evidencia como prueba |

### Preguntas de Entrevista Que Te Harán

1. **"Cuéntame de una investigación forense que hayas hecho"**
   - Esperado: Estructura de timeline, herramientas usadas, evidencia recolectada, conclusiones

2. **"¿Cuál es la diferencia entre 'causa raíz' y 'primer log sospechoso'?"**
   - Esperado: Causa raíz = primera acción que funcionó (acceso inicial). Primer log = detección de esa acción.

3. **"¿Cómo preservas la cadena de custodia?"**
   - Esperado: Hashing, imaging, documentación, sin modificar datos

4. **"Encuentras un archivo encriptado en servidor comprometido. ¿Cómo procedes?"**
   - Esperado: Imágenes, hashes, no desencriptar (preserva evidencia), analizar offline

---

## 🔍 El Concepto Desglosado

### Parte 1: El Equipo DFIR y Roles

```
SECURITY OPERATIONS CENTER (SOC)
│
├─ SIEM Analyst
│  ├─ Monitorea alertas
│  ├─ Ve "100 intentos fallidos de login a Domain Admin en 5 minutos"
│  └─ Escala a Incident Response
│
├─ Detection Engineer
│  ├─ Construyó la regla que disparó
│  └─ Después usa hallazgos DFIR para mejorar detección
│
└─ Incident Response Team
   ├─ Recibe alerta de SOC
   ├─ Decide: "Aislar servidor comprometido AHORA"
   ├─ Preserva evidencia (¡no borres disco aún!)
   ├─ Recolecta dump de memoria mientras sistema está corriendo
   └─ Llama DFIR Analyst una vez sistema aislado
   
   └─ DFIR Analyst (se une investigación)
      ├─ Toma control de evidencia
      ├─ Imagena discos (copia forense)
      ├─ Analiza offline
      ├─ Construye timeline
      ├─ Identifica causa raíz
      ├─ Documenta cadena de ataque
      └─ Produce reporte para IR + Detection Engineering
```

### Parte 2: Causa Raíz vs Primer Log vs Primer Movimiento Sospechoso

**Esto es crítico. La mayoría de gente lo confunde.**

```
Ejemplo de timeline: Brote ransomware en Server-DB-01

┌─ 2024-01-15 08:00:00 ─ Admin crea backup (legítimo)
├─ 2024-01-15 08:15:00 ─ PRIMER LOG GENERADO POR SESIÓN ATACANTE
│                         (Usuario "john" login desde IP externa 203.0.113.45)
│                         [SOC no lo sabe aún, no está marcado]
├─ 2024-01-15 08:30:00 ─ Atacante enumera shares (net view)
├─ 2024-01-15 09:00:00 ─ Atacante escala a SYSTEM vía CVE-2021-1732
│
└─ 2024-01-15 09:15:00 ─ PRIMER MOVIMIENTO SOSPECHOSO DETECTADO
                         (Ejecutable ransomware caído en C:\Windows\Temp)
                         [SOC detecta patrón de acceso a archivos inusual]

¿CAUSA RAÍZ = 2024-01-15 08:00:00? NO
¿CAUSA RAÍZ = 2024-01-15 08:15:00? NO
¿CAUSA RAÍZ = 2024-01-15 09:15:00? NO

CAUSA RAÍZ = El CVE-2021-1732 sin parche en Server-DB-01
             O credencial débil "john" estaba usando
             O método de acceso inicial (email phishing a john? RDP expuesto?)

Qué DEBES entender:
- Primer log ≠ causa raíz (es solo cuando atacante empezó a dejar rastros)
- Primer movimiento sospechoso ≠ causa raíz (es cuando lo detectamos)
- Causa raíz = primera acción que funcionó (acceso inicial o escalada de privilegios)
```

### Parte 3: Cadena de Custodia (CoC)

**Por qué importa:**
Si evidencia está comprometida, es inadmisible en corte Y no confiable para análisis técnico.

```
Manejo de Evidencia DFIR:

Paso 1: PRESERVACIÓN
├─ No toques la evidencia
├─ No "rm", no "del", sin modificaciones
├─ Si posible, aísla de red (air-gap)
└─ Documenta quién tocó qué, cuándo

Paso 2: RECOLECCIÓN
├─ Imagena disco (copia bit-a-bit)
├─ Hash disco original (SHA256)
├─ Hash de imagen (debe coincidir exactamente)
├─ Dump de memoria (mientras corre, si posible)
├─ Logs de red, logs de SIEM
├─ Headers de email, historial de navegador
└─ Documenta todo con timestamps

Paso 3: ANÁLISIS
├─ Trabaja en COPIA, nunca original
├─ Todas acciones registradas (output de herramienta, no solo "lo encontré")
├─ Mantén audit trail de quién accedió evidencia
└─ No modifiques nada

Paso 4: DOCUMENTACIÓN
├─ Verificación de hash (original vs copia coinciden)
├─ Forma de cadena de custodia firmada por todos manipuladores
├─ Notas detalladas de hallazgos
├─ Reproducibilidad (otro analyst podría repetir tus pasos)
└─ Compliance legal (si se necesita para corte)

EJEMPLO DE CoC:

Elemento de Evidencia: Disco Primario Server-DB-01
Hash Original: SHA256:a1b2c3d4e5f6...
Hash de Imagen: SHA256:a1b2c3d4e5f6... ✓ COINCIDE

Recolectado por: Jane Analyst (2024-01-15 14:00:00 UTC)
Sellado por: John Manager (2024-01-15 14:30:00 UTC)
Analizado por: Sarah DFIR (2024-01-16 09:00:00 UTC)
```

### Parte 4: Fuentes de Evidencia en DFIR

```
Dónde vive la evidencia:

┌─ EVIDENCIA DE DISCO
│  ├─ Sistema de archivos (NTFS, ext4, etc)
│  ├─ Master File Table (MFT) - contiene TODOS metadatos de archivo
│  ├─ Archivos borrados (aún en espacio no asignado)
│  ├─ Registry hives (configuración Windows + actividad usuario)
│  ├─ Logs (Windows Event Logs, application logs, web server logs)
│  ├─ Prefetch files (programas ejecutados + count + timestamps)
│  └─ Historial navegador, cache, cookies
│
├─ EVIDENCIA DE MEMORIA
│  ├─ RAM en momento compromiso (¡volátil!)
│  ├─ Procesos corriendo
│  ├─ Conexiones de red
│  ├─ Claves de encriptación en memoria
│  ├─ Comportamiento malware (antes que golpee disco)
│  └─ Hashes de contraseña
│
├─ EVIDENCIA DE RED
│  ├─ Logs firewall (conexiones bloqueadas/permitidas)
│  ├─ Firmas IDS/IPS
│  ├─ Logs de query DNS (¿qué dominios consultó atacante?)
│  ├─ Netflow/syslog (patrones de tráfico)
│  └─ Logs de proxy (tráfico web)
│
└─ EVIDENCIA DE APLICACIÓN
   ├─ Headers de email (¿de dónde vino phishing?)
   ├─ Logs web server (¿qué fue accedido?)
   ├─ Logs de base de datos (¿qué queries se ejecutaron?)
   ├─ Logs de conexión VPN (¿quién conectó desde dónde?)
   └─ Logs de autenticación (logins exitosos + fallidos)
```

---

## ⚙️ El Proceso de Investigación DFIR (Paso-a-Paso)

### Fase 1: Alcance del Incidente

**Pregunta a responder:** "¿Qué tan malo es realmente?"

```
Cuando se llama DFIR:

1. Entender qué IR ya hizo
   ├─ ¿Cuáles sistemas aislados?
   ├─ ¿Cuáles aún en vivo?
   ├─ ¿Se dumpeo memoria? ¿De qué sistemas?
   └─ ¿Qué datos disponibles?

2. Entrevista Incident Responder
   ├─ "¿Cuándo se activó alerta?"
   ├─ "¿Cuál fue lo observable que alertó?"
   ├─ "¿Qué hiciste para contener?"
   └─ "¿Qué ventanas de oportunidad quedan para recolectar evidencia?"

3. Identifica sistemas de interés
   ├─ Paciente cero (primer sistema comprometido)
   ├─ Ruta de movimiento lateral
   ├─ Endpoints de exfiltración de datos
   └─ Puntos de contacto Command & Control

4. Crea hipótesis inicial
   ├─ "Atacante usó phishing → acceso inicial → escalada → movimiento lateral"
   ├─ O "Exploit vulnerabilidad → reverse shell → persistence"
   └─ "Probaremos hipótesis con evidencia"
```

### Fase 2: Recolección de Evidencia

**Regla de Oro:** Recolecta de volátil a no-volátil

```
¿Por qué este orden?
- Memoria (volátil) = desaparece al reiniciar
- Disco (no-volátil) = sobrevive reinicio
- Red (semi-volátil) = logs rotan, se sobrescriben

Prioridad de recolección:

PASO 1: VOLÁTIL (haz primero, pronto desaparecerá)
├─ Dump de memoria (si sistema aún corre)
│  └─ Herramienta: DumpIt, Belkasoft RAM Capturer, WinPmem
├─ Conexiones de red actuales
│  └─ Comando: netstat -ano (Windows) o netstat -tulpn (Linux)
├─ Procesos corriendo
│  └─ Comando: tasklist /v (Windows) o ps aux (Linux)
└─ Archivos abiertos
   └─ Comando: Handle, lsof

PASO 2: NO-VOLÁTIL (después que volátil recolectado)
├─ Imagen de disco
│  └─ Herramienta: FTK Imager, Guymager, dd
├─ Event logs
│  └─ Ubicación: C:\Windows\System32\winevt\Logs (Windows)
├─ Registry hives
│  └─ Ubicación: C:\Users\*\NTUSER.DAT (Windows)
└─ Logs de aplicación
   └─ Ubicación: Varía (Apache, IIS, SQL Server, etc)

PASO 3: LOGS DE RED (ya recolectados por IR, verifica completud)
├─ Logs firewall (¿desde cuándo?)
├─ Alertas IDS/IPS (¿timeframe?)
├─ Logs DNS (¿disponibles?)
└─ Logs de proxy (¿período retención?)
```

### Fase 3: Construcción de Timeline

**Este es el CORAZÓN de DFIR.**

```
Construir timeline maestro significa:
- Recolectar timestamps de múltiples fuentes
- Fusionar en vista cronológica única
- Identificar patrones y secuencias

Fuentes de timeline:

1. TIMESTAMPS DE SISTEMA DE ARCHIVOS
   ├─ Creado (C) = creación archivo
   ├─ Accedido (A) = archivo abierto último (o leído)
   ├─ Modificado (M) = contenido archivo cambió
   └─ Cambiado (Ch) = metadatos cambiaron (permisos, etc)
   
   Herramienta: FTK Imager, Timeline Explorer, PowerShell

2. WINDOWS EVENT LOGS
   ├─ EventID 4624 = Logon (exitoso)
   ├─ EventID 4625 = Logon Failed
   ├─ EventID 4688 = Creación de proceso
   ├─ EventID 4720 = Cuenta de usuario creada
   ├─ EventID 4728 = Usuario agregado a grupo admin
   └─ EventID 4656 = Objeto (archivo/registry) accedido
   
   Herramienta: Event Log Explorer, Windows Event Viewer, Splunk

3. TIMESTAMPS REGISTRY
   ├─ LastWrite timestamps en llaves registry
   ├─ Listas MRU (Most Recently Used)
   ├─ Llaves Run/RunOnce (mecanismos persistencia)
   └─ Historial dispositivos USB (tracking dispositivos conectados)
   
   Herramienta: RegRipper, Registry Explorer, PowerShell

4. LOGS DE APLICACIÓN
   ├─ Logs web server (IIS, Apache)
   ├─ Logs base de datos (SQL Server, MySQL)
   ├─ Logs email (Exchange, Gmail)
   └─ Logs antivirus (¿cuándo detectó AV malware?)

5. PREFETCH & ARCHIVOS RECIENTES
   ├─ Prefetch = programas ejecutados + count ejecución + timestamps
   ├─ LnkFiles (accesos directos) = cuándo archivo fue accedido
   └─ RecentFileCache = qué fue abierto recientemente
   
   Herramienta: Registry Explorer, utilidades Nirsoft

CONSTRUYENDO EL TIMELINE:

Eventos crudos de múltiples fuentes:
├─ 2024-01-15 08:15:22 → EventID 4624: Usuario "john" login desde 203.0.113.45
├─ 2024-01-15 08:16:45 → Log netstat: conexión establecida a C2 (103.145.23.67:4444)
├─ 2024-01-15 08:17:00 → Archivo creado: C:\Temp\update.exe (payload atacante)
├─ 2024-01-15 08:30:15 → EventID 4688: powershell.exe ejecutado por "john"
├─ 2024-01-15 08:31:22 → Llave Registry modificada: HKLM\Software\...\Run (persistencia)
└─ 2024-01-15 09:15:00 → Ejecutable ransomware detectado en disco

Interpretación timeline:
1. Acceso inicial vía cuenta john (8:15)
2. Conexión C2 establecida (8:16)
3. Payload descargado (8:17)
4. Payload ejecutado vía PowerShell (8:30)
5. Persistencia establecida (8:31)
6. Empieza encriptación (9:15)

CAUSA RAÍZ: Cuenta john fue comprometida (¿contraseña débil? ¿phishing?)
```

### Fase 4: Análisis de Causa Raíz

**Pregunta:** "¿Cómo entró atacante?"

```
Trabajando hacia atrás desde timeline:

Ransomware detectado 09:15
    ↑
Payload ejecutado 08:30 (por john)
    ↑
Payload subido 08:17 (por john)
    ↑
Conexión C2 08:16 (por john)
    ↑
John login desde IP externa 08:15
    ↑
CAUSA RAÍZ: Credenciales john fueron comprometidas

Pregunta siguiente: ¿CÓMO se comprometieron credenciales john?
├─ Email phishing (ver logs email, historial navegador)
├─ Credential stuffing (ver password history john)
├─ Contraseña débil adivinada (política check)
├─ Keylogger instalado (search malware en máquina john)
└─ Credenciales VPN robadas (ver logs acceso VPN)

Ejemplo investigación:

HIPÓTESIS 1: Phishing
├─ Ver email john por mensaje sospechoso
├─ Ver descarga attachment
├─ Ver historial navegador por dominio malicioso
└─ Si encontrado → "john click link phishing, credenciales dadas sin querer"

HIPÓTESIS 2: Credential Stuffing (atacante probó contraseñas comunes)
├─ Ver Event Log 4625 (failed logins) antes 08:15
├─ Contar: 47 intentos fallidos en 10 minutos a cuenta "john"
├─ IP source: misma que 08:15 login exitoso (203.0.113.45)
└─ Conclusión → "Atacante brute-forced contraseña john exitosamente"

HIPÓTESIS 3: Contraseña Débil
├─ Política de contraseña check: "Mínimo 8 chars, sin complejidad requerida"
├─ Historial contraseña: john no cambió en 2 años
└─ Conclusión → "Contraseña débil/vieja vulnerable a adivinanza"
```

### Fase 5: Análisis de Impacto

**Pregunta:** "¿Qué hizo atacante? ¿Qué fue afectado?"

```
Respondiendo estas preguntas:

1. ¿Qué fue accedido?
   ├─ Logs de acceso archivo (¿qué directorios abiertos?)
   ├─ Logs base de datos (¿qué tablas consultadas?)
   ├─ Logs email (¿qué buzones accedidos?)
   └─ Conclusión: "Atacante accedió Financial_Data folder + Customer_Database"

2. ¿Qué fue robado?
   ├─ Archivos copiados a máquina atacante (logs red muestran exfiltración)
   ├─ Volumen datos (100GB? 1GB? 10MB?)
   ├─ Sensibilidad (PII? financiero? trade secrets?)
   └─ Conclusión: "Base datos clientes (5GB, contiene SSN/Tarjeta Crédito) exfiltrada"

3. ¿Qué fue modificado?
   ├─ Ransomware encriptado (¿cuántos archivos?)
   ├─ Backdoors instalados (¿dónde? ¿cuáles?)
   ├─ Mecanismos persistencia (edits registry? scheduled tasks?)
   └─ Conclusión: "47,230 archivos encriptados. Backdoor en System32. Persistencia en Run key"

4. ¿Qué sistemas fueron afectados?
   ├─ Paciente cero: Server-DB-01
   ├─ Movimiento lateral a: Server-APP-02, Server-APP-03, Workstation-john
   └─ Conclusión: "4 sistemas comprometidos en 45 minutos"

5. ¿Por cuánto tiempo estuvo presente atacante?
   ├─ Primer log: 08:15
   ├─ Último contacto C2: 09:15
   ├─ Pero ¿hay persistencia? (backdoor activo?)
   └─ Conclusión: "Intrusión activa 1 hora. Backdoor desconocido si aún activo"
```

### Fase 6: Generación de Reporte

**Lo que reporte DEBE contener:**

```
ESTRUCTURA REPORTE INVESTIGACIÓN DFIR

1. RESUMEN EJECUTIVO
   ├─ Qué pasó en 2 párrafos
   ├─ Resumen impacto (sistemas, datos, duración)
   └─ Recomendaciones (prevenir, detectar, responder)

2. TIMELINE DE EVENTOS
   ├─ Lista cronológica con timestamps
   ├─ Fuente evidencia para cada evento
   ├─ Interpretación de cada evento
   └─ Representación visual (Gantt chart o diagrama texto)

3. ANÁLISIS CAUSA RAÍZ
   ├─ Método acceso inicial (phishing? exploit? brute force?)
   ├─ Evidencia apoyo conclusión
   ├─ ¿Por qué este método funcionó? (vulnerabilidad, debilidad, oversight?)
   └─ Lecciones aprendidas

4. CADENA DE ATAQUE (MAPPING MITRE ATT&CK)
   ├─ Acceso Inicial (T1566 = Phishing)
   ├─ Acceso a Credenciales (T1110 = Brute Force)
   ├─ Persistencia (T1547.001 = Registry Run Key)
   ├─ Escalada Privilegios (T1548 = CVE-2021-1732)
   ├─ Evasión Defensa (T1036 = Masquerading)
   ├─ Movimiento Lateral (T1570 = Lateral Tool Transfer)
   ├─ Exfiltración (T1041 = Exfiltration Over C2)
   └─ Impacto (T1486 = Data Encrypted for Impact)

5. EVIDENCIA RECOLECTADA
   ├─ Dump memoria (tamaño, hash, hallazgos análisis)
   ├─ Imagen disco (tamaño, hash, hallazgos análisis)
   ├─ Logs recolectados (¿cuáles? ¿rango fechas? ¿gaps?)
   └─ Cadena custodia (firmada & verificada)

6. HALLAZGOS FORENSES
   ├─ Archivos ejecutables encontrados (hashes malware, firmas YARA)
   ├─ Modificaciones registry (mecanismos persistencia)
   ├─ Análisis sistema archivos (archivos borrados recuperados, timestamps)
   ├─ Análisis memoria (código inyectado, malware en RAM)
   └─ Artefactos red (IPs C2, queries DNS, patrones exfiltración)

7. RESUMEN IMPACTO
   ├─ Sistemas comprometidos (lista)
   ├─ Datos afectados (tipos, volumen, sensibilidad)
   ├─ Servicios disrupted (¿por cuánto tiempo?)
   ├─ Tiempo recuperación (¿cuánto para erradicar + restaurar?)
   └─ Pérdidas estimadas (si relevante)

8. RECOMENDACIONES
   ├─ INMEDIATO (para la hemorragia):
   │  └─ "Parchear CVE-2021-1732 en todos sistemas Windows"
   ├─ CORTO PLAZO (prevenir recurrencia):
   │  ├─ "Implementar MFA en todas cuentas usuario"
   │  ├─ "Desplegar application whitelisting"
   │  └─ "Segmentar red para aislar sistemas críticos"
   ├─ MEDIANO PLAZO (mejorar detección):
   │  ├─ "Implementar nueva regla detección para conexiones C2 anómalas"
   │  └─ "Desplegar EDR (Endpoint Detection & Response)"
   └─ LARGO PLAZO (mejora holística):
      ├─ "Entrenamiento seguridad awareness (simulación phishing)"
      ├─ "Implementar Zero Trust Architecture"
      └─ "Ejercicios tabletop trimestral"

9. APÉNDICES
   ├─ Datos timeline crudos (sorteable)
   ├─ Reporte análisis malware
   ├─ Dumps registry
   ├─ Listados archivo
   └─ Documentación cadena custodia
```

---

## 🛠️ Herramientas DFIR Que DEBES Conocer

### Herramientas Esenciales por Categoría

**IMAGING DE DISCO & HASHING:**
```
FTK Imager
├─ Windows, Mac, Linux
├─ Crear imágenes forenses (E01, dd, AFF formatos)
├─ Verificación hash (MD5, SHA1, SHA256)
├─ Recuperación archivo de espacio no asignado
└─ GRATIS

Guymager
├─ Basado Linux
├─ Imaging más rápido que FTK en Linux
├─ Soporte encriptación AES
└─ GRATIS

dd (línea comandos)
├─ Linux/Mac
├─ if=/dev/sda of=/media/usb/disk.img (imagen disco completo)
├─ if=/dev/sda1 of=/media/usb/partition.img (partición única)
└─ GRATIS
```

**ANÁLISIS DE MEMORIA:**
```
Volatility 3
├─ Análisis memoria Windows, Mac, Linux
├─ Encontrar procesos corriendo, código inyectado, conexiones red
├─ Extraer malware de RAM
├─ pslist, pstree, netscan, yarascan (comandos clave)
└─ GRATIS

DumpIt / Belkasoft RAM Capturer
├─ Dumps memoria rápidos de sistemas corriendo
├─ Antes de apagar (preserva evidencia volátil!)
└─ GRATIS/Comercial
```

**CONSTRUCCIÓN DE TIMELINE:**
```
Plaso (log2timeline)
├─ Basado Linux
├─ Parsea eventos de imagen disco + logs
├─ Output a CSV/timeline
└─ GRATIS

Timeline Explorer (SANS)
├─ GUI Windows
├─ Importar/filtrar/buscar timelines
├─ Color-code eventos
└─ GRATIS

Autopsy
├─ Plataforma forense visual
├─ Análisis imagen disco
├─ Generación timeline
├─ Búsqueda por palabra clave
└─ GRATIS
```

**ANÁLISIS REGISTRY:**
```
RegRipper
├─ Análisis registry Windows
├─ Extraer Run keys, listas MRU, historial USB
├─ Plugins para cada tipo análisis
└─ GRATIS

Registry Explorer (Eric Zimmerman)
├─ Visor registry moderno
├─ Timeline desde registry LastWrite times
├─ Búsqueda en todos hives
└─ GRATIS
```

**ANÁLISIS DE LOGS:**
```
Windows Event Log Parser
├─ Parsear archivos .evtx offline
├─ Filtrar por EventID, timestamp, usuario
├─ Exportar a CSV
└─ GRATIS (built-in o powershell)

Splunk / ELK / GreyLog
├─ Agregar logs de múltiples fuentes
├─ Búsqueda + pivot
├─ Crear timelines desde timestamps log
└─ Comercial/GRATIS (ELK)
```

**EJEMPLO PRÁCTICO:**

```
Workflow investigación usando herramientas:

1. Imagena disco comprometido
   └─ FTK Imager: Crear imagen E01, verificar hash

2. Extrae dump memoria (si aún corre)
   └─ DumpIt: Capturar RAM antes apagar

3. Monta imagen forense (solo lectura)
   └─ FTK Imager o loopback mount

4. Parsea eventos desde imagen
   └─ Plaso: Extraer todos timestamps a CSV

5. Analiza dump memoria
   └─ Volatility: volatility3 -f memory.dump windows.pslist
      (encontrar procesos sospechosos, código inyectado)

6. Extrae registry hives
   └─ RegRipper: rip.pl -r C:\Windows\System32\config\SYSTEM
      (extraer Run keys, tiempo último login, etc)

7. Construye timeline maestro
   └─ Timeline Explorer: Importar CSV desde Plaso, agregar manual entries de Volatility/RegRipper

8. Analiza timeline
   ├─ Buscar secuencias (login → payload → ejecución → persistencia)
   ├─ Cross-reference con logs red (¿atacante C2?)
   └─ Identificar causa raíz

9. Genera reporte
   └─ Documentar hallazgos con evidencia + hashes + screenshots
```

---

## 📊 Ejemplo Investigación Real: Paso-a-Paso

**Escenario: Ransomware en File Server**

```
TIMELINE DE INVESTIGACIÓN:

2024-01-15 14:00:00 (Investigación comienza)
├─ SOC detectó 10,000 archivos encriptados en FILE-SERVER-01
├─ Incident Response inmediatamente aisló servidor (red desconectada)
└─ DFIR analyst llamado para investigar

2024-01-15 14:30:00 (DFIR llega)
├─ Recolecta dump memoria (sistema aún corriendo, RAM tiene evidencia)
├─ Inicia imaging disco (FTK Imager)
└─ Documenta cadena custodia

2024-01-15 18:00:00 (Análisis comienza)
├─ Análisis Volatility de memoria:
│  └─ Encontrar proceso sospechoso: rundll32.exe (usualmente legítimo, pero corre desde Temp)
│  └─ Extrae código inyectado (parece C2 beacon)
│
├─ Análisis imagen disco:
│  ├─ Encontrar archivos en C:\Temp\update.exe (payload caído 2024-01-15 08:17)
│  ├─ Encontrar archivos en C:\Users\john\AppData\Roaming (área staging atacante)
│  └─ Análisis MFT muestra timestamps creación archivo
│
└─ Análisis registry:
   ├─ HKLM\Software\...\Run contiene "C:\Temp\update.exe"
   ├─ Timestamp LastWrite: 2024-01-15 08:31
   └─ Este es mecanismo persistencia

2024-01-15 22:00:00 (Construcción timeline)
├─ Timeline maestro construido:
│  ├─ 2024-01-15 08:15:22 → EventID 4624: john login desde 203.0.113.45 ✓
│  ├─ 2024-01-15 08:16:45 → Red: conexión a 198.51.100.50:4444 (IP C2) ✓
│  ├─ 2024-01-15 08:17:00 → Archivo creado: update.exe ✓
│  ├─ 2024-01-15 08:30:15 → EventID 4688: rundll32.exe ejecutado (cargando update.exe) ✓
│  ├─ 2024-01-15 08:31:22 → Registry modificado: Run key agregado ✓
│  └─ 2024-01-15 09:15:00 → Encriptación comienza ✓

2024-01-16 08:00:00 (Análisis causa raíz)
├─ ¿Cómo cuenta john fue comprometida?
│  ├─ Ver email john → Encontrado email phishing con asunto "Urgent: Security Update"
│  ├─ Email desde attacker@phishsite.com (spoofed)
│  ├─ john clickeó link, entró credenciales
│  └─ CAUSA RAÍZ IDENTIFICADA: Ataque phishing
│
└─ ¿Movimiento lateral?
   ├─ Ver logs red → Cuenta john (ahora comprometida) accedió otras shares
   ├─ Encontrado: john accedió \FILE-SERVER-01\Financial_Data (inusual)
   ├─ john normalmente solo accede HR_Data
   └─ Movimiento lateral confirmado

2024-01-16 14:00:00 (Generación reporte)
├─ Reporte entregado:
│  ├─ Resumen Ejecutivo: "Ataque ransomware vía email phishing a john@company.com"
│  ├─ Timeline: 08:15 acceso inicial → 09:15 encriptación comienza (ventana 60 min)
│  ├─ Impacto: 47,230 archivos encriptados, carpeta Financial_Data accedida, 2.3 GB datos posiblemente exfiltrados
│  ├─ Causa Raíz: Email phishing, contraseña john débil (sin MFA), segmentación red insuficiente
│  │
│  └─ Recomendaciones:
│     ├─ INMEDIATO: Restaurar de backup (28 horas para restaurar todos datos)
│     ├─ CORTO: Implementar MFA en todas cuentas
│     ├─ MEDIANO: Desplegar EDR en todos endpoints
│     ├─ LARGO: Segmentación red (Financial_Data en VLAN aislada)
│     └─ Entrenamiento: Awareness phishing para todo staff (john fue vector)

SEGUIMIENTO DETECTION ENGINEERING:
├─ Nueva regla necesaria: Detectar "Cuenta usuario haciendo acceso archivo inusual fuera carpeta normal"
├─ john normalmente accede \\HR_DATA\*
├─ john accediendo \\FINANCIAL_DATA\* es anomalía
├─ Regla: EventID 5145 (acceso archivo) + john + /Financial_Data + NO en últimos 90 días
├─ Alertar en primera ocurrencia
└─ Esto hubiera podido prevenir 47,230 archivos encriptados
```

---

## ❌ Errores Comunes Que Hacen Analistas DFIR Junior

### Error #1: Modificar Evidencia

```
❌ MAL:
"Necesito ver qué hay en este archivo, déjame abrirlo con Notepad"
(Timestamps archivo cambian, contenido podría ser modificado por OS)

✅ BIEN:
"Usaré FTK Imager o HexDump para ver contenido archivo SIN modificarlo"
(Acceso solo-lectura, timestamps preservados)

Consecuencia: Evidencia modificada = inadmisible en corte, análisis no confiable
```

---

### Error #2: Confundir Secuencias Timeline

```
❌ MAL:
"Proceso X.exe ejecutó, entonces X.exe debe ser malware"
(Quizá X.exe es legítimo, malware inyectó en él)

✅ BIEN:
"Proceso X.exe ejecutó. ¿Creó archivos? ¿Conectó a red? ¿Modificó registry?
Correlacionaré todos eventos alrededor timestamp ejecución X.exe"
(Timeline multi-fuente dice la verdadera historia)

Consecuencia: Conclusión equivocada, falta malware actual, acusación falsa herramienta legítima
```

---

### Error #3: No Documentar Cadena de Custodia

```
❌ MAL:
"Recolecté imagen disco, pero no documenté quién la manejó, cuándo, o verifiqué hashes"
(En caso legal, evidencia es rechazada)

✅ BIEN:
"Imagen disco recolectada 2024-01-15 14:30 por Jane Analyst
Hash original: SHA256:a1b2c3d4e5f6...
Hash imagen: SHA256:a1b2c3d4e5f6... ✓ VERIFICADO
Sellado por: John Manager (2024-01-15 15:00)
Accedido por: Sarah DFIR (2024-01-16 09:00) para análisis"
(Legalmente defensible, reproducible)

Consecuencia: Evidencia eliminada de caso, caso desestimado, reputación dañada
```

---

### Error #4: Ignorar Volatilidad

```
❌ MAL:
"Sistema está comprometido. Déjame apagarlo para imagena disco"
(¡RAM se pierde para siempre! Malware en memoria = ¡desaparece!)

✅ BIEN:
"Sistema está comprometido. Prioridades rápido:
1. Dump memoria AHORA (mientras corre)
2. Recolecta datos volátiles (netstat, tasklist, archivos abiertos)
3. DESPUÉS imagena disco"
(Memoria frecuentemente revela malware que nunca golpea disco)

Consecuencia: Falta malware-en-RAM-solamente, investigación incompleta
```

---

## 🧪 Escenario de Práctica

**Escenario: Investigación Insider Threat**

```
Recibes ticket:
"Analista sospecha ALICE podría exfiltrar datos clientes.
Alice tiene acceso Customer_Database. Anoche, IT notó tráfico red inusual
desde máquina Alice (transferencia salida grande, 500MB+).
Laptop Alice ahora aislada. Investiga dentro 4 horas."

TU INVESTIGACIÓN (responde estas preguntas):

1. ALCANCE:
   - ¿Qué fuentes evidencia necesitas?
   - ¿Cuál es prioridad? (¿volátil primero?)
   - ¿Qué buscas?

2. RECOLECCIÓN:
   - Dump memoria (¿por qué es crítico en caso insider threat?)
   - Historial navegador (¿dónde envió Alice datos?)
   - Email (¿planeó? ¿Comunicaciones?)
   - Sistema archivos (¿preparó Alice datos? ¿Cuándo?)
   - Red (¿qué C2 contactó? ¿O cloud drive personal?)

3. TIMELINE:
   - ¿Cuándo comenzó actividad inusual?
   - ¿Fue spike repentino o gradual?
   - ¿Correlaciona con eventos conocidos (noticia despido? Enojado?)?

4. ANÁLISIS:
   - ¿Alice realmente exfiltró? ¿O es trabajo legítimo?
   - ¿Cuenta Alice fue comprometida? (ver failed logins antes spike)
   - ¿Tráfico red es realmente DESDE ella? (podría ser malware en máquina)

5. CAUSA RAÍZ:
   - ¿Es Alice realmente exfiltrando? ¿O es falso positivo?
   - ¿Cuánto tiempo ha estado sucediendo?
   - ¿Qué datos específicamente?

6. REPORTE:
   - Evidencia apoyo conclusiones
   - Timeline mostrando progresión
   - Recomendaciones (legal vs técnico)
```

**Hallazgos Esperados:**

```
Una hora dentro investigación:

Análisis timeline:
├─ 2024-01-14 17:00:00 → Alice envía email a cuenta Gmail personal
│  └─ Asunto: "Empezando trabajo nuevo en competitors.com próxima semana"
│
├─ 2024-01-14 18:30:00 → Alice sube 500MB a Google Drive
│  └─ Archivo: "Customer_Database_20240114.zip"
│  └─ Hash: SHA256:abc123... (¿malware? no, solo ZIP)
│
├─ 2024-01-14 19:00:00 → Alice borra archivos área staging
│  └─ Intenta ocultar evidencia
│
└─ 2024-01-14 19:30:00 → Alice logout, deja oficina

CONCLUSIÓN:
✓ Alice intencionalmente exfiltró datos
✓ Planea llevarlos a competitors.com
✓ No es caso malware, pero INSIDER THREAT
✓ Acción: Involucramiento Legal + HR (no solo incident response técnico)
```

---

## 🎯 Preguntas de Entrevista Que Podrían Hacerte

### Level 1 (Entry-Level)

**P1: "¿Cuál es diferencia entre Digital Forensics e Incident Response?"**
> "Digital Forensics es investigación post-ataque: '¿Qué pasó exactamente?' Incident Response es durante/después: 'Detén ataque, contén daño, recupera sistemas.' IR es decisiones tiempo-real, Forensics es análisis detallado después."

**P2: "¿Por qué preservamos cadena custodia?"**
> "Para que evidencia sea legalmente defensible, técnicamente confiable, reproducible. Si cadena rota, evidencia inadmisible corte y puede estar comprometida."

**P3: "¿Qué es 'causa raíz' vs 'primer log sospechoso'?"**
> "Causa raíz es primera acción que funcionó (acceso inicial o exploit). Primer log sospechoso es cuándo detectamos algo estaba mal. Son tiempos diferentes."

### Level 2 (Mid-Level)

**P1: "Camina me a través investigación insider threat sospechado."**
> "1) Alcance: ¿Qué datos podrían acceder? ¿Qué sistemas? 2) Recolecta: Memoria, historial navegador, email, sistema archivos. 3) Timeline: ¿Cuándo comenzó actividad inusual? 4) Analiza: ¿Fue copia manual o malware? 5) Determina: ¿Es realmente insider threat o cuenta comprometida? 6) Reporte: Conclusión basada-evidencia."

**P2: "Encuentras archivo modificado 3 AM con extensión inusual (.exe en carpeta temp). Definitivamente malware. ¿Cómo procedes?"**
> "ENFOQUE MAL: Ejecutarlo para ver qué hace. ENFOQUE CORRECTO: 1) No lo toques. 2) Imagena disco (preserva evidencia). 3) Hash archivo. 4) Analiza en sandbox o usando Volatility si en memoria. 5) Rastrea movimiento lateral: ¿qué hizo malware? ¿De dónde vino?"

**P3: "Imagen disco muestra 47,000 archivos borrados en espacio no asignado. ¿Cómo recuperas analizas?"**
> "Usa FTK Imager similar para recuperar archivos borrados de espacio no asignado. Hash cada uno. Ejecuta firmas YARA o chequea hashes malware conocidos. Timeline ellos (¿borrados cuándo?). Podría mostrar área staging atacante o datos que fueron robados luego borrados."

### Level 3 (Senior / Experimentado)

**P1: "Investigas breach. Atacante fue 6 meses. Logs solo 30 días atrás. ¿Cómo determinas punto acceso inicial?"**
> "1) Analiza mecanismos persistencia (registry, scheduled tasks, WMI) — ¿cuándo creados? 2) Busca cuentas creadas últimos 6 meses (cuentas backdoor). 3) Encuentra evidencia movimiento lateral más temprana. 4) Correlaciona con advisories vendor (¿qué CVE probablemente explotado cuándo?). 5) Entrevista: ¿cuándo sistemas empezaron actuar extraño? 6) Examina logs Change Management (¿cambio inusual 6 meses atrás?). No puedes ver todos logs, pero puedes inferir timeline de otra evidencia."

**P2: "Malware ejecutándose en memoria nunca golpeó disco. ¿Cómo analizas esto pruebas qué hizo?"**
> "1) Dump memoria es oro. Usa Volatility: pslist (encuentra proceso), vaddump (extrae address space), disasm (desensambla código inyectado). 2) Analiza API calls malware hizo (¿qué funciones Win32 llamó?). 3) Busca strings en memoria (dominio C2, rutas archivo, registry keys). 4) Cross-reference con logs IDS/IPS (¿qué IPs conectó proceso?). 5) Sandbox testing: si recuperaste código, detona en ambiente aislado confirmar comportamiento. El dump memoria ES la evidencia."

**P3: "Tienes dos hipótesis: (A) Atacante usó CVE conocido, O (B) Atacante tuvo acceso insider. Timeline podría apoyo ambos. ¿Cómo diferencias?"**
> "1) Si explotación CVE: verías creación proceso sospechosa, intento escalada privilegios, relaciones parent-child inusuales. 2) Si insider: verías login normal, patrones acceso archivo normales, pero timing inusual o archivos inusuales accedidos. 3) Chequea: ¿fue parche para CVE? Si sistema parcheado, descarta CVE. 4) Chequea failed logins antes breach: si 1000s failed logins (brute force), insider improbable. 5) Entrevista: ¿comportamiento atacante match algún employee actual/anterior? 6) Motivación: ¿qué datos robados? ¿Alínea con nivel acceso insider? Construyes probabilidad, presentas evidencia ambos, leadership decide."

---

## 🔗 Cómo DFIR Conecta Todo

```
Detection Engineering
├─ Encuentra ataque (alerta SIEM)
└─ DFIR lo analiza, encuentra causa raíz
   └─ Detection Engineer escribe regla atrapar ataques similares futuro

Incident Response
├─ Decide tiempo-real (¿aislar? ¿no aún?)
└─ DFIR hace investigación detallada
   └─ IR usa hallazgos DFIR para decisiones contención

Threat Hunting
├─ Busca patrones sospechosos
└─ Si encuentra algo, escala a DFIR
   └─ DFIR confirma es ataque real o falso positivo

Network Security
├─ Logs todo tráfico
└─ DFIR analiza logs firewall/IDS conexiones C2
   └─ Usado rastrear ruta exfiltración atacante

Endpoint Security (EDR/AV)
├─ Bloquea malware, registra comportamiento
└─ DFIR analiza logs EDR comportamiento ataque timeline
   └─ Combinado con disco/memoria forensics = panorama completo

Compliance (SOC 2, HIPAA, PCI-DSS)
├─ Requiere investigación incidente
└─ DFIR proporciona prueba documentada
   └─ "Aquí está timeline, cadena custodia, evidencia"
```

---

## 💾 TL;DR - Referencia Rápida

### El Proceso DFIR
```
1. ALCANCE → ¿Qué pasó? ¿Qué tan malo?
2. RECOLECCIÓN → Volátil (memoria) → No-volátil (disco) → Red
3. TIMELINE → Fusiona todos timestamps, encuentra secuencias
4. ANÁLISIS → Causa raíz, impacto, cadena ataque
5. REPORTE → Conclusiones basadas-evidencia + recomendaciones
6. PREVENCIÓN → Detection Engineering escribe nuevas reglas
```

### Métricas Clave
```
Cadena Custodia: Mantenida ✓
Hash Evidencia Verificado: ✓
Timeline Reproducible: ✓
Secuencia Ataque Clara: ✓
Causa Raíz Identificada: ✓
→ Investigación exitosa
```

### Cheat Sheet Herramientas
```
Imaging: FTK Imager, dd, Guymager
Análisis Memoria: Volatility3, DumpIt
Timeline: Plaso (log2timeline), Timeline Explorer
Registry: RegRipper, Registry Explorer
Logs: Windows Event Viewer, Splunk, ELK
```

### Errores Comunes
```
1. ❌ Modificar evidencia → Trabaja en copias solamente
2. ❌ Perder datos volátiles → Dump memoria PRIMERO
3. ❌ Saltar cadena custodia → Documenta TODO
4. ❌ Confundir secuencias → Construye timeline multi-fuente
```

---

## 📌 Realidad Mundo Real

### Caso: Ransomware Wannacry (2017)

```
Investigación después ataque:

Reconstrucción timeline:
├─ 2017-05-12 14:32:00 → Sistema Windows vulnerable infectado vía exploit EternalBlue
├─ 2017-05-12 14:35:00 → Payload ransomware ejecuta, comienza encriptación
└─ 2017-05-12 15:00:00 → Compañía paga ransom (después comienza investigación)

Hallazgos DFIR:
├─ Causa raíz: Windows sin parche (parche MS17-010 disponible 2 meses!)
├─ Persistencia: Ninguna encontrada (ransomware no instaló backdoor)
├─ Movimiento lateral: Infectó múltiples sistemas en misma red (pobre segmentación)
├─ Impacto: 200,000+ archivos encriptados en 5 servidores

Lecciones aprendidas:
├─ Patch management es crítico (esto era prevenible)
├─ Segmentación red necesaria (ransomware no debería cruzar límites)
└─ Backups mejor que ransom (tiempo restauración 72 horas vs ransom + incertidumbre desencriptación)
```

### Caso: Insider Threat (SOC Real)

```
Escenario: Employee despedida, sospecha robo datos

DFIR Timeline:
├─ 2024-01-10 → Employee notificado despido
├─ 2024-01-10 15:00:00 → Employee accede carpeta "Confidential_Formulas" (inusual)
├─ 2024-01-10 16:30:00 → Gran transferencia archivo a OneDrive personal (500MB)
├─ 2024-01-10 18:00:00 → Employee logout, deja oficina
└─ 2024-01-11 → Investigación comienza

Evidencia Forense:
├─ Historial navegador muestra login OneDrive (cuenta personal)
├─ Archivo staging en %TEMP% (compresión antes upload)
├─ Intentos borrado (Shift+Delete, recuperación no posible)
└─ Email a competidor (encontrado Outlook cache)

Conclusión:
✓ Insider threat confirmado
✓ Datos exfiltrados a cuenta personal (podría acceder desde cualquier lado)
✓ Prueba para acción legal

Acciones:
├─ Legal: Demanda contra employee
├─ Técnico: Cambiar credenciales, auditar cloud storage
└─ Proceso: Implementar USB blocking, DLP (Data Loss Prevention) en uploads cloud
```

---

## 📚 Próximos Pasos

1. **Aprende Volatility** → Forensics memoria (muchos ataques dejan rastros RAM primero)
2. **Practica construcción timeline** → Cómodo fusionando eventos múltiples fuentes
3. **Configura lab forense** → Laptop vieja, imagen Windows, herramientas instaladas
4. **Haz ejercicios tabletop** → Simula breach, practica workflow DFIR
5. **Estudia MITRE ATT&CK** → Mapea hallazgos forenses a framework ataque

---

## 📌 Recuerda

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  DFIR ≠ Solo mirar logs                                   │
│  DFIR = Investigación estructurada: Recolecta → Timeline →│
│         Analiza → Reporta → Previene                      │
│                                                            │
│  Evidencia sin cadena custodia = Inútil                   │
│  Timeline sin correlación = Significado                   │
│  Reporte sin recomendaciones = Incompleto                 │
│                                                            │
│  Éxito es cuándo puedes decir con certeza:               │
│  "Esto es EXACTAMENTE lo que pasó, cuándo, y por qué"    │
│  Y puedes PROBARLO con evidencia                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

**Última actualización**: 2024
**Versión**: 1.0
**Para**: Blue Team Students - TripleTen Bootcamp
