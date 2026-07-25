# 🔐 Fundamentos de Ciberseguridad: La Base de Todo Análisis y Defensa

## 📖 ¿Qué es Ciberseguridad?

Ciberseguridad es el conjunto de **prácticas, herramientas y procesos** diseñados para proteger los activos digitales de una organización contra accesos no autorizados, modificaciones maliciosas y pérdida de disponibilidad. Es la **base fundamental** sobre la cual se construyen análisis de seguridad, detección de amenazas y aseguramiento de servidores y sistemas.

En esencia:
> **Ciberseguridad = Protección de activos + Detección de amenazas + Respuesta a incidentes**

---

## 🎯 ¿Por Qué Necesita Un Profesional de SOC/Blue Team Esto?

### En el Mundo Real del SOC:
- **Todos los días**: Respondes alertas basadas en la Triada CIA que se ha roto
- **En entrevistas**: Te preguntarán sobre la diferencia entre prevención y detección
- **En producción**: Trabajarás con SIEM, Firewalls, AD y playbooks todos los días
- **En escalaciones**: Necesitarás explicar por qué cierto activo tiene mayor prioridad que otro

### Preguntas de Entrevista Que Recibirás:
1. *"¿Cuál es la diferencia entre prevención y detección?"*
2. *"¿Es más importante la Disponibilidad o la Integridad en un banco?"*
3. *"¿Cuál es el rol del SIEM vs el Firewall?"*
4. *"¿Por qué Active Directory es un activo crítico?"*
5. *"¿Cómo prioritizas qué defender primero?"*

---

## 🔍 El Concepto Desglosado

### **Parte 1: La Triada CIA - El Pilar Fundamental**

La **Triada CIA** define los tres objetivos de seguridad que SIEMPRE debes proteger:

#### **C = Confidencialidad (Confidentiality)**
**¿Qué es?** Solo las personas autorizadas pueden ver la información.

| Ejemplo | Breach |
|---------|--------|
| Base de datos de clientes | Hacker roba nombres, emails, números de tarjeta |
| Código fuente propietario | Competidor accede al código secreto |
| Emails ejecutivos | Filtración de comunicaciones privadas |

**Impacto:** Pérdida de reputación, multas regulatorias (GDPR), daño competitivo.

---

#### **I = Integridad (Integrity)**
**¿Qué es?** La información está completa, correcta y no ha sido modificada por personas no autorizadas.

| Ejemplo | Breach |
|---------|--------|
| Base de datos de finanzas | Hacker modifica $1M a $100M |
| Histórico médico | Registros falsos de medicamentos |
| Logs de auditoría | Atacante borra evidencia de su presencia |

**Impacto:** Decisiones de negocio equivocadas, violación de cumplimiento, pérdida financiera directa.

---

#### **A = Disponibilidad (Availability)**
**¿Qué es?** La información y sistemas están accesibles cuando se necesitan, sin interrupciones.

| Ejemplo | Breach |
|---------|--------|
| Servidor de e-commerce | Ataque DDoS derriba el sitio durante Black Friday |
| Hospital | Ransomware encripta sistemas de historia clínica |
| Banco | API de transferencias "offline" durante 6 horas |

**Impacto:** Pérdida de ingresos inmediata, vidas en riesgo, confianza del cliente erosionada.

---

#### **¿Cuál es Más Importante?**

**RESPUESTA CORRECTA: Depende del negocio**

| Industria | Prioridad #1 | Prioridad #2 | Por qué |
|-----------|-------------|-------------|--------|
| **Banco/Finanzas** | Integridad | Confidencialidad | Si modifican dinero, pierdes TODO. Disponibilidad se recupera. |
| **Hospital** | Disponibilidad | Integridad | Si el sistema cae, muere gente. Un dato incorrecto es menos grave. |
| **Agencia Gubernamental** | Confidencialidad | Integridad | Secretos filtrados = seguridad nacional en riesgo. |
| **E-commerce/Retail** | Disponibilidad | Integridad | Si caes en Black Friday, pierdes millones. La modificación de precios es detectable. |
| **Abogados/Consultoría** | Confidencialidad | Disponibilidad | Secretos de clientes filtrados = fin del negocio. |

**Concepto clave:** En un banco (finanzas), la **Integridad** es más crítica que la Disponibilidad. Aunque el servidor esté "offline" 2 horas, eso se tolera. Pero si alguien modificó números financieros SIN ser detectado, el daño es irreversible.

