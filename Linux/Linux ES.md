# Linux para Analistas de Blue Team y SOC

## 📖 ¿Qué Es Esto? (En 30 Segundos)

Linux es un **sistema operativo basado en Unix, de código abierto** que es la base de la infraestructura de seguridad en SOCs modernos. A diferencia de Windows, Linux se construye sobre principios de permisos de archivos, aislamiento de procesos, y logging detallado—lo que lo hace esencial para investigaciones de seguridad, ejecutar herramientas de seguridad, y asegurar servidores. En trabajos de Blue Team/SOC, ENCONTRARÁS sistemas Linux tanto como objetivos a investigar como plataformas donde corren tus herramientas de seguridad.

**Distinción clave:** Linux no es "fácil de editar"—más bien, es transparente. Puedes ver exactamente qué está corriendo, quién tiene acceso, y qué cambió. Esta transparencia es lo que lo hace poderoso para seguridad.

---

## 🎯 ¿Por Qué Un Profesional De Blue Team/SOC Necesita Esto?

### En Entrevistas De Trabajo, Escucharás:
- "Explica cómo funcionan los permisos de Linux y por qué importan en investigaciones de seguridad"
- "¿Cómo investigarías un servidor Linux comprometido? Camina a través del proceso"
- "¿Cuál es la diferencia entre un usuario normal y sudo? ¿Por qué importa?"
- "¿Qué puedes aprender de /var/log/auth.log?"
- "Si un archivo tiene permisos `rwxr-x---`, ¿quién puede leerlo? ¿Quién puede ejecutarlo?"
- "¿Cómo detectarías escalada de privilegios en un sistema Linux?"
- "Compara cómo investigarías un compromiso en Linux vs Windows"

### En Tu Primer Mes En Un SOC:
- Tu SOC probablemente tendrá servidores Linux (servidores web, bases de datos, infraestructura de logging)
- Necesitarás SSH dentro de estos servidores para investigar incidentes
- Examinarás `/var/log/auth.log`, `/var/log/syslog`, y otros logs buscando evidencia
- Buscarás cambios sospechosos de permisos de archivos, uso inusual de sudo, o procesos desconocidos
- Necesitarás entender aislamiento de procesos, permisos de usuarios, y redes básicas
- Ejecutarás herramientas como `nmap`, `tcpdump`, `ss`, `netstat` para recopilar evidencia
- **Hacerlo mal = perder el ataque, contaminar evidencia, o malinterpretar logs**

---

## 🔍 Conceptos Principales Desglosados

### **CONCEPTO 1: USUARIOS Y PERMISOS DE LINUX (El Fundamento)**

#### Qué Es:
Linux es un **sistema operativo multi-usuario**. A diferencia de tu PC Windows personal (donde solo eres tú), los servidores Linux tienen:
- Usuario Root (como Administrador en Windows)
- Usuarios regulares (con permisos limitados)
- Usuarios del sistema (para servicios como motores de base de datos, servidores web)

Cada usuario tiene un único **UID (ID de Usuario)** y pertenece a **grupos** (como "admin", "sudo", "www-data").

#### Por Qué Importa En Seguridad:

**Escenario 1: Atacante obtiene acceso**
- Si atacante compromete una cuenta de usuario regular = daño limitado (no puede modificar archivos del sistema)
- Si atacante obtiene acceso root = compromiso total (puede hacer cualquier cosa)
- Tu trabajo en SOC = determinar qué usuario fue comprometido y qué podían acceder

**Escenario 2: Escalada de privilegios**
- Atacante comienza como usuario regular, explota vulnerabilidad para volverse root
- Evidencia: `/var/log/auth.log` muestra uso de `sudo` o intentos fallidos de sudo
- Tu trabajo = detectar este patrón de escalada

#### Ejemplo Real:

```
# Usuario "john" intenta ver archivos del sistema (falla porque no tiene permiso)
john@server:~$ cat /etc/shadow
cat: /etc/shadow: Permission denied

# Usuario intenta usar sudo (permite acceso temporal como root)
john@server:~$ sudo cat /etc/shadow
[sudo] password for john: 
root:*:19000:0:99999:7:::
mysql:!:19000:0:99999:7:::
```

En este ejemplo:
- `/etc/shadow` = archivo que solo root puede leer (almacena hashes de contraseña)
- `john` no puede leerlo como usuario regular
- `john` PUEDE leerlo usando `sudo` (si john está en el archivo sudoers)

**En una investigación de SOC:**
- Si ves a "john" accediendo `/etc/shadow` sin sudo, es una bandera roja importante (compromiso)
- Si ves a "john" usando `sudo` 50 veces a las 3 AM, eso es sospechoso (movimiento lateral o escalada)

---

### **CONCEPTO 2: PERMISOS DE ARCHIVOS (Cómo Funciona La Seguridad)**

#### Qué Es:
Cada archivo en Linux tiene permisos que definen:
- **Quién** puede acceder a él (dueño, grupo, otros)
- **Qué** pueden hacer (leer, escribir, ejecutar)

#### Cómo Leer Permisos:

