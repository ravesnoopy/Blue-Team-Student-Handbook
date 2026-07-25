# 🪟 Windows en Operaciones Blue Team: Guía de Seguridad Completa

---

## 📖 ¿Qué Es Esto?

Windows es el sistema operativo más ampliamente desplegado globalmente, ejecutándose en ~75% de los endpoints empresariales. En un Centro de Operaciones de Seguridad (SOC), Windows es **la superficie de ataque primaria** y la fuente principal de registros de seguridad. Entender Windows desde la perspectiva del blue team significa saber cómo detectar actividad maliciosa, investigar compromisos y endurecerse contra ataques. No es solo sobre el SO en sí—es sobre los **event logs, registro, procesos y actividad de red** que revelan qué está sucediendo en una máquina comprometida.

---

## 🎯 ¿Por Qué Un Profesional de SOC/Blue Team Necesita Esto?

### **Escenarios Reales del Trabajo**

1. **Detección de Compromiso de Endpoint**
   - Analista en SOC recibe alerta: intentos de login fallidos por RDP (Event 4625) suben de 5 a 150 en 10 minutos
   - Tu trabajo: investigar si fue intento de brecha o problema de configuración
   - Sin conocimiento de Windows: pierdes el ataque

2. **Investigación de Movimiento Lateral**
   - Atacante compromete una máquina Windows, crea cuenta backdoor (Event 4720)
   - Usa esa cuenta para acceder a recursos compartidos (Event 5140) y moverse a otros sistemas
   - Tu trabajo: rastrear el movimiento, detenerlo, encontrar qué se robó
   - Sin conocimiento de Windows: no sabes dónde buscar

3. **Detección de Persistencia**
   - Atacante instala un servicio (Event 7045) que ejecuta ransomware cada mañana a las 3 AM
   - O crea una tarea programada (Event 4698) escondida en Windows Task Scheduler
   - Tu trabajo: encontrarlo antes de que se ejecute
   - Sin conocimiento de Windows: pierdes la fase de configuración

4. **Respuesta a Ataque PowerShell**
   - Adversario usa PowerShell (Event 4688) para codificar y descargar malware en memoria
   - Nunca toca disco, así que antivirus tradicional no lo detecta
   - Tu trabajo: detectar malware residente en memoria antes de ejecutarse
   - Sin conocimiento de Windows: no sabes que PowerShell es el vector de amenaza

### **Preguntas de Entrevista que Enfrentarás**

- *"Caminemos a través de cómo investigarías un Event ID 4688 sospechoso en nuestro SIEM."*
- *"¿Cuál es la diferencia entre Event Logs y el Registro de Windows, y por qué importa cada uno?"*
- *"Describe un ataque de movimiento lateral en Windows y cómo lo detectarías."*
- *"¿Por qué PowerShell es más peligroso que cmd.exe?"*
- *"¿Cómo detectarías una tarea programada creada por malware?"*
- *"¿Qué Event IDs indican un compromiso de cuenta?"*

---

## 🔍 El Concepto Desglosado

### **Parte 1: Las Tres Capas de Windows Que Debes Conocer**

#### **Capa 1: Event Logs (Qué Sucedió)**
**Definición:** Registros estructurados de eventos del sistema generados en tiempo real cuando acciones ocurren en Windows.

**Ubicación:** Visor de Eventos → Registros de Windows → Seguridad, Aplicación, Sistema

**Qué muestran:**
- Inicios/cierres de sesión de usuario (Event 4624, 4647)
- Ejecución de procesos (Event 4688)
- Creación de cuentas (Event 4720)
- Instalación de servicios (Event 7045)
- Conexiones de red (Event 5156)
- Acceso a archivos (Event 5140)

**Por qué importa para SOC:**
- Estos logs se reenvían a tu SIEM en tiempo real
- Son la fuente de datos primaria para detección de amenazas
- También es lo primero que atacantes intentan eliminar (Event 1102 = encubrimiento)

**Ejemplo:** Usuario "admin" inicia sesión a las 3 AM desde IP desconocida:
```
Event 4624: Inicio de Sesión Exitoso
Usuario: admin
Hora de Inicio: 2024-01-15 03:22:15
IP Origen: 192.168.1.50
Tipo de Inicio: 3 (Red)
```

#### **Capa 2: Registro (Cómo Está Configurado)**
**Definición:** Base de datos jerárquica que almacena datos de configuración para el SO y aplicaciones.

**Ubicaciones clave:**
- `HKEY_LOCAL_MACHINE\Software` = Configuraciones de software de todo el sistema
- `HKEY_LOCAL_MACHINE\System` = Drivers de dispositivo, servicios
- `HKEY_CURRENT_USER\Software` = Configuraciones específicas del usuario
- `HKEY_LOCAL_MACHINE\SAM` = Hashes de contraseña de usuario local (solo offline)

**Qué atacantes esconden aquí:**
- Mecanismos de persistencia de malware
- Credenciales robadas cacheadas de intentos de login fallidos
- Streams de datos alternos
- Características de seguridad deshabilitadas (Windows Defender, Firewall)

**Por qué importa:**
- Revela técnicas del adversario antes de ejecutarse
- Muestra mecanismos de persistencia a largo plazo
- Indica endurecimiento del sistema o debilitamiento deliberado