---

### **Parte 2: Activos - ¿Qué Protegemos?**

En cualquier infraestructura, hay dos tipos de activos:

#### **Activos de Valor**
Son los que contienen información crítica o representan directamente el negocio.

| Activo | Ejemplo | Por qué es crítico |
|--------|---------|-------------------|
| **Active Directory (AD)** | Servidor Windows con todas las identidades y permisos | Sin AD, no sabes quién es quién. Es el "corazón de identidades" |
| **Servidores de Almacenamiento (Storage)** | NAS con datos de finanzas, clientes, secretos | Contiene datos de valor incalculable |
| **Bases de Datos de Finanzas** | SQL Server con saldos, transacciones | Modificación = pérdida financiera directa |
| **Servidores de Email** | Exchange con comunicaciones ejecutivas | Contiene secretos comerciales y negociaciones |

**Regla de Oro:** Protege los activos de valor PRIMERO.

---

#### **Activos de Funcionamiento**
Son la infraestructura que HABILITA que todo funcione.

| Activo | Ejemplo | Por qué es importante |
|--------|---------|----------------------|
| **Router** | Cisco/Fortinet que dirige tráfico | Sin él, no hay conectividad |
| **Firewall** | Dispositivo que filtra tráfico | Primera línea de defensa |
| **Servidores DNS** | Resuelve nombres a IPs | Sin él, nadie accede a nada |
| **Servidores de Autoridad de Certificados (CA)** | Emite certificados SSL/TLS | Sin él, no hay conexiones seguras |

**Relación:** Los activos de funcionamiento PROTEGEN a los activos de valor.

---

### **Parte 3: Prevención vs Detección - Dos Estrategias Diferentes**

#### **Prevención (PROACTIVE) - Antes del Ataque**
**Objetivo:** Que el ataque nunca suceda.

| Técnica | Descripción | Ejemplo |
|---------|-------------|---------|
| **Hardening** | Reforzar sistemas eliminando lo innecesario | Desactivar puertos no usados en Firewall |
| **Parcheo** | Actualizar para cerrar vulnerabilidades conocidas | Instalar parches de Windows |
| **Análisis de Penetración** | NOSOTROS hacemos exploits para encontrar brechas | Red team intenta hackear a blue team |
| **WAF (Web Application Firewall)** | Filtra ataques contra aplicaciones web | Bloquea SQL Injection, XSS |
| **IDS (Intrusion Detection System)** | Detecta patrones de ataque conocidos | Descubre intentos de acceso anómalo |

**Realidad:** La prevención SIEMPRE falla en algún momento. Por eso necesitamos detección.

---

#### **Detección (REACTIVE) - Durante/Después del Ataque**
**Objetivo:** Encontrar lo que pasó por (el firewall/prevención).

| Técnica | Descripción | Ejemplo |
|---------|-------------|---------|
| **SIEM (Security Information & Event Management)** | Recolecta logs de TODAS partes y busca patrones maliciosos | Detecta 100 intentos fallidos de login en 1 minuto |
| **Análisis de Logs** | Buscar evidencia de compromiso | "Alguien accedió a finanzas.xlsx a las 3:47 AM desde IP rara" |
| **Búsqueda de Indicadores de Compromiso (IOCs)** | Buscar huellas dactilares del atacante | "Este hash de malware está en nuestra red" |
| **Correlación de Eventos** | Conectar eventos aparentemente no relacionados | "Login fallido + Cambio de permisos = patrón sospechoso" |

**Realidad:** Si esperas a prevención, sufres ataques. Si solo esperas a detección, el daño ya ocurrió. Necesitas AMBOS.

---

#### **Response (REACTIVE) - Después de Detectar**
**Objetivo:** Contener, investigar y remediar.

| Fase | Descripción | Tiempo |
|------|-------------|--------|
| **Contención** | Aislar sistemas comprometidos de la red | Primeros 15 minutos |
| **Investigación** | Determinar alcance, causa, evidencia | Primeras 2-4 horas |
| **Remediación** | Restaurar sistemas, cambiar credenciales | Horas/Días |
| **Post-Incidente** | Lessons learned, mejorar reglas | Después de resolver |

---

## ⚙️ ¿Qué DEBES Memorizar?

### **La Triada CIA en Orden Contextual**