```
-rwxr-x---  1  root  admin  4096  Mar 15 10:00  /etc/passwd

Desglosándolo:
- = archivo regular (d = directorio, l = enlace simbólico)
rwx = dueño (root) puede leer, escribir, ejecutar
r-x = grupo (admin) puede leer, ejecutar (NO escribir)
--- = otros no pueden hacer NADA
```

**Los tres tipos de permisos:**
- **r (leer)** = puede ver contenidos del archivo
- **w (escribir)** = puede modificar o eliminar archivo
- **x (ejecutar)** = puede ejecutar archivo como programa (para directorios: puede entrar)

#### Ejemplo De Seguridad Real:

```
-rw-r--r--  important_file.txt
= Dueño puede leer/escribir, grupo puede leer, otros pueden leer
= RIESGO DE SEGURIDAD: Cualquiera en el sistema puede leer esto

-rw-------  password.txt
= Solo dueño puede leer/escribir, nadie más puede acceder
= SEGURO: Archivo privado protegido de otros usuarios
```

#### En Investigaciones De SOC:

**Bandera Roja 1: Cambios Sospechosos De Permisos**
```
Normal:    -rw-r--r--  /var/www/html/index.php
Sospechoso: -rwxrwxrwx  /var/www/html/index.php
            ^ Ahora todos pueden leer, escribir, ejecutar!
```
Esto indica que un atacante probablemente modificó el archivo para agregar una puerta trasera.

**Bandera Roja 2: Archivos Del Sistema Escribibles**
```
Normal:    -rw-r--r--  /etc/passwd
Sospechoso: -rw-rw-rw-  /etc/passwd
            ^ Usuarios no-root pueden modificar la base de datos de usuarios!
```
Esto indica escalada de privilegios—el atacante está modificando archivos del sistema.

---

### **CONCEPTO 3: SUDO (Acceso Temporal Como Root)**

#### Qué Es:
`sudo` = "superuser do" = permite a usuarios regulares ejecutar comandos como root, pero solo si están autorizados (en el archivo `/etc/sudoers`).

#### Cómo Funciona:

```bash
# Usuario regular john intenta reiniciar apache
john@server:~$ service apache2 restart
Permission denied (debes ser root)

# john usa sudo (si está autorizado)
john@server:~$ sudo service apache2 restart
[sudo] password for john: 
apache2 restarted
```

**En el archivo `/etc/sudoers`:**
```
john ALL=(ALL) ALL
^ john puede usar sudo para CUALQUIER comando como CUALQUIER usuario
^ ¡Esto es peligroso! Normalmente más restrictivo como:

john ALL=(ALL) /usr/sbin/service
^ john solo puede usar sudo para /usr/sbin/service
```

#### Por Qué Importa En Seguridad:

**Escenario: Atacante escala privilegios**
1. Atacante compromete usuario regular "john"
2. Atacante descubre que john está en el archivo sudoers
3. Atacante ejecuta: `sudo -i` (se convierte en root)
4. Evidencia en logs: `/var/log/auth.log` muestra que "john" usó sudo

**Tu trabajo en SOC:**
- Verificar quién puede usar sudo: `cat /etc/sudoers`
- Revisar logs de uso de sudo: `grep sudo /var/log/auth.log`
- Señalar patrones sospechosos (ej: intentos fallidos de sudo = atacante adivinando contraseña)

#### Banderas Rojas En Uso De Sudo:

```
Mar 15 03:47:22 server sudo: john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/bash
^ Las 3 AM es sospechoso para usuario regular
^ john ejecutó bash como root (shell completamente interactivo)
^ BANDERA ROJA IMPORTANTE: Posible compromiso

Mar 15 03:47:45 server sudo: john : 1 incorrect password attempt
^ Intento de contraseña fallido de sudo = atacante intentando escalar
^ 10+ intentos fallidos = ataque de diccionario en sudo
```

---

### **CONCEPTO 4: LOGS DE LINUX (Dónde Vive La Evidencia)**

#### Logs Clave Que Investigarás:

**1. `/var/log/auth.log` (Log De Autenticación)**
- Cada login, logout, uso de sudo
- Intentos de login fallidos (detección de fuerza bruta)
- Conexiones SSH, cambios de contraseña

Ejemplo de entrada:
```
Mar 15 09:23:45 server sshd[5432]: Failed password for john from 192.168.1.100 port 22 ssh2
^ Usuario "john" falló en login desde IP 192.168.1.100
^ 10+ entradas como esta = ataque de fuerza bruta

Mar 15 09:25:12 server sshd[5433]: Accepted password for john from 192.168.1.100 port 22 ssh2
^ La misma IP finalmente tuvo éxito = credenciales comprometidas
```

**2. `/var/log/syslog` o `/var/log/messages` (Log Del Sistema)**
- Eventos del sistema, errores, advertencias
- Inicios/paradas de servicios
- Mensajes del kernel

**3. `/var/log/secure` (Versión RHEL/CentOS de auth.log)**
- Similar a auth.log en sistemas Red Hat