**Ejemplo:** Atacante desactiva Windows Defender:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender
DisableAntiSpyware = 1
```

#### **Capa 3: Procesos & Memoria (Qué Está Ejecutándose)**
**Definición:** Los programas activos que se ejecutan en RAM en cualquier momento dado.

**Herramientas para verlos:**
- Administrador de Tareas (GUI)
- PowerShell: `Get-Process`
- Línea de comandos: `tasklist`
- SIEM: Event 4688 (Creación de Proceso)

**Por qué importa:**
- Malware a menudo se ejecuta solo en memoria (nunca toca disco)
- Event 4688 incluye la **línea de comandos completa** (argumentos, payloads codificados)
- Ataques residentes en memoria son los más peligrosos porque antivirus no puede verlos

**Ejemplo:** PowerShell ejecutando malware codificado:
```
Event 4688: Creación de Proceso
Image: C:\Windows\System32\powershell.exe
CommandLine: "powershell.exe -enc JABjAG8AbQA="
ParentImage: explorer.exe
```

---

### **Parte 2: Por Qué Importa la Seguridad de Windows en un SOC**

**Windows es el objetivo de ataque porque:**

1. **Está en todas partes** (~75% de endpoints empresariales)
2. **Es complejo** (cientos de partes móviles, muchas malas configuraciones)
3. **Ejecuta aplicaciones empresariales** (donde viven las joyas de la corona)
4. **Tiene acceso remoto incorporado** (RDP, WinRM) que atacantes abusan
5. **Confía en otras máquinas Windows por defecto** (Active Directory = incendio forestal si una quema)

**La perspectiva del atacante:**
- Compromete 1 endpoint Windows
- Lo usa para reconocer la red (reconocimiento)
- Roba credenciales de esa máquina
- Usa esas credenciales para acceder a otros sistemas
- Una vez en AD, compromete todo lo conectado

**La perspectiva del blue team:**
- Detecta el compromiso inicial en Event 4625 (pico de inicios fallidos por RDP)
- Detiene movimiento lateral en Event 5140 (acceso sospechoso a archivos)
- Previene persistencia en Event 7045 (instalación maliciosa de servicio)
- Contiene el daño aislando la máquina

---

### **Parte 3: Los 10 Event IDs Que DEBES Memorizar**

| Event ID | Nombre | Fase del Ataque | Objetivo de Detección | Ejemplo |
|----------|--------|-----------------|----------------------|---------|
| **4688** | Creación de Proceso | Ejecución | Lanzamiento de malware/herramienta | powershell.exe con argumentos sospechosos |
| **4720** | Cuenta de Usuario Creada | Persistencia | Creación de cuenta backdoor | "admin_backup" creada fuera de ventana de cambios |
| **4625** | Fallo de Inicio de Sesión | Acceso Inicial | RDP fuerza bruta, ataque de diccionario | 200 intentos fallidos en 5 minutos |
| **4624** | Inicio de Sesión Exitoso | Indicador de Éxito | Inicio de sesión anómalo de usuario/hora/ubicación | Usuario inicia sesión a las 3 AM desde IP desconocida |
| **7045** | Servicio Instalado | Persistencia | Creación de servicio malicioso | Servicio "WindowsUpdate" instalado con binario en %TEMP% |
| **4724** | Reseteo de Contraseña de Cuenta | Robo de Credenciales | Preparación de movimiento lateral | Admin reseteó contraseña de usuario, luego la usa para acceder a recurso compartido |
| **5140** | Acceso a Recurso Compartido de Red | Movimiento Lateral | Exfiltración de datos, reconocimiento | Usuario accede a recurso compartido sensible a hora inusual |
| **4698** | Tarea Programada Creada | Persistencia | Ejecución programada de malware | Tarea ejecuta PowerShell a las 3 AM diariamente |
| **1102** | Registro de Auditoría Borrado | Encubrimiento | Destrucción de evidencia del adversario | Registro de seguridad borrado (ESCALACIÓN INMEDIATA) |
| **4769** | Ticket Kerberos Solicitado | Movimiento Lateral | Kerberoasting, escalación de privilegios | Usuario solicita ticket para servicio de admin de dominio |

---

## ⚙️ Lo Que DEBES Memorizar

### **El Sistema de Memoria de Event ID**

**4688 - Creación de Proceso: "4-6-88" = Cuatro procesos (árbol padre-hijo-nieto-bisnieto)**
- Padre: explorer.exe
- Hijo: powershell.exe
- Nieto: cmd.exe
- Bisnieto: malware.exe

**Recuerda:** Cada proceso en Event 4688 muestra padre e hijo → construye el árbol de ejecución

---

**4720 - Usuario Creado: "4-7-20" = 7 letras en "USUARIO" = creación de usuario (mnemotécnica en español)**
- Cuentas backdoor usualmente tienen nombres como: admin_backup, svc_restore, temp_admin
- Creadas fuera de horario comercial → bandera roja
- Usadas inmediatamente → doble bandera roja

---

**4625 - Fallo de Inicio de Sesión: "4-6-25" = 46 intentos repetidos = fuerza bruta (6 intentos, repetidos x25)**
- Fallo único = normal
- 10 fallos en 1 minuto = intento de fuerza bruta
- 200 fallos en 5 minutos = ATAQUE EN PROGRESO

---

**7045 - Instalación de Servicio: "7-0-45" = Windows tiene ~45 servicios incorporados, uno nuevo = sospechoso**
- Los servicios se ejecutan a nivel SYSTEM (máximo privilegio)
- Se inician automáticamente al arranque
- Perfecto para persistencia

---

**1102 - Registro Borrado: "1-1-0-2" = Primero y evento más crítico**
- Si ves esto: asume brecha
- El atacante SIEMPRE borra los registros para esconder evidencia
- Este evento mismo demuestra que estuvieron ahí

---

**4769 - Ticket Kerberos: "4-7-69" = Protocolo Kerberos (puerto 88 + 69 = tickets)**
- Normal: usuario solicita ticket para impresora
- Sospechoso: usuario solicita ticket para controlador de dominio o servicio de admin de dominio
- Así es como atacantes se mueven lateralmente

---

### **Términos Clave que DEBES Conocer**

| Término | Definición | Relevancia para SOC |
|---------|-----------|---------------------|
| **RDP** | Protocolo de Escritorio Remoto—permite login remoto a Windows | Vector de ataque primario para acceso inicial |
| **WinRM** | Administración Remota de Windows—remoting de PowerShell | Menos visible que RDP, más difícil de detectar |
| **Active Directory (AD)** | Directorio central de todos usuarios/computadoras/permisos en dominio | Compromiso de AD = compromiso de toda la organización |
| **NTFS** | Sistema de Archivos NT—permisos (quién puede leer/escribir archivos) | Muestra quién puede acceder a qué en disco |
| **Share Permissions** | Derechos de acceso de red (separado de NTFS) | Los atacantes usan para mover datos a otros sistemas |
| **Kerberos** | Protocolo de autenticación usado en redes Windows | Los atacantes falsifican tickets para movimiento lateral |
| **PowerShell** | Lenguaje de scripting moderno en Windows (se ejecuta en memoria) | Herramienta primaria para post-explotación |
| **Task Scheduler** | Ejecución de tareas automatizadas en tiempos especificados | Usado por atacantes para persistencia |
| **Services** | Procesos de fondo que se ejecutan al inicio del sistema | Usado por atacantes para persistencia |

---

## 📚 Lo Que DEBES Entender

### **Entendimiento 1: La Cadena de Ataque en Windows**

Cada compromiso sigue este patrón:

```
1. ACCESO INICIAL
   └─ Event 4625: Fuerza bruta de RDP
      O Event 4688: Link de phishing abre malware