```
¿Qué negocio es?         Prioridad de CIA
─────────────────────────────────────────
Banco                 → Integridad > Confidencialidad > Disponibilidad
Hospital              → Disponibilidad > Integridad > Confidencialidad
Abogados              → Confidencialidad > Integridad > Disponibilidad
E-commerce            → Disponibilidad > Integridad > Confidencialidad
```

**Truco Mnemotécnico:** Piensa en el PEOR escenario para cada industria:
- **Banco:** ¿Qué es peor? Que down 2 horas O que alguien robó $1M? → Integridad
- **Hospital:** ¿Qué es peor? Que el sistema falle O que un dato médico está mal? → Disponibilidad
- **Gobierno:** ¿Qué es peor? Que algo esté offline O que se filtren secretos? → Confidencialidad

---

### **Los Cuatro Pilares de Defensa**

```
1. IDENTIDAD (Active Directory)
   ├─ ¿Quién eres? (Autenticación)
   └─ ¿Qué puedes hacer? (Autorización)

2. SEGMENTACIÓN (Firewall + VLAN + DMZ)
   ├─ Firewall: ¿Permitir este tráfico?
   ├─ VLAN: Separar redes por departamento
   └─ DMZ: Zona desmilitarizada para servidores públicos

3. VISIBILIDAD (SIEM + Logs + Análisis)
   ├─ ¿Qué está pasando? (Logs en tiempo real)
   ├─ ¿Es normal? (Correlación)
   └─ ¿Es una amenaza? (Alerta)

4. DEFENSA (Antivirus + WAF + IDS/IPS)
   ├─ Antivirus: Detecta malware conocido
   ├─ WAF: Bloquea ataques contra aplicaciones web
   └─ IDS/IPS: Detecta/Bloquea patrones de ataque
```

---

### **Activos Que DEBES Proteger PRIMERO**

1. **Active Directory** - El corazón de identidades (Windows)
2. **Base de Datos de Finanzas** - Dinero en riesgo directo
3. **Servidor de Almacenamiento** - Datos de clientes y secretos
4. **Servidores de Email** - Comunicaciones ejecutivas

**Por qué AD primero:** Si AD se corrompe, no sabes quién es legítimo. Es como si desaparecieran todos los DNIs del país.

---

## 📚 ¿Qué DEBES Entender Profundamente?

### **Concepto 1: Active Directory es más que "credenciales"**

AD NO es solo una base de datos de contraseñas. Es un sistema de:

1. **Autenticación** - Verificar que eres quien dices ser
2. **Autorización** - Decidir qué puedes hacer (permisos)
3. **Auditoría** - Registrar quién hizo qué y cuándo

**Ejemplo real:** 
- User "Juan" intenta acceder a "Finanzas.xlsx"
- AD busca: ¿Juan existe? (Autenticación)
- AD verifica: ¿Grupo de Juan tiene permiso de lectura en Finanzas? (Autorización)
- AD registra: 14:32:15 - Juan accedió a Finanzas.xlsx (Auditoría)

Si AD falla, ninguno de estos tres pasos funciona.

---

### **Concepto 2: SIEM NO es lo mismo que Firewall**

| Característica | SIEM | Firewall |
|----------------|------|----------|
| **Cuándo actúa** | Después del tráfico (análisis post-facto) | ANTES/DURANTE el tráfico (tiempo real) |
| **Qué hace** | Analiza logs, busca patrones | Acepta/rechaza conexiones |
| **Velocidad** | Puede tener segundos de retraso | Instantáneo (microsegundos) |
| **Es bloqueante?** | NO (solo reporta) | SÍ (bloquea activamente) |
| **Ventaja** | Ve DENTRO de lo que pasó en los sistemas | Ve TODO lo que entra/sale |

**Analógía:** 
- **Firewall** = Guardia en la puerta (detiene a desconocidos)
- **SIEM** = Detective que revisa cámaras (encuentra qué pasó adentro)

Necesitas AMBOS.

---

### **Concepto 3: Prevención SIEMPRE Falla**

No existe defensa perfecta. Ejemplos reales:
- Firewall bloquea puertos, pero atacante entra por puerto 443 (HTTPS legítimo)
- Antivirus no conoce nuevo malware
- Patch Tuesday llega, pero hay 0-day no parcheado aún
- Admin viejo abre email de phishing