**4. Logs Específicos De Aplicaciones:**
```
/var/log/apache2/access.log    (acceso a servidor web)
/var/log/apache2/error.log     (errores del servidor web)
/var/log/mysql/error.log       (errores de base de datos)
/var/log/nginx/access.log      (servidor web nginx)
```

#### En Investigaciones De SOC:

```bash
# Encontrar todos los intentos de login fallidos
grep "Failed password" /var/log/auth.log | wc -l
# Resultado: 1247 intentos fallidos = ataque de fuerza bruta

# Encontrar todo uso de sudo
grep "sudo" /var/log/auth.log
# Revisar para encontrar comandos sospechosos de sudo

# Encontrar todos los logins SSH
grep "sshd" /var/log/auth.log | grep "Accepted"
# Identificar todos los logins exitosos, verificar horas/IPs por anomalías

# Encontrar logins desde IP específica
grep "192.168.1.100" /var/log/auth.log
# Rastrear toda actividad de esa IP fuente
```

---

### **CONCEPTO 5: PROCESOS Y SERVICIOS EN EJECUCIÓN**

#### Qué Es:
Un **proceso** es un programa en ejecución. Cada proceso tiene:
- **PID** (ID Del Proceso)
- **Dueño** (qué usuario lo inició)
- **Proceso padre** (qué inició este proceso)
- **Línea de comandos** (argumentos y banderas)

#### Por Qué Importa En Seguridad:

**Proceso Normal:**
```
www-data  5432  0.1  0.2  65536 8192  ?  S  10:00  0:01  /usr/sbin/apache2 -k start
^ Proceso propiedad del usuario www-data (no root)
^ Ejecutando servidor web legítimo apache2
```

**Proceso Sospechoso:**
```
root  6789  15.2  45.1  524288 450000  ?  S  03:47  1:23  /tmp/miner.sh
^ Ejecutándose desde /tmp (ubicación inusual)
^ Usando 45% de memoria del sistema (consumo excesivo)
^ Propiedad de root (privilegios root comprometidos)
^ A las 3:47 AM (cronograma sospechoso)
^ PROBABLE MALWARE (minero de criptomonedas)
```

#### Comandos Para Investigar Procesos:

```bash
ps aux
# Listar todos los procesos en ejecución con detalles

ps aux | grep apache
# Encontrar todos los procesos apache

netstat -tlnp
# Mostrar todos los puertos en escucha y qué proceso los posee

ss -tlnp
# Versión moderna de netstat (más confiable)

top
# Monitor de procesos en tiempo real (ver CPU, uso de memoria)

lsof -i :80
# Encontrar qué proceso está escuchando en puerto 80
```

---

## ⚙️ Lo Que DEBES Memorizar

### Permisos De Archivos (chmod)

```
rwxrwxrwx
│││││││└─ Otros pueden ejecutar
││││││└── Otros pueden escribir
│││││└─── Otros pueden leer
││││└──── Grupo puede ejecutar
│││└───── Grupo puede escribir
││└────── Grupo puede leer
│└─────── Dueño puede ejecutar
└──────── Dueño puede leer y escribir
```

**Permisos Comunes:**
- `755` = rwxr-xr-x (dueño: rwx, grupo+otros: rx) - típico para scripts ejecutables
- `644` = rw-r--r-- (dueño: rw, grupo+otros: r) - típico para archivos
- `700` = rwx------ (dueño: rwx solo) - archivos privados
- `777` = rwxrwxrwx (todos pueden hacer todo) - RIESGO DE SEGURIDAD

### Patrón De Escalada Via Sudo

```
1. Atacante compromete usuario regular
2. Atacante verifica si usuario está en sudoers: sudo -l
3. Si está permitido, atacante ejecuta: sudo -i (se convierte en root)
4. Evidencia: auth.log muestra uso de sudo y comandos
```

### Directorios Clave De Linux

```
/home/          - Directorios de inicio de usuarios (archivos personales)
/root/          - Directorio de inicio del usuario root
/etc/           - Archivos de configuración (sudoers, passwd, shadow)
/var/log/       - Archivos de log (dónde empiezan investigaciones)
/tmp/           - Archivos temporales (a menudo usado para malware)
/usr/bin/       - Programas ejecutables estándar
/usr/sbin/      - Programas de administración del sistema (requieren sudo)
```

### Comandos Críticos Para SOC

```
# Comandos de usuario y permisos
id                          # Mostrar info de usuario actual
whoami                      # Mostrar nombre de usuario actual
sudo -l                     # Listar qué puede hacer sudo el usuario actual
cat /etc/sudoers            # Ver permisos de sudo
cat /etc/passwd             # Ver todos los usuarios
cat /etc/shadow             # Ver hashes de contraseña (solo root)

# Comandos de archivo
ls -la /directory           # Listar archivos con permisos
chmod 755 filename          # Cambiar permisos de archivo
chown owner:group filename  # Cambiar dueño de archivo
find / -perm -4000         # Encontrar archivos setuid (riesgo de escalación)

# Comandos de proceso
ps aux                      # Listar todos los procesos
netstat -tlnp              # Mostrar puertos en escucha y procesos
ss -tlnp                   # Alternativa moderna a netstat
lsof -i :PORT              # Mostrar qué está escuchando en puerto

# Comandos de log
grep "pattern" /var/log/auth.log    # Buscar patrones
tail -n 100 /var/log/auth.log       # Mostrar últimas 100 líneas
grep "sudo" /var/log/auth.log       # Encontrar uso de sudo
grep "Failed" /var/log/auth.log     # Encontrar logins fallidos

# Comandos de red
ip addr show                # Mostrar interfaces de red
ip route show              # Mostrar tabla de rutas
ss -an                     # Mostrar todas las conexiones
```