2. PERSISTENCIA
   └─ Event 4720: Crear cuenta backdoor
   └─ Event 7045: Instalar servicio malicioso
   └─ Event 4698: Crear tarea programada
   └─ Event 4624: Cuenta backdoor inicia sesión exitosamente

3. ROBO DE CREDENCIALES
   └─ Atacante ejecuta dump de memoria lsass.exe
   └─ Registro muestra credenciales en caché
   └─ Event 4720 muestra nueva cuenta con contraseña robada

4. MOVIMIENTO LATERAL
   └─ Event 4625: Intentos fallidos en otras máquinas (password spray)
   └─ Event 4624: Inicio de sesión exitoso en otras máquinas
   └─ Event 5140: Acceso a recursos compartidos de red para robar datos

5. ENCUBRIMIENTO
   └─ Event 1102: Atacante borra registro de seguridad
   └─ Intentos de eliminación de Registro
```

**Tu trabajo:** Interrumpir esta cadena lo antes posible (idealmente en paso 1 o 2)

---

### **Entendimiento 2: Por Qué PowerShell es Más Peligroso que cmd.exe**

| Característica | cmd.exe | PowerShell |
|----------------|---------|-----------|
| **Ejecución** | Basada en disco | Basada en memoria |
| **Detectabilidad** | Fácil (escribe en disco) | Difícil (solo en RAM) |
| **Capacidades** | Limitadas | Acceso completo a .NET |
| **Ofuscación** | Mínima | Puede codificar comandos enteros |
| **Remoting** | No | Sí (WinRM) |
| **Ejemplos en naturaleza** | Malware antiguo | Ransomware moderno, herramientas APT |

**Ejemplo de ataque PowerShell (imposible con cmd):**
```powershell
# Comando codificado (base64) - parece galimatías
powershell.exe -enc JABjAG8AbQA=