**Conclusión:** Asume que el atacante SIEMPRE logrará entrar. Tu trabajo es detectar cuándo lo hace.

---

### **Concepto 4: Playbooks son Algoritmos de Respuesta**

Un playbook NO es un documento teórico. Es un PASO A PASO que ejecutas cuando algo malo sucede.

**Ejemplo: Playbook de Ransomware**

```
TRIGGER: SIEM detecta múltiples archivos encriptados

PASO 1: Verificar
  ¿Es ransomware real o falso positivo?
  → Sí: Ir a Paso 2
  → No: Cerrar ticket

PASO 2: Escalar
  Avisar a SOC Lead y Incident Commander

PASO 3: Contener (CRÍTICO - máx 15 minutos)
  → Aislar máquina: Desconectarla de red (Ethernet + WiFi)
  → Aislar equipo: Si es servidor, apagarlo para preservar memoria

PASO 4: Investigar
  → ¿Cuánto tiempo lleva comprometido?
  → ¿Qué más se encriptó?
  → ¿Cómo entró?

PASO 5: Remediar
  → Restaurar desde backup
  → Cambiar contraseñas de admins
  → Parchear vulnerabilidad de entrada

PASO 6: Reportar
  → Documentar timeline
  → Calcular costo
  → Comunicar a ejecutivos
```

**Quién escribe playbooks:**
- Certificaciones: NIST, ISO 27001, CIS Controls
- Empresas de seguridad especializadas
- Tu propio SOC (basado en experiencia)

---

## 🚨 Defensa Práctica: Capas de Seguridad

### **Modelo OSI de Defensa (De Afuera para Adentro)**

```
CAPA 1: PERÍMETRO (Firewall, Proxy, WAF)
│
├─ Router Fronterizo: ¿Qué tráfico entra a mi red?
├─ Firewall: ¿Qué puertos son accesibles?
└─ WAF: ¿Qué ataques van a mis aplicaciones web?

CAPA 2: SEGMENTACIÓN (VLAN, DMZ, Microsegmentación)
│
├─ DMZ: Zona para servidores públicos (menos críticos)
├─ VLAN de Finanzas: Red separada para sistemas críticos
└─ Microsegmentación: Cada servidor en su propia zona

CAPA 3: IDENTIDAD (Active Directory, MFA, PAM)
│
├─ AD: ¿Quién eres y qué puedes hacer?
├─ MFA: ¿Realmente eres tú? (contraseña + token)
└─ PAM: Gestión de credenciales de administrador

CAPA 4: APLICACIÓN (Antivirus, EDR, IDS)
│
├─ Antivirus: ¿Hay malware conocido?
├─ EDR (Endpoint Detection & Response): ¿Comportamiento sospechoso?
└─ IDS: ¿Patrón de ataque conocido?

CAPA 5: VISIBILIDAD (SIEM, Logging, Análisis)
│
└─ SIEM: ¿Qué está pasando en TODA mi red?
```

**Concepto clave:** Si falla una capa, la siguiente lo detiene. Defensa en profundidad.

---

### **Ataques Comunes y Cómo Se Detectan**

| Ataque | Capa que Falla | Cómo Se Detecta |
|--------|----------------|-----------------|
| **DDoS** | Perímetro (Firewall) | Millones de paquetes de la misma IP |
| **Phishing + Ransomware** | Identidad (MFA) | EDR ve proceso sospechoso ejecutando encriptación |
| **SQL Injection** | Aplicación (WAF) | WAF ve patrón SQL en parámetro GET |
| **Escalación de Privilegios** | Identidad (AD) | SIEM ve user sin permisos accediendo a admin |
| **Movimiento Lateral** | Segmentación | Tráfico entre VLANs sin autorización |
| **Exfiltración de Datos** | Visibilidad (SIEM) | Archivos grandes saliendo a IP externa desconocida |

---

## ❌ Errores Comunes de Estudiantes (Y Sus Consecuencias Reales)

### **Error 1: "Firewall Lo Protege Todo"**

**Lo que piensan:** "Si tengo firewall, estoy seguro"

**La realidad:** 
- Firewall bloquea el PERÍMETRO, no lo que está DENTRO
- Atacante que ya está adentro = Firewall es inútil
- Necesitas SIEM para ver qué hacen los atacantes internos

**Consecuencia real:** Empleado descontento roba datos. Firewall no lo ve. SIEM detiene el robo en 10MB.