---

## 📚 Lo Que DEBES Entender

- [ ] **Linux es transparente por defecto** — Puedes ver qué está corriendo, quién es su dueño, y qué cambió (a diferencia de Windows que oculta cosas)
- [ ] **Los permisos no son opcionales** — Son lo ÚNICO que previene que usuarios accedan archivos de otros
- [ ] **Sudo es un vector de escalada de privilegios** — Si atacante obtiene sudo, obtiene root; si obtiene root, lo obtiene todo
- [ ] **Los logs son tu punto de inicio de investigación** — `/var/log/auth.log` te dice quién accedió qué y cuándo
- [ ] **Usuarios y grupos son límites de seguridad** — Usuario www-data está restringido; root no lo está
- [ ] **La propiedad de archivos importa** — Un archivo malicioso propiedad de root es más peligroso que uno de usuario regular
- [ ] **Los procesos revelan compromiso** — Procesos inesperados, especialmente en /tmp o corriendo como root, indican ataque
- [ ] **Las claves SSH reemplazan contraseñas en servidores** — Muchos sistemas Linux usan claves SSH, no contraseñas (hace imposible fuerza bruta)

---

## 🚨 Aplicación Del Mundo Real: Una Investigación De Compromiso Linux

### Escenario: Servidor Web Hackeado

**Alerta Inicial (7:00 AM):**
- IDS alerta sobre tráfico de salida sospechoso desde servidor web
- CPU del servidor web está en 95% (normalmente 10-15%)
- I/O de disco es extremadamente alto

**Paso 1: Investigar Procesos (8:00 AM)**
```bash
ps aux | head -20
# Ves:
# www-data  5432  87.3  42.1  524288 400000  ?  R  06:47  1:13  /tmp/xmrig
# ^ Usuario www-data ejecutando minero crypto en /tmp
# ^ Usando 87% CPU y 400MB RAM
# ^ Iniciado a las 6:47 AM (correlaciona con alerta)
```

**Paso 2: Verificar Conexiones De Red (8:05 AM)**
```bash
ss -tlnp | grep 5432
# Muestra conexión a 192.168.1.50:3333
# ^ Este es un pool de mining conocido
# ^ Confirma que malware está exfiltrando recursos de computación
```

**Paso 3: Investigar Cómo Entraron (8:15 AM)**
```bash
grep "www-data" /var/log/auth.log | tail -50
# Ves:
# Mar 15 06:30:00 server sshd: Accepted password for www-data from 203.0.113.50
# ^ ¡Pero usuario www-data no debería poder SSH!
# ^ Alguien habilitó SSH para www-data o cambió permisos
```

**Paso 4: Verificar Permisos De Archivos (8:20 AM)**
```bash
ls -la /etc/sudoers.d/
# Ves:
# -rw-r--r--  www-data
# ^ www-data ahora puede ejecutar comandos como root!
# ^ Así es cómo atacante escaló privilegios

ls -la /tmp/ | grep xmrig
# -rwxr-xr-x  www-data  /tmp/xmrig
# ^ Ejecutable sospechoso en /tmp (ubicación de malware)
```

**Paso 5: Reconstrucción De Cronograma**
```
6:30 AM - Atacante SSH en servidor como www-data (contraseña comprometida)
6:32 AM - Atacante usa sudo para agregarse a archivo sudoers
6:35 AM - Atacante descarga minero xmrig en /tmp
6:47 AM - Atacante inicia minero (detectado por IDS/spike de CPU)
```

**Causa Raíz:** Contraseña de www-data era débil y fue rota por fuerza bruta.

**Tu Respuesta:**
1. Matar proceso del minero
2. Remover www-data de sudoers
3. Reiniciar contraseña de www-data
4. Auditar todos los archivos modificados por www-data
5. Verificar si atacante accedió datos del cliente
6. Parchar vulnerabilidad de aplicación web que llevó al compromiso

---

## ❌ Errores Comunes Que Cometen Los Estudiantes

### Error 1: Confundir "Código Es Abierto" Con "El Sistema Es Fácil De Cambiar"
**Incorrecto:** "Linux es código abierto así que puedo modificar cualquier archivo que quiera"
**Correcto:** El código fuente de Linux es visible, pero los permisos de archivo aún te previenen de editar archivos que no posees
**Consecuencia Real:** No puedes modificar `/etc/passwd` a menos que seas root, incluso aunque puedas leer el código fuente