# Se decodifica a: $com=....(descargar malware)
# Atacante: malware nunca toca disco
# Analista: ve Event 4688 con argumento codificado
```

---

### **Entendimiento 3: Active Directory es Tu Joya de la Corona**

Piensa en Active Directory como la bóveda de un banco:

- **Usuario normal** = Cliente (puede acceder a su propia cuenta)
- **Admin de Dominio** = Gerente del banco (puede acceder a todo)
- **Controlador de Dominio** = La bóveda misma (almacena todas credenciales, todos permisos)

**Si atacante compromete Admin de Dominio o Controlador de Dominio:**
- Puede resetear contraseñas para CUALQUIER usuario (Event 4724)
- Puede añadirse a CUALQUIER grupo (Event 4728)
- Puede acceder a CUALQUIER recurso compartido de archivos (Event 5140)
- **Resultado:** Organización completa comprometida en minutos

**Ejemplo de ataque (Kerberoasting):**
```
1. Atacante encuentra usuario "service_account"
2. Ejecuta: Invoke-Kerberoast (herramienta PowerShell)
3. Solicita ticket Kerberos para service_account (Event 4769)
4. Offline, quiebra el ticket (toma horas, no en tiempo real)
5. Obtiene contraseña en texto claro para service_account
6. La usa para acceder a sistemas de alto privilegio
```

**Tu detección:** Alertar en Event 4769 para solicitudes inusuales de tickets de servicio

---

### **Entendimiento 4: El Registro Revela Secretos**

Los atacantes usan el Registro para:

1. **Desactiva características de seguridad**
   ```
   DisableAntiSpyware = 1
   DisableRealtimeMonitoring = 1
   ```

2. **Añade hooks de persistencia**
   ```
   Run key: malware.exe (se ejecuta en cada arranque)
   Runonce key: backdoor.ps1 (se ejecuta una vez)
   ```

3. **Almacena datos robados**
   ```
   Lista de archivos recientes
   Credenciales de red cacheadas
   Historial del navegador
   ```

**Perspectiva SOC:** La inspección del Registro es trabajo forense (ocurre DESPUÉS del descubrimiento de brecha vía Event Logs)

---

## 🚨 Aplicación Práctica: El Escenario de Ataque Real

### **Escenario: Ataque de Ransomware en Endpoint Windows**

**Línea de tiempo de eventos:**

**23:15 - Acceso Inicial**
```
Event 4625 (Fallo de Inicio de Sesión)
Usuario: desconocido
IP Origen: 203.0.113.45
Intentos: 1 de muchos por venir
Razón: Intento de fuerza bruta vía RDP
```

**23:47 - Escalada de Fuerza Bruta**
```
Event 4625 (Fallo de Inicio de Sesión) - repetido 150 veces
IP Origen: 203.0.113.45
Intentos en 30 minutos: 150 (patrón de ataque obvio)
```

**00:05 - Brecha Exitosa**
```
Event 4624 (Inicio de Sesión Exitoso)
Usuario: admin (cuenta legítima)
IP Origen: 203.0.113.45 (externa, coincide con intentos fallidos)
Tipo de Inicio: 3 (Red/RDP)
→ ATACANTE AHORA EN SISTEMA
```

**00:12 - Reconocimiento**
```
Event 4688 (Creación de Proceso)
Image: C:\Windows\System32\cmd.exe
CommandLine: "cmd /c whoami"
Parent: explorer.exe
→ ATACANTE APRENDIENDO SOBRE EL SISTEMA
```

**00:15 - PowerShell para Ataque Avanzado**
```
Event 4688 (Creación de Proceso)
Image: C:\Windows\System32\powershell.exe
CommandLine: powershell.exe -enc JABsaXN0... [comando codificado]
Parent: cmd.exe
→ ATACANTE EJECUTANDO MALWARE CODIFICADO (evita antivirus)
```

**00:18 - Configuración de Persistencia**
```
Event 7045 (Servicio Instalado)
ServiceName: WindowsUpdate
ImagePath: C:\ProgramData\malware.exe
StartType: Auto
→ MALWARE SE EJECUTARÁ EN CADA ARRANQUE
```

**00:20 - Creación de Cuenta Backdoor**
```
Event 4720 (Cuenta de Usuario Creada)
NewAccountName: svc_backup
→ ATACANTE PUEDE INICIAR SESIÓN INCLUSO SI CUENTA ORIGINAL SE DESACTIVA
```

**00:25 - Comienza Exfiltración de Datos**
```
Event 5140 (Acceso a Recurso Compartido de Red)
ShareName: \\server\Finance
UserName: admin
AccessMask: Lectura (malware leyendo archivos para cifrar)
→ ATACANTE MAPEANDO QUÉ CIFRAR
```

**03:00 - Ransomware Se Ejecuta (Programado para horas fuera de oficina)**
```
Event 4688 (Creación de Proceso)
Image: C:\ProgramData\ransomware.exe
CommandLine: ransomware.exe /encrypt /key:stolen_key
Parent: services.exe
→ ARCHIVOS SIENDO CIFRADOS
→ NEGOCIO CAÍDO
```

**03:05 - Encubrimiento**
```
Event 1102 (Registro de Auditoría Borrado)
ClearedBy: svc_backup
→ ATACANTE DESTRUYENDO EVIDENCIA
```

---

### **Tu Respuesta como Analista de Blue Team**

| Hora | Acción | Prioridad |
|------|--------|-----------|
| 23:47 | Alerta en pico de 4625 (150 RDP fallidos en 30 min) | MEDIA - Bloquear IP |
| 00:05 | Alerta en 4624 desde misma IP que intentos fallidos | **ALTA** - Aislar máquina inmediatamente |
| 00:15 | Alerta en PowerShell con argumentos codificados | **CRÍTICA** - Matar proceso, investigar |
| 00:18 | Alerta en instalación de servicio 7045 (malware) | **CRÍTICA** - Eliminar servicio, quitar malware |
| 00:20 | Alerta en creación de cuenta 4720 nueva | **ALTA** - Desactivar cuenta, resetear contraseña de admin |
| 00:25 | Alerta en acceso inusual a recurso compartido 5140 | **ALTA** - Investigar qué se accedió |
| 03:05 | Alerta en registro borrado 1102 | **CRÍTICA** - Asumir contención falló, escalar |

**Si atrapaste Event 00:05 (pico de 4625):** Ransomware prevenido
**Si atrapaste Event 00:15 (PowerShell):** Ransomware prevenido
**Si atrapaste Event 00:18 (servicio malicioso):** Ransomware prevenido  
**Si solo atrapas Event 03:05 (registro borrado):** 3 horas demasiado tarde

---

## ❌ Errores Comunes que Cometen los Estudiantes

### **Error 1: Pensar "Un Event ID = Una Alerta"**

**Enfoque incorrecto:**
```
Alerta: Event 4688 creación de proceso detectada
Respuesta: Ver un proceso, borrarlo si se ve ok
```

**Enfoque correcto:**
```
Alerta: Event 4688 creación de proceso detectada
Respuesta: 
1. Ver proceso PADRE (¿de dónde viene?)
2. Ver procesos HIJO (¿qué lanzó?)
3. Ver TIEMPO (¿cuándo se ejecutó?)
4. Ver FRECUENCIA (¿cuántas veces?)
5. Referencia cruzada con otros eventos (5140, 4720, etc.)
```

**Consecuencia real:** Analista borra el proceso padre y pierde toda la cadena de ataque. El atacante permanece en el sistema.

---

### **Error 2: Ignorar Event 1102 (Registro Borrado)**

**Enfoque incorrecto:**
```
Aparece Event 1102
Analista: "Probablemente mantenimiento programado"
Respuesta: Ninguna
```

**Enfoque correcto:**
```
Aparece Event 1102
Analista: "Alguien está destruyendo evidencia"
Respuesta: ESCALACIÓN INMEDIATA
- Verificar si borrado de logs fue autorizado
- Si no: ASUMIR BRECHA ACTIVA
- Aislar máquina, recopilar forenses, llamar a respuesta a incidentes
```

**Consecuencia real:** El atacante cubre exitosamente sus huellas y permanece sin detectar durante meses.

---

### **Error 3: No Entender la Diferencia Entre Fallo de Inicio (4625) y Inicio Exitoso (4624)**

**Enfoque incorrecto:**
```
4625 (Fallido) = Alerta
4624 (Exitoso) = Normal
```

**Enfoque correcto:**
```
4625 (Fallido) × 200 = Intento de fuerza bruta (alerta)
4624 (Exitoso) desde misma IP = BRECHA (escalar inmediatamente)
```

**Consecuencia real:** Analista ve inicio exitoso como normal y no lo conecta con intentos fallidos. La brecha comienza.

---

### **Error 4: No Revisar CommandLine en Event 4688**

**Enfoque incorrecto:**
```
Event 4688: powershell.exe iniciado
Analista: "PowerShell es normal, los usuarios lo usan todo el tiempo"
Respuesta: Ignorar
```

**Enfoque correcto:**
```
Event 4688: powershell.exe iniciado
Revisar: ¿Cuál es la LÍNEA DE COMANDOS COMPLETA?
- Si: -enc JABzAGM... (codificado) = ALERTA
- Si: -NoProfile (sin perfil de usuario) = ALERTA
- Si: -ExecutionPolicy Bypass (evadir restricciones) = ALERTA
- Si: -NonInteractive -NoProfile (automatizado) = ALERTA
- Si: Usuario normal lanzando notepad = OK
```

**Consecuencia real:** Analista pierde explotación de PowerShell porque no leyó la línea de comandos completa.

---

## 🧪 Escenario de Práctica: Investigación de Incidente

### **Escenario: Recibes Alerta a las 2 PM**

**Alerta:** "Ejecución sospechosa de PowerShell detectada en WORKSTATION-05"

**Datos de Evento Crudos:**
```
Event 4688: Creación de Proceso
Hora: 2024-01-15 14:22:33
Computadora: WORKSTATION-05
Usuario: jsmith
Image: C:\Windows\System32\powershell.exe
CommandLine: powershell.exe -enc JgBjAG8AbQA=
ParentImage: C:\Windows\Explorer.exe
ParentProcessID: 4892