---

### **Error 2: "Confidencialidad es SIEMPRE el #1"**

**Lo que piensan:** "Proteger datos es lo más importante"

**La realidad:**
- En banco: Integridad > Confidencialidad (robo de dinero > fuga de datos)
- En hospital: Disponibilidad > Integridad (vidas > datos incorrectos)

**Consecuencia real:** Gastas millones en encriptación (confidencialidad), pero falta detección de modificación de datos (integridad). Banco pierde $50M en fraude.

---

### **Error 3: "AD es solo una base de datos de contraseñas"**

**Lo que piensan:** "AD = credenciales"

**La realidad:** AD también controla permisos, auditoría, políticas de grupo, delegación.

**Consecuencia real:** AD se corrompe. No sabes quién tiene acceso a qué. Hackers se camuflan como admins. Tarda 3 días en descubrirlo (y otro 3 en remediarlo).

---

### **Error 4: "Prevención es más importante que Detección"**

**Lo que piensan:** "Si prevengo bien, no necesito detectar"

**La realidad:** Prevención FALLA. Todos los días. La detección es tu red de seguridad.

**Consecuencia real:** 0-day entra (prevención falla). Sin SIEM (detección), atacante roba datos por 6 meses sin saberlo. Con SIEM, lo ves en 6 minutos.

---

### **Error 5: "Un playbook es solo un documento"**

**Lo que piensan:** "Leo el playbook cuando pase algo"

**La realidad:** Playbooks deben ser automatizados y practicados. Cuando pase algo real, no hay tiempo de leer.

**Consecuencia real:** Ransomware encrypta 1000 servidores. Tu team improvisa respuesta. Tarda 8 horas contener. Con playbook automatizado, 15 minutos.

---

## 🧪 Práctica / Análisis: Escenarios Reales

### **Escenario 1: Banco - Modificación de Saldo (Integridad)**

**Situación:**
Un hacker logra acceder a la base de datos de finanzas a través de SQL Injection. Modifica tu saldo de $1,000 a $100,000. El dinero nunca sale de la cuenta (aún no). ¿Qué defensa detecta esto PRIMERO?

**Opciones:**
A) Firewall  
B) SIEM analizando cambios en base de datos  
C) Antivirus  
D) Active Directory  

**Respuesta Correcta:** **B) SIEM**

**Por qué:** 
- Firewall: No lo ve (el tráfico es legítimo hacia la DB)
- SIEM: Detecta cambio anómalo en registro de DB ("UPDATE finanzas SET balance=100000 WHERE cuenta=Juan")
- AD: No lo ve (el usuario tiene permisos legítimos)
- Antivirus: Es legítimo, no hay malware

**Lección:** La integridad se rompe DENTRO de los sistemas. SIEM es crucial.

---

### **Escenario 2: Hospital - Ataque DDoS (Disponibilidad)**

**Situación:**
Un atacante lanza DDoS contra el servidor de historias clínicas. Recibe 1 millón de paquetes por segundo de IP 123.45.67.89. El sistema cae. ¿Qué lo detiene PRIMERO?

**Opciones:**
A) SIEM  
B) Firewall  
C) AD  
D) Antivirus  

**Respuesta Correcta:** **B) Firewall**

**Por qué:**
- Firewall: VE todos los paquetes. Detecta 1M paquetes de misma IP, los bloquea EN TIEMPO REAL
- SIEM: Lo ve DESPUÉS (retraso de segundos). Demasiado lento para DDoS
- AD: No ve tráfico de red
- Antivirus: No es malware

**Lección:** La disponibilidad se rompe en el PERÍMETRO. Firewall + IDS son lo primero.

---

### **Escenario 3: Startup Tech - Robo de Código (Confidencialidad)**

**Situación:**
Empleado descontento copia 10GB del repositorio privado (GitHub) a USB. Lo mete en su mochila. ¿Qué lo detecta?

**Opciones:**
A) Firewall  
B) SIEM analizando transferencia de datos  
C) DLP (Data Loss Prevention)  
D) Todas las anteriores  

**Respuesta Correcta:** **D) Todas las anteriores** (pero primero C)

**Por qué:**
- **DLP (primero):** Bloquea 10GB saliendo a USB. "Transferencia no permitida"
- **SIEM (segundo):** Registra intentos de acceso masivo a repo
- **Firewall (tercero):** Si intenta subirlo a cloud, lo bloquea
- **AD (contexto):** Registra que empleado descargó 10GB