### Error 2: No Entender Los Números De Permisos
**Incorrecto:** "777 y 755 son básicamente lo mismo"
**Correcto:** 777 = rwxrwxrwx (TODOS pueden escribir), 755 = rwxr-xr-x (solo dueño puede escribir)
**Consecuencia Real:** Usar 777 en archivos sensibles = permitir a cualquier usuario eliminar o modificarlos

### Error 3: Asumir Sudo = Solo "Ejecutar Como Root"
**Incorrecto:** "sudo es solo para que administradores ejecuten comandos"
**Correcto:** sudo es cómo sucede la escalada de privilegios. Atacante obtiene usuario regular → usa sudo → se convierte en root
**Consecuencia Real:** Perder abuso de sudo en logs = perder toda la cadena de escalada

### Error 4: No Verificar Propiedad De Procesos
**Incorrecto:** "Mataré el proceso de malware y listo"
**Correcto:** Verificar quién posee el proceso, de dónde vino, a qué se está conectando
**Consecuencia Real:** Matas el proceso visible pero dejas la puerta trasera, atacante lo re-ejecuta

### Error 5: Ignorar Compromiso De Claves SSH
**Incorrecto:** "Los logins SSH están en auth.log, así que puedo detectarlos fácilmente"
**Correcto:** Si atacante tiene clave SSH, ningún intento de contraseña se registra—solo aparecen como "Accepted publickey"
**Consecuencia Real:** Perder compromiso de clave SSH en tu investigación

### Error 6: Pensar Que Habilidades De Investigación Windows Se Transfieren Directamente
**Incorrecto:** "Investigué Windows, Linux debe ser similar"
**Correcto:** Investigación Linux se basa en logs de línea de comandos, árboles de procesos, y permisos; Windows se basa en Visor de Eventos, registro, y ACLs
**Consecuencia Real:** Buscar en logs equivocados, malinterpretar evidencia, conclusiones equivocadas

---

## 🧪 Escenario De Práctica: Analiza Este Compromiso

### Datos Del Escenario:

```
# Procesos de usuario:
Salida ps aux:
root      1234  0.1  0.1  65536 8192   ?  S  09:00  0:01  /usr/sbin/sshd -D
john      5432  2.3  1.5  524288 12288 ?  S  10:47  0:23  /bin/bash
john      5433  87.4 45.0 1000000 400000 ? R  10:48  3:45  /usr/bin/python3 /tmp/miner.py
root      5434  0.1  0.2  131072 16384 ?  S  10:50  0:02  /usr/bin/scp

# Permisos de archivos:
ls -la /etc/sudoers.d/:
-rw-r--r-- root root    john
-rw-r--r-- root root    www-data

# Logs:
grep "john" /var/log/auth.log:
Mar 15 10:45:32 server sshd[4999]: Accepted password for john from 203.0.113.10 port 22 ssh2
Mar 15 10:46:15 server sudo: john : TTY=pts/0 ; PWD=/tmp ; USER=root ; COMMAND=/usr/bin/python3
Mar 15 10:47:23 server sudo: john : 1 incorrect password attempt ; TTY=? ; PWD=/tmp ; USER=root ; COMMAND=/bin/bash
Mar 15 10:47:45 server sudo: john : TTY=pts/0 ; PWD=/tmp ; USER=root ; COMMAND=/bin/bash

# Red:
Salida ss -tlnp:
tcp  LISTEN  5433   0.0.0.0:4444  * (python3)
tcp  LISTEN  5434   192.168.1.50:3333  * (scp)
```

### Preguntas:

1. **¿Qué sucedió?** Cronograma del ataque (cuándo entró atacante, qué hizo, en qué orden)
2. **¿Cómo escalaron?** ¿Qué método usó atacante para ir de john→root?
3. **¿Cuál es el malware?** ¿Qué proceso debería preocuparte más y por qué?
4. **¿Qué evidencia es crítica?** ¿Qué entradas de log prueban el compromiso?
5. **¿Qué falta?** ¿Qué investigarías más?

### Respuestas:

**1. Cronograma:**
- 10:45:32 - Atacante SSH en servidor como usuario "john" desde IP externa 203.0.113.10
- 10:46:15 - Atacante ejecuta exitosamente comando sudo como root (escalación exitosa)
- 10:47:23 - Atacante intenta cambiar a bash como root (primer intento falló)
- 10:47:45 - Atacante se convierte exitosamente en shell root
- 10:48:00 - Atacante lanza proceso de mining (/tmp/miner.py) como root

**2. Método de escalación:**
- Usuario john estaba en archivo sudoers (ves en /etc/sudoers.d/ que john tiene derechos sudo)
- Atacante conocía contraseña de john (fuerza bruta o ingeniería social)
- Atacante ejecutó `sudo /usr/bin/python3` (sin aviso de contraseña si sudoers lo permite)

**3. Identificación de malware:**
- `/usr/bin/python3 /tmp/miner.py` es el malware
- Ejecutándose como root (PID 5434, dueño root)
- Usando 87% CPU, 400MB RAM (consumo excesivo = minero de criptomonedas)
- Escuchando en puerto 4444 y conectando a 192.168.1.50:3333 (pool de mining)