Seguido por:

Event 5140: Acceso a Recurso Compartido de Red
Hora: 2024-01-15 14:23:45
Computadora: WORKSTATION-05
Usuario: jsmith
ShareName: \\FileServer01\Payroll
AccessMask: Lectura

Seguido por:

Event 5140: Acceso a Recurso Compartido de Red
Hora: 2024-01-15 14:24:12
Computadora: WORKSTATION-05
Usuario: jsmith
ShareName: \\FileServer01\HR
AccessMask: Lectura
```

### **Preguntas de Análisis:**

**P1:** ¿Es definitivamente un ataque, o podría ser legítimo?
- A) Definitivamente ataque
- B) Podría ser legítimo (usuario usando PowerShell para trabajo)
- C) Necesito más datos para decidir
- D) Preguntar al usuario directamente

**P2:** ¿Cuál es el evento más preocupante aquí?
- A) Ejecución de PowerShell en sí
- B) El comando PowerShell codificado
- C) Acceso a archivos de Nómina y RRHH
- D) El tiempo

**P3:** ¿Qué deberías hacer PRIMERO?
- A) Matar el proceso PowerShell
- B) Desactivar la cuenta de usuario
- C) Revisar si jsmith normalmente accede a esos recursos a esta hora
- D) Escalar a respuesta a incidentes

**P4:** ¿Qué eventos adicionales quisieras ver?
- A) Event 4720 (nuevas cuentas creadas)
- B) Event 4698 (tareas programadas creadas)
- C) Event 4724 (reseteos de contraseña)
- D) Todos los anteriores

### **Solución:**

**Respuesta P1: C) Necesito más datos**
- Razón: PowerShell en sí no es malicioso, y acceso a archivos podría ser normal. Pero la COMBINACIÓN es sospechosa.

**Respuesta P2: B) Comando PowerShell codificado**
- Razón: Comandos PowerShell legítimos usualmente no están codificados. Codificación = esconder algo.

**Respuesta P3: C) Revisar si jsmith normalmente accede a esos recursos**
- Razón: Saltar a acción podría alertar al atacante si aún está activo. Primero, validar si es comportamiento normal o anómalo.

**Respuesta P4: D) Todos los anteriores**
- Razón: Si EXISTEN Events 4720/4698/4724, esto confirma tácticas de persistencia/escalación.

---

## 🎯 Preguntas de Entrevista Que Podrías Recibir

### **Preguntas Fáciles (L1 - Analista Junior)**

**P: "Explica la diferencia entre Event Log y Registro."**

**Respuesta esperada:**
> "Event Log es un registro de qué sucedió (acciones en el sistema), mientras que Registro es datos de configuración almacenados persistentemente. Event Log va a SIEM para detección en tiempo real; Registro se revisa durante forenses."

---

**P: "¿Qué te dice el Event ID 4688?"**

**Respuesta esperada:**
> "Es Creación de Proceso - registra cuándo un nuevo proceso se inicia, incluyendo nombre del programa, quién lo ejecutó, argumentos de línea de comandos completos, y proceso padre. Es crítico porque ejecución de malware aparece aquí."

---

**P: "¿Por qué PowerShell es más peligroso que cmd.exe?"**

**Respuesta esperada:**
> "PowerShell se ejecuta en memoria y puede codificar sus comandos, haciéndolo más difícil de detectar. Tiene acceso al framework .NET completo, así que puede hacer mucho más daño que cmd. Y está incorporado en Windows, así que no hay archivo sospechoso que encontrar."

---

### **Preguntas Medias (L2 - Analista de Nivel Medio)**

**P: "Camina a través de cómo investigarías un ataque sospechoso de ransomware en un endpoint Windows."**

**Respuesta esperada:**
> "Primero, reviearía Event 4625 para intentos fallidos de RDP para identificar el ataque inicial. Luego Event 4624 para login exitoso desde IP externa. Event 4688 para ejecución de proceso sospechosa (especialmente PowerShell). Event 7045 para instalación maliciosa de servicio. Event 5140 para datos siendo leídos antes de cifrado. Y críticamente, revisar si Event 1102 fue borrado (destrucción de evidencia). Si todos estos eventos existen, es brecha confirmada y escalería a respuesta a incidentes para aislar la máquina antes de que contenencia falle."

---

**P: "Un atacante compromete una máquina Windows pero no crea nuevas cuentas ni servicios. ¿Cómo los detectarías?"**

**Respuesta esperada:**
> "Incluso sin crear mecanismos de persistencia, tienen que HACER algo, así que:
> - Event 4688: Si ejecutaron cualquier herramienta (PowerShell, cmd, malware)
> - Event 5140: Si accedieron a recursos compartidos para robar datos
> - Event 4625 + 4624: La brecha inicial vía RDP u otro login
> - Análisis de memoria: Si ejecutan malware solo en RAM
> - Conexiones de red salientes: Si se comunican con comando y control
> Si no tocan Event Logs o Registro, el único rastro es sus acciones (4688, 5140, red)."

---

### **Preguntas Difíciles (L3 - Senior/Entrevista)**

**P: "Diseña una estrategia de detección para ataques de Kerberoasting en Windows."**

**Respuesta esperada:**
> "Kerberoasting explota el protocolo Kerberos para craquear contraseñas de cuentas de servicio offline. Detección:
> 1. Monitorear Event 4769 (Ticket de Servicio Kerberos solicitado) para solicitudes inusuales de tickets de servicio
> 2. Alertar en múltiples eventos 4769 del mismo usuario en corto período
> 3. Enfocarse en cuentas de servicio (vs cuentas de usuario) en solicitudes de tickets
> 4. Correlacionar con Event 5140 o 4688 (¿qué hicieron después de obtener el ticket?)
> 5. Investigar si usuario que solicitó ticket tiene razón legítima para acceder a ese servicio
> Implementación: Regla SIEM que dispara en >5 Event 4769 por usuario por hora, filtra tráfico conocido-bueno, escala a respuesta a incidentes"

---

**P: "Un atacante tiene credenciales de admin para un endpoint Windows. ¿Cómo detectarías que está usando esas credenciales, incluso si borra Event Logs?"**

**Respuesta esperada:**
> "Si Event Logs se borran (Event 1102), se destruyen a nivel de endpoint. Pero:
> 1. SIEM debería haber reenviado logs antes del borrado
> 2. Datos SIEM de red (firewall, DNS, proxy) aún muestran conexiones salientes
> 3. Registros de servidor de archivos (Event 5140 en el SERVIDOR) muestran quién accedió a archivos
> 4. Datos de tiempo-de-acceso en archivos mismos revelan quién accedió
> 5. Análisis de Registro post-brecha muestra qué se cambió
> El atacante solo puede borrar logs locales; no puede borrar logs en servidores que accedió. Así que investigar logs de esos servidores para nombre de usuario del atacante."

---

**P: "¿Qué indicaría compromiso de Active Directory, y cómo responderías?"**

**Respuesta esperada:**
> "Indicadores de compromiso de AD:
> - Event 4720/4722: Múltiples cuentas de usuario creadas/modificadas fuera de ventana de cambios
> - Event 4728: Adición masiva de usuarios a grupo Domain Admins
> - Event 4769: Solicitudes de tickets para cuentas de servicio sensibles
> - Event 4625: Ataques de spray de credenciales (misma contraseña probada en muchas cuentas)
> Respuesta:
> 1. INMEDIATO: Aislar DC de red (desconectar cable de red)
> 2. Revisar cuentas backdoor (4720) y eliminarlas
> 3. Resetear contraseñas para todas cuentas Domain Admin
> 4. Revisar Objetos de Política de Grupo (GPOs) para cambios maliciosos
> 5. Si comprometido: asumir bosque AD completo comprometido, planear reconstrucción
> Esta es la opción nuclear porque compromiso de AD = organización completa comprometida"

---

## 🔗 Cómo Esto Se Conecta a Todo Lo Demás

### **Windows → Active Directory**
- Máquinas Windows se autentican a AD usando Kerberos
- Endpoint Windows comprometido = punto de entrada potencial a AD
- Compromiso de AD = todas las máquinas Windows comprometidas
- Estudia AD si quieres entender seguridad empresarial

### **Windows → Seguridad de Red**
- Windows genera conexiones de red salientes (Event 5156)
- Registros de firewall muestran con qué el endpoint intenta comunicarse
- Solicitudes DNS muestran qué el endpoint está resolviendo
- Referencia cruzada: Si Event 4688 muestra ejecución de PowerShell, revisar logs de firewall para conexiones salientes a C&C de malware

### **Windows → SIEM**
- Event Logs son la fuente de datos primaria para monitoreo de Windows
- SIEM recolecta, parsea y correlaciona eventos de Windows
- Correlación: Event 4625 + Event 4624 desde misma fuente = brecha
- Sin conocimiento de Windows Event Log, no puedes usar SIEM efectivamente

### **Windows → Respuesta a Incidentes**
- IR recolecta artefactos forenses de Windows (Registro, EventLog, MFT)
- Reconstrucción de línea de tiempo: ¿Qué eventos sucedieron en qué orden?
- Atribución: ¿Quién fue el atacante (qué cuenta de usuario)?
- Evaluación de daño: ¿Qué archivos se accedieron/modificaron?

### **Windows → Threat Hunting**
- Busca signos de persistencia: Event 7045 (servicios), Event 4698 (tareas)
- Busca reconocimiento: Event 4688 (comandos whoami, ipconfig)
- Busca movimiento lateral: Event 5140 (patrones de acceso a archivos)
- Busca exfiltración: Event 5140 + logs de red (transferencias de datos grandes)

---

## 💾 TL;DR Para Gente Ocupada

### **Los 10 Event IDs Más Críticos (Memoriza Estos)**

| Event ID | Qué | Por Qué Alertar |
|----------|-----|-----------------|
| **4688** | Proceso iniciado | Ejecución de malware/herramienta |
| **4720** | Usuario creado | Cuenta backdoor |
| **4625** | Inicio fallido | Fuerza bruta, spray de contraseña |
| **4624** | Inicio exitoso | + 4625 desde misma IP = brecha |
| **7045** | Servicio instalado | Persistencia de malware |
| **4724** | Contraseña reseteada | Robo de credenciales |
| **5140** | Recurso compartido accedido | Robo de datos, movimiento lateral |
| **4698** | Tarea creada | Malware programado |
| **1102** | Registros borrados | Encubrimiento = ESCALAR |
| **4769** | Ticket Kerberos | Preparación de movimiento lateral |

### **La Secuencia de Ataque (En Orden)**

1. **Event 4625** (Fuerza bruta de RDP) → Atacante probando contraseñas
2. **Event 4624** (Inicio exitoso) → Atacante en el sistema
3. **Event 4688** (PowerShell) → Atacante ejecutando herramientas
4. **Event 7045** o **4698** (servicio/tarea) → Atacante creando persistencia
5. **Event 5140** (acceso a recurso compartido) → Atacante robando datos
6. **Event 1102** (registros borrados) → Atacante destruyendo evidencia

**Interrumpe en CUALQUIERA de estos pasos y detiene el ataque.**

### **Resumen en Una Oración**

*"Los Event Logs de Windows son la fuente de datos de detección de amenazas primaria de tu SOC; domina los 10 Event IDs principales, entiende cómo se encadenan juntos, y puedes detectar 80% de ataques antes de que ocurra daño significativo."*

---

## 📌 Realidad de Producción: Cómo Funciona Realmente

### **Entorno Real de SOC**

**Escenario:** Eres analista en una empresa mediana con 500 endpoints Windows.

**Realidad Diaria:**
- Los 500 endpoints reenvían Event Logs a SIEM cada 30 segundos
- SIEM ingiere ~1 millón de eventos por hora (solo de Windows)
- Tu trabajo: Encontrar el 1 evento malicioso entre 1 millón legítimos

**Tus herramientas:**
```
Regla 1: Alerta si Event 4625 > 20 en 10 minutos (fuerza bruta)
Regla 2: Alerta si Event 4688 con argumento -enc de PowerShell (malware codificado)
Regla 3: Alerta si Event 7045 con binario en %TEMP% (servicio de malware)
Regla 4: Alerta si Event 1102 es generado (borrado de logs = brecha)
Regla 5: Alerta si Event 5140 fuera de horario comercial (robo de datos post-horario)
```

**Falsos Positivos que Enfrentarás:**
- Script de PowerShell de admin de TI = parece malware
- Instalación de servicio por actualizador de software = parece persistencia de malware
- Sistema reiniciando = genera muchos eventos en sucesión rápida
- Bloqueo de cuenta de usuario = genera muchos eventos 4625

**Tu trabajo:** Construir reglas que atrapen ataques reales ignorando falsos positivos. Este es el arte del trabajo en SOC.

---

### **Línea de Tiempo Real de Brecha**

**Empresa XYZ obtiene ransomware:**

```
Día 1 (14:00): Atacante comienza fuerza bruta de RDP (pico Event 4625)
→ SOC alerta
→ Analista desestima como "probablemente usuario con contraseña mala"

