# Framework de Respuesta a Incidentes (NIST SP 800-61)

> **Nota:** Debido al límite de longitud por interacción, este documento
> contiene el inicio de la traducción. En las siguientes partes se
> continuará hasta completar el documento íntegramente manteniendo la
> estructura y formato Markdown.

## 📖 ¿Qué es esto? (En 30 segundos)

El Framework de Respuesta a Incidentes es la metodología estructurada
que siguen las organizaciones cuando ocurre un incidente de seguridad.
NIST SP 800-61 define **cuatro fases** que guían a los investigadores
desde el momento en que se detecta un incidente hasta que se extraen las
lecciones aprendidas y los sistemas son restaurados. Piénsalo como el
**playbook** que indica qué hacer, en qué orden hacerlo y cuándo escalar
la situación.

------------------------------------------------------------------------

## 🎯 ¿Por qué un Analista SOC necesita conocer esto?

### En entrevistas de trabajo escucharás preguntas como:

-   «Explícame el proceso de respuesta a incidentes desde la detección
    hasta la recuperación.»
-   «¿Cuál es la diferencia entre contención y erradicación?»
-   «¿Cuándo escalarías un incidente? ¿A quién?»
-   «¿Cómo preservas la evidencia mientras detienes un ataque?»
-   «¿Cuál es tu metodología para investigar incidentes?»

### Durante tu primer mes en un SOC:

- Recibirás una alerta y pensarás: **"¿Y ahora qué?"**
- Tu SIEM generará una alerta. Seguirás el framework.
- Tendrás que tomar decisiones de contención que pueden afectar al negocio.
- Preservarás evidencia mientras respondes a ataques.
- Reportarás tus hallazgos a la dirección utilizando esta estructura.
- Participarás en revisiones posteriores al incidente.
- **Hacer esto mal = perder evidencia, propagar la infección o dejar escapar al atacante.**

---

# 🔍 Las Cuatro Fases (Explicadas)

## **FASE 1: PREPARACIÓN**

### ¿Qué es?

Antes de que ocurra un incidente, debes prepararte. Este es el trabajo realizado en tiempos de calma que determinará si tendrás éxito cuando llegue una crisis.

### ¿Qué haces?

- **Construir el equipo:** Capacitar a los analistas, definir funciones y establecer rutas de escalamiento.
- **Implementar herramientas:** SIEM, EDR, infraestructura de registros (logging), firewalls y herramientas de monitoreo.
- **Crear playbooks:** Procedimientos paso a paso para responder a ataques comunes.
- **Establecer políticas:** Cómo preservar evidencia, quién tiene autoridad para tomar decisiones y cómo comunicar incidentes.
- **Probar todo:** Ejecutar simulacros de respuesta a incidentes, probar respaldos y validar los canales de comunicación.
- **Documentar líneas base (Baselines):** Conocer exactamente cómo luce un comportamiento normal dentro de la infraestructura.

### ¿Por qué es importante?

Si no existe preparación, cuando ocurra un incidente todo será improvisación.

Los atacantes no esperarán mientras decides qué hacer.

La velocidad y calidad de la respuesta dependen casi por completo del trabajo realizado antes del incidente.

### Ejemplo en un SOC real

**Lunes**
- Se instala un nuevo SIEM.
- Se configuran reglas de detección.

**Martes**
- Se crea un playbook:
  > "Si observamos más de 1000 intentos fallidos de autenticación desde una misma dirección IP, realizar X procedimiento."

**Jueves**

La alerta finalmente ocurre.

En lugar de improvisar, simplemente sigues el playbook.

El proceso funciona exactamente como fue diseñado.

### 🚩 Señal de alerta

Si una organización no ha invertido tiempo en la fase de preparación, normalmente los incidentes tardan **tres veces más** en resolverse.

---

## **FASE 2: DETECCIÓN Y ANÁLISIS**

### ¿Qué es?

Un incidente es detectado mediante una alerta, un usuario o una herramienta de seguridad.

A partir de ese momento debes investigar para determinar **qué ocurrió realmente**.

### ¿Qué haces?

## DETECCIÓN
*(Alguien detecta que algo no está bien)*

- Una herramienta de seguridad genera una alerta.
- Un administrador reporta actividad sospechosa.
- Un usuario informa que perdió acceso a su cuenta.
- El firewall bloquea tráfico sospechoso.
- El antivirus pone malware en cuarentena.

## ANÁLISIS
*(Investigas para confirmar el incidente)*

### 1. Realizar el triage de la alerta

- ¿Es un **True Positive** o un **False Positive**?
- ¿Qué nivel de criticidad tiene?
  - Baja
  - Media
  - Alta
  - Crítica
- ¿Coincide con un patrón de ataque conocido?

### 2. Recolectar evidencia inicial

- Obtener logs del sistema afectado.
- Capturar tráfico de red.
- Construir una línea temporal.
- Identificar activos comprometidos.
- Identificar usuarios afectados.

### 3. Determinar el alcance

- ¿Está limitado a un único equipo?
- ¿Afecta a toda la red?
- ¿Cuántos sistemas fueron comprometidos?
- ¿Desde cuándo ocurre?

### 4. Clasificar el incidente

**¿Malware?**
- Analizar comportamiento y firmas antivirus.

**¿Acceso no autorizado?**
- Revisar logs de autenticación y elevaciones de privilegios.

**¿Exfiltración de datos?**
- Analizar grandes volúmenes de tráfico hacia direcciones IP externas.

**¿Denegación de servicio?**
- Analizar patrones de tráfico.

**¿Amenaza interna?**
- Revisar comportamiento del usuario y acceso a datos sensibles.

### 5. Mapear contra MITRE ATT&CK

- ¿Qué técnicas está utilizando el atacante?
- Esto ayuda a predecir cuál será el siguiente movimiento del adversario.

## Decisiones críticas durante esta fase

- ¿Debemos escalar el incidente?
  - Sí, si se confirma actividad maliciosa.

- ¿Debemos preservar evidencia?
  - Siempre.

- ¿Debemos aislar el sistema?
  - Todavía no.
  - Eso pertenece a la siguiente fase: **Contención**.

## Errores comunes

❌ Actuar demasiado rápido sin confirmar el incidente.

❌ No documentar los hallazgos.

❌ Eliminar registros antes de analizarlos.

❌ Asumir que una sola alerta representa todo el ataque.

## Ejemplo en un SOC real

**10:35 AM**

El SIEM genera la alerta:

> **20 intentos fallidos de autenticación en CORP-SQL-SERVER desde 192.168.1.105**

### Tus acciones

1. Verificas si 192.168.1.105 corresponde a un servidor conocido.
   - No.
   - Es sospechoso.

2. Identificas qué usuario estaba siendo atacado.
   - SA_Admin.
   - Muy sospechoso.

3. Construyes la línea temporal.
   - Inicio: 9:47 AM.
   - Lleva 48 minutos de actividad.

4. Revisas el alcance.
   - Descubres que existen otros tres servidores bajo ataque.

5. Clasificas el incidente.
   - Posible **Brute Force** o **Password Spraying**.

6. Lo mapeas contra MITRE ATT&CK.

- T1110 — Brute Force
- T1110.003 — Password Spraying

7. Tomas la decisión.

**ESCALAR AL ANALISTA SENIOR.**

El incidente ha sido confirmado como actividad maliciosa.