**Lección:** La confidencialidad es un PROBLEMA PREVENTIVO primero (DLP), luego detectivo (SIEM).

---

### **Escenario 4: Movimiento Lateral - El Ataque Silencioso**

**Situación:**
1. Hacker entra a servidor web (fue por vulnerabilidad, firewall no lo supo)
2. Se mueve a servidor de Base de Datos en otra VLAN
3. Roba datos de clientes

¿Cuáles defesas lo detectan y en qué orden?

**Timeline correcta:**
```
Tiempo 0:00    → Hacekr entra a servidor web (WAF no lo ve, es puerto legítimo)
Tiempo 0:05    → SIEM detecta comportamiento anómalo (intenta directorios raros)
Tiempo 0:10    → Hacker intenta moverse a DB (otra VLAN)
Tiempo 0:11    → Firewall lo bloquea (regla: "Web to DB = Rechazar")
Tiempo 0:12    → SIEM genera alerta de movimiento lateral

DEFENSA CORRECTA: Microsegmentación (Firewall entre VLANs) + SIEM
```

**Lección:** El movimiento lateral es MUY peligroso. Necesitas segmentación + SIEM vigilante.

---

## 🎯 Preguntas de Entrevista Que Podrías Recibir

### **Nivel 1 (Fácil) - Candidato Junior**

**P1:** "¿Cuál es la diferencia entre Confidencialidad, Integridad y Disponibilidad?"

**Respuesta correcta:**
> "Confidencialidad significa que solo personas autorizadas ven los datos. Integridad significa que los datos no fueron modificados sin autorización. Disponibilidad significa que los sistemas están accesibles cuando se necesitan. Las tres forman la Triada CIA."

---

**P2:** "¿Cuál es el rol de Active Directory?"

**Respuesta correcta:**
> "Active Directory maneja identidades y permisos en redes Windows. Autentica (verifica quién eres), autoriza (decide qué puedes hacer) y audita (registra lo que hiciste)."

---

**P3:** "¿Qué es un SIEM?"

**Respuesta correcta:**
> "Un SIEM (Security Information & Event Management) recolecta logs de todos los sistemas, los traduce a un lenguaje universal de seguridad, busca patrones de ataque y genera alertas."

---

### **Nivel 2 (Intermedio) - Candidato Mid-Level**

**P4:** "¿Es más importante la Disponibilidad o la Integridad?"

**Respuesta correcta:**
> "Depende del negocio. En un banco, la Integridad es crítica porque la modificación fraudulenta de dinero es irreversible. En un hospital, la Disponibilidad es crítica porque las vidas están en riesgo. El contexto define la prioridad."

---

**P5:** "¿Cuál es la diferencia entre Prevención y Detección?"

**Respuesta correcta:**
> "Prevención evita que el ataque suceda (firewall, patches, hardening). Detección encuentra lo que se filtró por prevención (SIEM, logs, análisis). Ambas son necesarias porque prevención SIEMPRE falla en algún momento."

---

**P6:** "¿Por qué necesitamos Firewall SI tenemos SIEM?"

**Respuesta correcta:**
> "Firewall actúa en TIEMPO REAL en el perímetro, bloqueando ataques antes de que entren. SIEM analiza logs después, detectando qué logró entrar. Son capas diferentes. Firewall es prevención rápida. SIEM es detección profunda. Sin Firewall, todos los ataques logran entrar. Sin SIEM, no ves qué pasó dentro."

---

### **Nivel 3 (Difícil) - Senior/Lead**

**P7:** "Diseña la arquitectura de defensa para una startup financiera con 50 empleados y datos de 10,000 clientes."

**Respuesta correcta** (estructura):
```
1. PERÍMETRO:
   - Firewall con IDS/IPS
   - WAF para aplicaciones web
   - Proxy para inspeccionar tráfico saliente

2. SEGMENTACIÓN:
   - VLAN de Finanzas (datos críticos)
   - VLAN de Admin (servidores)
   - VLAN de Usuarios (desktop)
   - DMZ (servidores públicos)

3. IDENTIDAD:
   - Active Directory con MFA
   - PAM para credenciales de admin
   - Auditoría de todos los accesos

4. VISIBILIDAD:
   - SIEM recolectando logs de Firewall, AD, Servidores
   - Playbooks para respuesta automática
   - Dashboards de alertas críticas

5. DEFENSA EN PROFUNDIDAD:
   - Antivirus en endpoints
   - EDR para detección de comportamiento
   - Copias de seguridad (backup) de datos críticos
```