Día 1 (15:00): Atacante tiene éxito (Event 4624 desde IP externa)
→ Sin alerta configurada (¡oops!)
→ Atacante ahora en red, sin detectar

Día 2-3: Atacante construye persistencia (Event 7045, 4698)
→ Las alertas existen pero se ignoran como "mantenimiento del sistema"

Día 4: Atacante se mueve lateralmente (Event 5140, 4724)
→ Los eventos se generan pero no se correlacionan
→ Analista ve cada evento en aislamiento, pierde el patrón

Día 5 (03:00): Ransomware se ejecuta
→ Archivos cifrados
→ Negocio caído por 3 días
→ Rescate demandado

Día 5 (03:05): Atacante borra logs (Event 1102)
→ Evidencia parcialmente destruida
→ Forenses se vuelve más difícil

**Post-mortem:**
> "Si SOC hubiera correlacionado Event 4625 + Event 4624 en Día 1, lo habríamos atrapado en minutos. Si hubieran entendido Event 7045 y 4698, lo habríamos atrapado en Día 2. En su lugar, todos los datos estaban ahí y los perdimos."
```

---

### **Qué Significa Esto para Tu Carrera**

1. **El conocimiento de Windows es innegociable** en cualquier rol de SOC
2. **La comprensión de Event Log es tu habilidad fundamental** (no opcional)
3. **Construir reglas de detección efectivas** requiere saber qué se ve normal
4. **Aprender SIEM sin conocimiento de Windows** es como aprender a manejar sin entender motores

**Pasos siguientes después de dominar esto:**
- Aprende Active Directory a fondo (donde la seguridad de Windows se vuelve compleja)
- Aprende correlación de SIEM (cómo conectar múltiples eventos en patrones)
- Aprende Threat Hunting (búsqueda proactiva de ataques perdidos)
- Aprende Respuesta a Incidentes (qué hacer después de encontrar el ataque)

---

## 📚 Lectura Adicional

### **Libros**
- *Windows Security Internals* por Mark Russinovich (profundidad)
- *The Art of Memory Forensics* (captura malware en memoria)
- *Incident Response & Computer Forensics* por Chris Ryan

### **Documentación**
- [Microsoft Security Audit Events](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-events)
- [Recomendaciones de Windows Event Log](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Framework Mitre ATT&CK - Técnicas de Windows](https://attack.mitre.org/platforms/windows/)

### **Labs & Práctica**
- DetectLabs: Escenarios de análisis de event log de Windows
- TryHackMe: Ruta de Windows Fundamentals
- AlteredSecurity: Labs de Active Directory
- Desafío de Vacaciones de SANS (anual, gratis)

### **Herramientas para Aprender**
- Event Viewer (Windows nativo)
- PowerShell (para queries de logs)
- Splunk o ELK (plataformas SIEM)
- Velociraptor (visibilidad de endpoint)

---

## 🎓 Pensamiento Final

Windows es el campo de batalla donde sucede la defensa cibernética. Cada día, cientos de miles de ataques apuntan a endpoints Windows. Tu trabajo como analista de blue team es:

1. **Ver el ataque** (entender Event Logs)
2. **Entender el ataque** (conocer técnicas y tácticas)
3. **Detener el ataque** (responder antes de que ocurra daño)

Domina Windows, y dominas la base del trabajo moderno de SOC.

**Tú puedes hacerlo.** 🛡️

---

**Versión del Documento:** 1.0  
**Última Actualización:** Enero 2024  
**Creado para:** Capacitación de Seguridad Blue Team  
**Tema Siguiente:** Seguridad de Active Directory