**4. Evidencia crítica:**
```
"Accepted password for john from 203.0.113.10" = login del atacante
"sudo: john : USER=root ; COMMAND=" = escalada de privilegios
"1 incorrect password attempt" = atacante adivinando contraseña bash inicialmente
```

**5. Investigación necesaria:**
- ¿Cómo atacante conocía contraseña de john? (¿Spray de contraseña? ¿Phishing? ¿Credenciales filtradas?)
- ¿Las claves SSH de john fueron modificadas?
- ¿Qué accedió atacante antes de lanzar el minero?
- ¿Cuánto tiempo estuvo atacante en el sistema?
- ¿El atacante accedió datos del cliente?
- ¿Hay otras cuentas comprometidas?

---

## 🎯 Preguntas De Entrevista Que Podrías Recibir

### Fácil (Conocimiento L1)

**P:** "¿Cuál es la principal diferencia entre Windows y Linux desde perspectiva de seguridad?"
**R:** "Linux se construye en principio de privilegio mínimo—cada usuario/proceso tiene permisos mínimos por defecto. Los permisos son explícitos y visibles. Windows es por defecto más permisivo. También, servidores Linux suelen ser línea de comandos (CLI) mientras Windows es mucho GUI, lo que cambia el enfoque de investigación."

**P:** "Explica qué hace sudo en una oración."
**R:** "sudo permite a usuarios autorizados ejecutar comandos con privilegios elevados (usualmente como root) sin compartir la contraseña de root."

**P:** "¿Qué significa chmod 755?"
**R:** "Dueño puede leer/escribir/ejecutar, grupo puede leer/ejecutar, otros pueden leer/ejecutar. Típicamente usado para scripts ejecutables."

### Medio (Conocimiento L2)

**P:** "Encuentras un servidor Linux donde los permisos del archivo /etc/sudoers son `-rw-rw-rw-`. ¿Qué significa esto para seguridad y qué deberías investigar?"
**R:** "Esta es una vulnerabilidad crítica—cualquiera puede modificar el archivo sudoers para darse acceso root. Esto indica:
1. Probable escalada de privilegios ya ocurrió (atacante modificó permisos)
2. Necesito verificar logs de sudo para uso inusual
3. Necesito restaurar sudoers desde backup
4. Necesito identificar cómo atacante escaló para modificar sudoers en primer lugar"

**P:** "¿Cómo detectarías si atacante escaló privilegios usando sudo?"
**R:** "Verificaría `/var/log/auth.log` buscando patrones de uso sospechoso de sudo:
- Buscar intentos fallidos de sudo (atacante adivinando)
- Buscar comandos inusuales vía sudo (especialmente acceso shell)
- Buscar horarios inusuales (logins a las 3 AM)
- Buscar origen inusual de sudo (cuenta de servicio usando sudo no debería suceder)
Ejemplo: `grep sudo /var/log/auth.log | grep -v "sudo: root"`"

**P:** "Camina a través de investigar un proceso sospechoso que encontraste ejecutándose como root."
**R:** "1. Identificar el proceso: `ps aux | grep PROCESS`
2. Verificar qué está haciendo: `lsof -p PID` (archivos abiertos)
3. Verificar conexiones de red: `ss -tlnp | grep PID`
4. Encontrar cuándo empezó: verificar logs del sistema durante ese timeframe
5. Verificar quién lo inició: ver proceso padre y auth.log
6. Analizar el binario: `strings /path/to/binary` (encontrar IPs, URLs hardcodeadas)
7. Aislar y matar si es malicioso"

### Difícil (L3/Senior)

**P:** "Estás investigando servidor Linux comprometido. Usuario 'john' muestra uso de sudo a las 3 AM, pero john afirma que no se logó esa noche. Explica cómo es posible y cuáles son tus próximos pasos."
**R:** "Posibles explicaciones:
1. Compromiso de clave SSH—atacante tiene clave SSH de john, entonces se loga sin aviso de contraseña
2. Contraseña rota por fuerza bruta—1000+ intentos fallidos después un exitoso
3. Proceso padre comprometido—algo que john ejecuta tenía vulnerabilidad, atacante pivotó
4. Cuenta de john ya estaba comprometida hace días, atacante solo la está usando ahora

Próximos pasos:
- Verificar auth.log por método de login SSH (contraseña vs publickey)
- Si publickey, examinar ~/.ssh/authorized_keys para claves no autorizadas
- Verificar si clave SSH tiene tiempo de modificación diferente a esperado
- Revisar log de sudo completo para campo COMMAND—¿exactamente qué ejecutó atacante?
- Timeline: ¿cuándo fue la última verificación de contraseña/clave de john como segura?
- Cross-reference actividades legítimas de john—¿podemos excluir esas de investigación?
- Verificar si cuenta de john tiene automatización legítima que podría explicar actividad a las 3 AM"