---

**P8:** "Un playbook ejecuta durante un incidente y falla. ¿Qué haces?"

**Respuesta correcta:**
> "Primero, ejecuto paso manual. Continúo respuesta. Después del incidente, hago post-mortem: ¿Por qué falló el playbook? ¿Fue error humano? ¿Información incorrecta? ¿Paso confuso? Actualizo el playbook, hago drill (simulacro), capacito al team. La próxima vez funciona. Los playbooks evolucionan con experiencia real."

---

**P9:** "¿Cómo prioritzas qué defender si solo tienes presupuesto para dos cosas?"

**Respuesta correcta:**
> "Primero: Identifico activos más críticos (Finanzas + AD). Segundo: Identifico impacto si se pierden (pérdida financiera + imposibilidad de operar). Tercero: Miro mi Triada CIA (¿cuál está en riesgo primero?). Cuarto: Dedico presupuesto a lo que protege AMBOS. Ejemplo: Un SIEM de clase empresarial detecta problemas en Integridad (finanzas) e Identidad (AD). Dos problemas, una solución."

---

## 🔗 Cómo Conecta Todo

### **Mapa Mental de Ciberseguridad Fundamental**

```
┌─────────────────────────────────────────────────────┐
│  TRIADA CIA = OBJETIVO                              │
│  (Confidencialidad, Integridad, Disponibilidad)    │
└──────────────┬──────────────────────────────────────┘
               │
        ┌──────┴──────┐
        │             │
    ¿QUÉ PROTEGES?   ¿CÓMO?
        │             │
    ACTIVOS      CAPAS DE DEFENSA
        │             │
    ┌───┴────┐    ┌───┴──────┬──────────┬─────────────┐
    │         │    │          │          │             │
  VALOR   FUNCIONAMIENTO  PERÍMETRO  SEGMENTACIÓN  IDENTIDAD
    │         │    │          │          │             │
  AD      Router  Firewall   VLAN      Active Dir.
  Data    Firewall WAF       DMZ       Permisos
  Email   DNS     Proxy      Micro-seg Auditoría
          CA              Segregación  MFA
                                        PAM

        ┌──────────────────────────┐
        │  ¿CÓMO DETECTAS SI FALLA?│
        └──────────┬───────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
      SIEM              EDR/Antivirus
      Logs              Comportamiento
      Alertas           Malware
      Playbooks         Incidentes
```

---

### **Relación: Prevención → Detección → Response**

```
PREVENCIÓN (Las defensa intenta bloquear)
    ├─ Firewall
    ├─ Patches
    ├─ Hardening
    └─ ⚠️ SIEMPRE FALLA

        ↓ Algo se filtró ↓

DETECCIÓN (Encuentras lo que entró)
    ├─ SIEM
    ├─ EDR
    ├─ IDS
    └─ ⚠️ GENERA ALERTA

        ↓ Alerta confirmada ↓

RESPONSE (Ejecutas el playbook)
    ├─ Contención (aislar)
    ├─ Investigación (entender)
    ├─ Remediación (arreglar)
    └─ ✅ VUELVES A LA NORMALIDAD

        ↓ Lessons Learned ↓

MEJORA
    ├─ Actualizar playbook
    ├─ Cambiar reglas SIEM
    ├─ Parchear vulnerabilidad
    └─ ✅ PREVENCIÓN MÁS FUERTE
```

---

## 💾 TL;DR Para Personas Ocupadas

### **La Tabla Rápida de Referencia**

| Concepto | Definición | Ejemplo | Crítico |
|----------|-----------|---------|---------|
| **CIA - Confidencialidad** | Solo autorizados ven datos | Robo de customer data | Gobierno, Abogados |
| **CIA - Integridad** | Datos no modificados sin autorización | Cambio fraudulento de saldo | Bancos, Finanzas |
| **CIA - Disponibilidad** | Sistemas accesibles cuando se necesitan | Servidor down en Black Friday | Hospitales, E-commerce |
| **Activo de Valor** | Contiene datos críticos del negocio | AD, Base datos finanzas | Proteger PRIMERO |
| **Activo de Funcionamiento** | Infraestructura que habilita todo | Router, Firewall, DNS | Proteger SEGUNDO |
| **Prevención** | Evitar que el ataque suceda | Firewall, Patches | Falla a veces |
| **Detección** | Encontrar qué se filtró por prevención | SIEM, Logs | SIEMPRE necesaria |
| **Playbook** | Paso a paso de respuesta a incidente | Ransomware response plan | Documentado + Automatizado |
| **Active Directory** | Control de identidades y permisos | Autenticar + Autorizar + Auditar | Corazón de Windows |
| **SIEM** | Recolecta logs, busca patrones, genera alertas | Detecta 100 logins fallidos en 1 min | Ojo de seguridad |
| **Firewall** | Acepta/rechaza tráfico según reglas | Bloquea conexión de puerto no permitido | Guardián de perímetro |

---

## 📌 Realidad en Producción

### **Lo Que REALMENTE Pasa en un SOC**

**Día 1:**
- 8:00 AM - SIEM genera alerta: "200 intentos fallidos de login en 15 minutos"
- 8:05 AM - ¿Falso positivo o ataque real?
- 8:10 AM - Verificas Active Directory: ¿Qué cuenta se está atacando? ¿Cuántos intentos fallidos?
- 8:15 AM - Confirmas: Es ataque real (40 intentos diferentes, misma IP externa)
- 8:20 AM - Ejecutas playbook: "Bloquea IP en Firewall"
- 8:21 AM - Cambias contraseña del admin (caso estés comprometido)
- 8:30 AM - SIEM confirma: Los intentos pararon después del bloqueo
- 8:45 AM - Investigas: ¿Cómo supieron el usuario? ¿Hay datos filtrados? Buscas en historiales de AD
- 9:00 AM - Escribes reporte: "Ataque de fuerza bruta bloqueado. Causa probable: Email comprometido. Acción: Cambiar contraseña."

**Lección:** Detección + Playbook bien ensayado = Respuesta en 30 minutos.

---

### **Errores de Producción Que Ves Todos Los Días**

1. **Playbook desactualizado** → "El AD ya no es SQL Server 2008" → Falla la investigación
2. **SIEM sin reglas de correlación** → Ves eventos aislados, no patrones → Pierdes ataques
3. **Firewall demasiado permisivo** → Todo pasa → Demasiado trabajo para SIEM
4. **AD sin auditoría habilitada** → No sabes qué cambió → Investigación imposible
5. **Equipo sin capacitación** → Playbook existe pero nadie sabe qué hacer → Caos

---

## 📚 Lecturas Adicionales Recomendadas

### **Para Principiantes:**
1. NIST Cybersecurity Framework (https://www.nist.gov/cyberframework)
2. CIS Controls v8 (https://www.cisecurity.org/cis-controls)
3. OWASP Top 10 (https://owasp.org/Top10/)

### **Para Nivel Intermedio:**
1. MITRE ATT&CK Framework (https://attack.mitre.org)
2. Incident Response & Recovery (NIST SP 800-61)
3. Active Directory Security Best Practices (Microsoft Docs)

### **Para Profundizar:**
1. Incident Response by Michael Ligh et al.
2. Blue Team Handbook by Don Murdoch
3. Defensive Security Handbook by Lee, Martin, Neifer

---

## 🎓 Resumen: Lo Que Aprendiste

✅ **La Triada CIA no es estática** - Depende del contexto (Banco vs Hospital)

✅ **Activos de Valor vs Funcionamiento** - Protege valor PRIMERO, luego infraestructura

✅ **Prevención SIEMPRE falla** - Por eso detección es CRÍTICA

✅ **SIEM NO es Firewall** - Diferentes capas, diferentes roles

✅ **Active Directory es el corazón** - Si muere, todo se va abajo

✅ **Playbooks son algoritmos vivos** - Evolucionan con experiencia real

✅ **Microsegmentación salva vidas** - Movimiento lateral es LETAL

✅ **Contexto es rey** - Integridad vs Disponibilidad depende del negocio

---

**Próximo paso:** Domina estos fundamentos. TODO lo demás en ciberseguridad se construye sobre esto.

Vuelve a leer esto antes de cualquier entrevista. 🔐

---

*Documento generado para estudiantes de TripleTen Blue Team | Última actualización: 2026*