**P:** "Describe escenario donde permisos de archivos son MÁS importantes para seguridad que en Windows y explica por qué."
**R:** "Escenario Linux: Servidor de base de datos ejecuta como usuario 'mysql'. Archivos de base de datos tienen permisos `-rw-r-----` (propiedad de mysql:mysql). Si atacante compromete aplicación web (ejecutándose como usuario www-data), NO PUEDEN leer archivos de base de datos incluso aunque ambos estén en mismo servidor.

Comparación Windows: ACLs de Windows son más complejos pero menos comúnmente entendidos. Muchos administradores defaultean a 'Everyone: Full Control' lo que negaba la protección.

Por qué importa: Los permisos de Linux son simples, visibles, y difíciles de eludir. Este principio de privilegio mínimo es por qué Linux es preferido para infraestructura crítica. El atacante tiene que:
1. Comprometer cuenta www-data
2. Escalar a usuario mysql
3. Encontrar credenciales
4. Conectar a base de datos

vs Windows donde ACLs mal configuradas podrían permitir acceso directo."

---

## 🔗 Cómo Esto Se Conecta Con Todo Lo Demás

- **Marco de Respuesta a Incidentes (NIST):** Investigación Linux sucede durante fase de Detección y Análisis. Estás recopilando evidencia de logs y análisis de procesos.
- **MITRE ATT&CK:** Técnicas específicas de Linux como "Escalada de Privilegios vía sudo" (T1548.003), "Inyección de Proceso" (T1055), "Crear o Modificar Proceso del Sistema" (T1543)
- **IDs De Evento Windows:** Linux no tiene IDs de Evento pero `/var/log/auth.log` sirve propósito similar para eventos de autenticación
- **Active Directory:** A menudo integrado con Linux vía LDAP/Kerberos. Usuarios de Linux pueden ser usuarios de AD. El compromiso puede afectar ambos sistemas.
- **Conceptos De Redes:** Entender puertos, conexiones, aislamiento de red es más fácil de rastrear en Linux (cada proceso visible con `ss`)
- **SIEM:** Sistemas SIEM recopilan logs de Linux, parsean auth.log, detectan patrones (logins fallidos, abuso de sudo)
- **Threat Hunting:** Búsqueda proactiva de procesos sospechosos, cambios de permisos, o uso inusual de sudo en sistemas Linux
- **Análisis Forense (DFIR):** Análisis forense de memoria y disco de Linux se basa en entender estructura de procesos, propiedad de archivos, y ubicaciones de log

---

## 💾 TL;DR Para Personas Ocupadas

| Concepto | Qué Es | Por Qué Importa En SOC |
|----------|--------|----------------------|
| **Usuarios Y Permisos** | Cada archivo tiene dueño y permisos (rwx) | Acceso de atacante limitado por permisos; cambios de permisos = evidencia |
| **Sudo** | Acceso temporal a root para usuarios autorizados | Vector de escalada de privilegios; logs de sudo prueban compromiso |
| **Permisos De Archivos (chmod)** | Leer/Escribir/Ejecutar para dueño/grupo/otros | Archivos maliciosos cambian permisos; patrones sospechosos = ataque |
| **/var/log/auth.log** | Log de autenticación y uso de sudo | Donde encuentras intentos de login, abuso de sudo, evidencia de escalada |
| **Procesos** | Programas en ejecución con dueño, PID, padre | Malware aparece como procesos inesperados; propiedad indica nivel de privilegio |
| **Claves SSH** | Criptografía asimétrica para autenticación | Claves comprometidas = acceso persistente sin intentos de contraseña en logs |
| **Servidores** | Sistemas Linux ejecutando infraestructura | La mayoría de servidores web, bases de datos, routers ejecutan Linux; investigación requiere acceso CLI |

---

## 📌 Realidad De Producción: Tu Primera Semana

**Lunes:** Tu gerente te muestra servidor Linux que fue hackeado.
```bash
ssh john@compromised-server.internal
$ ps aux
root  5432  87.3  45.1  524288 400000  ?  R  03:47  1:23  /tmp/miner
$ grep "sudo" /var/log/auth.log | tail -20
# Descubrir: atacante usó cuenta de john para ejecutar comandos sudo
```

**Martes:** Investigas CÓMO obtuvieron contraseña de john.
```bash
$ grep "john" /var/log/auth.log | grep "Failed"
# 2,347 intentos fallidos desde 203.0.113.50
$ grep "john" /var/log/auth.log | grep "Accepted"
# Finalmente: "Accepted password for john from 203.0.113.50"
# = Ataque de fuerza bruta exitoso
```

**Miércoles:** Verificas escalada de privilegios.
```bash
$ sudo -l  # ¿Qué puede john hacer con sudo?
User john may run the following commands on server:
    (ALL) ALL
# OH NO - john puede ejecutar CUALQUIER comando como root
$ cat /etc/sudoers.d/john
# ¿Cuándo se agregó? ¿Quién lo agregó?
```

**Jueves:** Contienen y recuperan.
```bash
# 1. Matar proceso de malware
$ sudo kill -9 5432

# 2. Remover john de sudoers
$ sudo visudo -r /etc/sudoers.d/john

# 3. Reiniciar contraseña de john
$ sudo passwd john

# 4. Remover claves SSH (si comprometidas)
$ rm ~/.ssh/authorized_keys
```

**Viernes:** Documentas y presentas hallazgos.
```
Causa Raíz: Contraseña débil de john rota por fuerza bruta vía SSH
Cronograma De Ataque: 3:47 AM atacante se logó, inmediatamente escaló a root vía sudo
Evidencia: 2,347 intentos SSH fallidos, después login exitoso, después sudo a root
Impacto: Atacante ejecutó minero de criptomonedas durante ~3 horas
Remediación: Reinicio de contraseña, rotación de clave SSH, remover permisos sudo excesivos
Prevención: Habilitar solo autenticación por clave SSH, 2FA, rate limiting en SSH
```

---

## 📌 Perspectiva De Trabajo Real: Lo Que Realmente Importa

Cuando te contraten como Analista Junior De SOC:

1. **Tu gerente dirá:** "Tenemos un servidor Linux que podría estar comprometido, investiga"
2. **NECESITAS saber:**
   - Cómo SSH en el servidor
   - Cómo verificar qué procesos están ejecutándose
   - Cómo leer `/var/log/auth.log` buscando evidencia de ataque
   - Cómo identificar escalada de privilegios vía sudo
   - Cómo reconocer malware (procesos inesperados, CPU alto, conexiones salientes)
   - Cómo documentar hallazgos

3. **NO necesitas:**
   - Escribir módulos de kernel de Linux
   - Construir sistema Linux desde cero
   - Entender cada distribución de Linux
   - Convertirte en administrador de sistemas Linux

4. **Lo que te hará ver competente:**
   - Encontrar cronograma de ataque en logs
   - Identificar cómo atacante escaló privilegios
   - Reconocer patrones sospechosos de procesos
   - Saber qué evidencia preservar
   - Explicar tus hallazgos claramente

---

## ❌ Errores Comunes De Entrevista

**Error 1: Confundir Linux con GNU**
Incorrecto: "Linux es el sistema operativo completo"
Correcto: "Linux es el kernel; herramientas GNU proporcionan utilidades de espacio de usuario"
En entrevistas: Solo llámalo "Linux" y estarás bien. La distinción pedante importa para arquitectos de sistemas, no analistas de SOC.

**Error 2: No Entender Login SSH Vs Acceso Shell**
Incorrecto: "Si login SSH tuvo éxito, atacante tiene shell"
Correcto: Login SSH confirma autenticación; atacante aún necesita acceso shell
En entrevistas: Di "SSH permite acceso remoto, pero necesitamos verificar qué comandos ejecutaron"

**Error 3: Obsesionarse Con Permisos**
Incorrecto: Intentar calcular permisos octales mentalmente durante entrevista
Correcto: Conocer los comunes (755, 644, 700) y entender el principio
En entrevistas: "rwxr-xr-x significa dueño puede hacer todo, otros solo pueden leer/ejecutar" es suficiente

---

## 📚 Lectura Adicional Y Recursos

**Lectura Esencial:**
- NIST SP 800-53 (Controles de Seguridad para sistemas Linux)
- Guía de Seguridad Ubuntu (gratis en línea)
- "Linux Basics for Hackers" por OccupyTheWeb (pragmático para SOC)

**Recursos De Práctica:**
- TryHackMe (desafíos Linux gratuitos y pagos)
- HackTheBox (similar a TryHackMe)
- Juegos Wargames OverTheWire (gratuito, excelente para aprender)
- Crea tu propia VM Ubuntu localmente y practica comandos

**Páginas De Manual (man) Clave Para Leer:**
- `man sudo`
- `man chmod`
- `man ls`
- `man ssh`
- `man tail` (para análisis de logs)

**Análisis De Logs Del Mundo Real:**
- Toma reportes de incidentes de seguridad e identifica evidencia de Linux
- Sigue estudios de casos DFIR en blogs (2 SecShop, SANS Pen Test, etc.)
- Lee postmortems en comentarios de HackerNews

---

## ⚠️ Palabras De Cierre

Linux no es "fácil de editar" como sistema operativo. Es **transparente**—puedes ver qué está corriendo, quién lo posee, y qué cambió. Esta transparencia es lo que lo hace poderoso para investigaciones de seguridad.

En tu carrera de SOC, Linux estará en todos lados: servidores web, bases de datos, balanceadores de carga, routers, aparatos de seguridad. Entender cómo funciona la seguridad de Linux (usuarios, permisos, logs) no es opcional—es fundamental.

La buena noticia: La seguridad de Linux es realmente más simple que Windows una vez entiendes los conceptos principales. Todo es un archivo. Los archivos tienen permisos. Los usuarios tienen acceso limitado. Los logs te dicen todo.

Domina estos, y estarás significativamente adelante de la mayoría de analistas junior.

---

*Última Actualización: 2024*
*Dificultad: L1-L2*
*Relevancia De Entrevista: ⭐⭐⭐⭐⭐*
*Aplicabilidad De Trabajo: Conocimiento Requerido Para Todos Los Roles De SOC*
*Aplicabilidad De Producción: Uso Diario En Respuesta A Incidentes*
