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

---

# **FASE 3: CONTENCIÓN, ERRADICACIÓN Y RECUPERACIÓN**

Esta fase está compuesta por **tres subfases** que deben ejecutarse en orden.

Comprender la diferencia entre ellas es una de las habilidades más importantes para cualquier Analista SOC.

---

# SUBFASE 3A: CONTENCIÓN

## ¿Qué es?

**DETENER EL DAÑO.**

El objetivo de la contención es impedir que el atacante continúe causando impacto mientras se preserva la evidencia necesaria para la investigación.

## Estrategia de Contención

```text
Contención a Corto Plazo (0–30 minutos)
↓
Detener la propagación y preservar evidencia

Contención a Mediano Plazo (30 minutos – varias horas)
↓
Aislar sistemas comprometidos y bloquear comunicaciones del atacante

Contención a Largo Plazo (horas – días)
↓
Fortalecer defensas y monitorear mecanismos de persistencia
```

---

## ¿Cómo funciona la contención?

### Paso 1. Aislar el sistema comprometido

- Desconectar el equipo de la red (manteniéndolo encendido para preservar la memoria RAM).
- O aplicar políticas de red que limiten el movimiento lateral.
- Objetivo:
  - Evitar que el atacante comprometa otros sistemas.

---

### Paso 2. Preservar la evidencia

- Obtener un volcado de memoria (Memory Dump).
- Crear una copia forense del disco antes de modificar cualquier archivo.
- Documentar:
  - Capturas de pantalla.
  - Logs.
  - Hora exacta de cada acción.
- Mantener la **Cadena de Custodia (Chain of Custody)** indicando:
  - Quién manipuló la evidencia.
  - Cuándo.
  - Qué acciones realizó.

---

### Paso 3. Bloquear la comunicación del atacante

- Finalizar conexiones remotas si el atacante tiene acceso activo.
- Bloquear las direcciones IP del servidor C2 (Command & Control) en el firewall.
- Restablecer credenciales comprometidas.
- **Importante:** si la investigación aún está en curso, algunas acciones pueden esperar para no alertar al atacante prematuramente.

---

### Paso 4. Mantener la integridad del sistema

No debes:

- Reiniciar el sistema.
- Eliminar archivos.
- Borrar registros.
- Aplicar parches todavía.

Durante la contención, el sistema debe permanecer en el estado más cercano posible al momento del incidente para facilitar el análisis forense.

---

## La decisión crítica durante la contención

Debes responder una pregunta:

> **¿Cuánto daño adicional puede causar el atacante en este preciso momento?**

Si el riesgo es muy alto:

- Desconectar completamente el sistema.

Si el riesgo es bajo:

- Aplicar restricciones de red mientras se continúa recolectando evidencia.

---

## Ejemplo en un SOC real

**11:15 AM**

Se confirma que **CORP-LAPTOP-203** está infectado con un troyano.

### Acciones

✅ Obtener un Memory Dump.

✅ Desconectar el equipo de la red.

✅ Mantener el sistema encendido.

✅ Bloquear la IP del servidor C2 en el firewall.

✅ Obligar al usuario a cambiar su contraseña.

✅ Documentar todas las acciones en el ticket del incidente.

⏸ No reiniciar el equipo.

⏸ No ejecutar todavía el antivirus.

### Resultado

- El atacante ya no puede seguir actuando.
- La evidencia permanece intacta.
- La investigación continúa.

---

# SUBFASE 3B: ERRADICACIÓN

## ¿Qué es?

**ELIMINAR COMPLETAMENTE AL ATACANTE.**

Después de detener el ataque, ahora debes eliminar toda su presencia del sistema.

---

## ¿Cómo funciona la erradicación?

### Paso 1. Eliminar el malware

- Ejecutar análisis antivirus.
- Eliminar archivos maliciosos.
- Remover tareas programadas creadas por el malware.
- Eliminar puertas traseras (Backdoors).
- Desinstalar herramientas de acceso remoto utilizadas por el atacante.

---

### Paso 2. Corregir la vulnerabilidad

- Aplicar parches al sistema operativo.
- Actualizar aplicaciones vulnerables.
- Modificar reglas del firewall.
- Actualizar controles de acceso.

Ejemplo:

Si el atacante explotó una vulnerabilidad en SQL Server, ahora es el momento de instalar el parche correspondiente.

---

### Paso 3. Eliminar mecanismos de persistencia

Los atacantes rara vez dejan un único punto de acceso.

Debes buscar:

- Scheduled Tasks.
- Servicios maliciosos.
- Claves de Registro.
- Rootkits.
- Bootkits.
- Usuarios ocultos.
- Backdoors.

El objetivo es garantizar que el atacante no pueda regresar.

---

### Paso 4. Verificar la erradicación

- Ejecutar nuevos análisis antivirus.
- Revisar nuevamente los logs.
- Confirmar que no existen conexiones sospechosas.
- Verificar que la vulnerabilidad fue corregida.

---

# Diferencia entre Contención y Erradicación

| Contención | Erradicación |
|------------|--------------|
| Detiene el ataque | Elimina el ataque |
| Preserva evidencia | La evidencia ya puede modificarse |
| El sistema sigue comprometido | El sistema queda limpio |
| Puede revertirse | Es definitiva |
| Debe hacerse rápidamente | Requiere más tiempo |
| Ejemplo: Desconectar un equipo | Ejemplo: Eliminar malware y aplicar parches |

---

## Errores comunes

❌ Eliminar registros antes de entender cómo ocurrió el ataque.

❌ Aplicar parches antes del análisis forense.

❌ No buscar mecanismos de persistencia.

❌ Creer que reiniciar el equipo elimina completamente el malware.

---

## Ejemplo en un SOC real

**3:30 PM**

La contención ha finalizado.

Comienza la erradicación.

### Acciones

✅ Ejecutar antivirus.

Se detecta y elimina el troyano.

✅ Buscar persistencia.

Se encuentra una Scheduled Task que ejecuta malware.exe diariamente.

✅ Eliminar la tarea programada.

✅ Revisar el Registro de Windows.

Se eliminan entradas maliciosas.

✅ Aplicar el parche de SQL Server.

✅ Ejecutar un nuevo escaneo.

No se detectan amenazas.

✅ Documentar:

> Erradicación completada correctamente.

### Resultado

El sistema está limpio.

Las herramientas del atacante fueron eliminadas.

La vulnerabilidad ya fue corregida.

7. Tomas la decisión.

**ESCALAR AL ANALISTA SENIOR.**

---

# SUBFASE 3C: RECUPERACIÓN

## ¿Qué es?

**DEVOLVER LOS SISTEMAS A OPERACIÓN.**

Una vez que el atacante ha sido eliminado, comienza el proceso de restaurar los sistemas para que vuelvan a funcionar de forma segura y controlada.

El objetivo no es únicamente que los usuarios puedan volver a trabajar, sino garantizar que el entorno permanezca limpio y estable.

---

## ¿Cómo funciona la recuperación?

### Paso 1. Restaurar respaldos limpios

- Utilizar un respaldo realizado **antes** de la infección.
- Restaurar archivos del usuario.
- Restaurar configuraciones.
- Restaurar aplicaciones.
- Verificar que el respaldo no contenga malware.

---

### Paso 2. Reconstruir el sistema (si es necesario)

Si no existe un respaldo confiable:

- Reinstalar completamente el sistema operativo.
- Aplicar todos los parches de seguridad.
- Instalar nuevamente las aplicaciones.
- Restaurar únicamente información proveniente de respaldos limpios.

**Nunca sacrifiques la seguridad por rapidez.**

---

### Paso 3. Restaurar conectividad y acceso

- Reconectar el sistema a la red.
- Restaurar reglas de firewall.
- Restablecer permisos de usuarios.
- Habilitar nuevamente los servicios necesarios.

Sin embargo...

El monitoreo debe continuar.

---

### Paso 4. Validar el funcionamiento

Comprobar que:

- El sistema opera normalmente.
- Los usuarios pueden acceder a sus archivos.
- Las aplicaciones funcionan correctamente.
- No existen errores.
- No existen problemas de rendimiento.

---

### Paso 5. Monitoreo posterior a la recuperación

Durante varios días o semanas debes vigilar:

- Intentos de reinfección.
- Tráfico sospechoso.
- Nuevos intentos de autenticación.
- Indicadores de persistencia.

La recuperación no termina cuando el usuario vuelve a iniciar sesión.

---

## La decisión crítica durante la recuperación

Debes responder:

> **¿Restauramos desde un respaldo o reconstruimos completamente el sistema?**

Si existe la posibilidad de que el respaldo también esté comprometido:

- Reconstruir el sistema desde cero.

Si el respaldo es confiable:

- Restaurar desde dicho respaldo.

Aunque reconstruir sea más lento, suele ser la alternativa más segura.

---

## Ejemplo en un SOC real

**5:00 PM**

La erradicación ha sido completada.

Comienza la recuperación.

### Acciones

✅ Verificar el respaldo realizado el 15 de marzo.

La infección ocurrió el 17 de marzo.

El respaldo es limpio.

✅ Restaurar el sistema.

✅ Instalar las actualizaciones de seguridad más recientes.

✅ Restaurar los archivos del usuario.

✅ Reconectar el sistema a la red.

✅ El usuario valida el funcionamiento.

Todo opera correctamente.

✅ Habilitar monitoreo reforzado durante 30 días.

### Resultado

El sistema vuelve a producción.

El usuario puede trabajar nuevamente.

La organización mantiene un monitoreo activo para detectar cualquier intento de reinfección.

---

# ¿Por qué son importantes las tres subfases?

```text
CONTENCIÓN
↓
Detener el ataque

ERRADICACIÓN
↓
Eliminar completamente al atacante

RECUPERACIÓN
↓
Restaurar las operaciones normales
```

Cada una tiene un objetivo distinto.

Si omites una de ellas:

❌ Sin Contención

- El atacante continúa propagándose.

❌ Sin Erradicación

- El malware volverá a ejecutarse después de restaurar el sistema.

❌ Sin Recuperación

- Los usuarios no podrán volver a trabajar.

---

# **FASE 4: ACTIVIDADES POSTERIORES AL INCIDENTE (POST-INCIDENT ACTIVITY / LESSONS LEARNED)**

## ¿Qué es?

Cuando el incidente ha sido resuelto, comienza una fase igual de importante:

**Aprender de lo ocurrido.**

El objetivo es comprender:

- Qué ocurrió.
- Por qué ocurrió.
- Cómo respondió el equipo.
- Qué debe mejorarse para evitar futuros incidentes.

---

## ¿Qué haces?

### Paso 1. Obtener reportes del equipo de infraestructura

Verificar:

- Todos los cambios realizados durante la respuesta.
- Estado operativo de los sistemas.
- Que no existan equipos comprometidos.
- Que los respaldos funcionen correctamente.

---

### Paso 2. Revisar nuevas herramientas implementadas

Durante la respuesta pueden haberse desplegado nuevas herramientas.

Por ejemplo:

- Nuevas reglas del SIEM.
- Nuevas reglas del Firewall.
- Agentes EDR.
- Nuevas políticas de monitoreo.

Debes comprobar que:

- Funcionan correctamente.
- Generan alertas útiles.
- No afectan la operación del negocio.

---

### Paso 3. Realizar Threat Hunting (si es necesario)

Preguntas importantes:

- ¿El atacante permanecía dentro de la red desde antes?
- ¿Comprometió otros equipos?
- ¿Existe evidencia de movimientos laterales?
- ¿Hay indicadores de compromiso adicionales?

El Threat Hunting busca amenazas que aún no han sido detectadas automáticamente.

---

### Paso 4. Reunión de Lessons Learned

El equipo responde preguntas como:

### ¿Qué ocurrió?

Construcción objetiva de la línea temporal.

---

### ¿Por qué ocurrió?

Identificación de la causa raíz.

---

### ¿Qué hicimos bien?

Analizar qué procedimientos funcionaron correctamente.

---

### ¿Qué hicimos mal?

Detectar errores del proceso.

---

### ¿Qué debemos mejorar?

Definir acciones concretas para futuros incidentes.

---

### ¿Quién necesita conocer esta información?

- Dirección.
- Cumplimiento (Compliance).
- Equipos de TI.
- Seguridad.
- Área Legal.

Según el impacto del incidente.

---

### Paso 5. Documentar recomendaciones

Ejemplos:

- Cambios de políticas.
- Implementación de nuevas herramientas.
- Capacitación para usuarios.
- Nuevas reglas del firewall.
- Nuevas reglas del SIEM.
- Mejoras a los playbooks existentes.

---

### Paso 6. Actualizar los Playbooks

Cada incidente debe mejorar el proceso.

Preguntas importantes:

- ¿Qué tomó demasiado tiempo?
- ¿Qué fue confuso?
- ¿Qué puede automatizarse?
- ¿Qué funcionó especialmente bien?

Toda esa información debe incorporarse al playbook correspondiente.

---

## Decisiones críticas durante la fase posterior al incidente

Al finalizar un incidente, aún existen decisiones importantes por tomar:

- **¿Debemos realizar un reporte de cumplimiento?**
  - PCI DSS
  - HIPAA
  - GDPR
  - ISO 27001
  - u otras regulaciones aplicables.

- **¿Debe intervenir el área legal?**
  - Especialmente si existió una posible fuga de información.

- **¿Es necesario notificar a los usuarios o clientes?**
  - Si sus datos fueron comprometidos.

- **¿Qué información debe recibir la dirección?**
  - Impacto técnico.
  - Impacto operativo.
  - Impacto financiero.
  - Riesgos futuros.

---

## Ejemplo en un SOC real

**18 de marzo — 10:00 AM**

El incidente ha sido completamente resuelto.

Es momento de realizar la revisión posterior al incidente.

### Acciones

✅ Obtener el estado de infraestructura.

Todos los sistemas funcionan correctamente.

---

✅ Revisar las nuevas reglas implementadas en el firewall.

Se confirma que bloquean correctamente las comunicaciones con el servidor C2.

---

✅ Realizar Threat Hunting.

Se descubren dos intentos adicionales de movimiento lateral.

Ambos fueron contenidos exitosamente.

---

✅ Reunión de Lessons Learned.

### ¿Qué ocurrió?

Un usuario abrió un correo de phishing y descargó un troyano.

### ¿Por qué ocurrió?

El filtro de correo electrónico no detectó el mensaje malicioso.

### ¿Qué funcionó bien?

- El SIEM detectó rápidamente la actividad.
- La contención fue inmediata.
- La recuperación tomó únicamente dos horas.

### ¿Qué falló?

El sistema de filtrado de correo necesitaba mejores reglas.

### Recomendaciones

- Fortalecer el filtrado de correo.
- Capacitación obligatoria sobre Phishing para todos los empleados.
- Implementar protección adicional en los endpoints.

---

### Actualización del Playbook

El playbook de respuesta ante troyanos ahora incluye procedimientos específicos para ataques que evaden los filtros de correo electrónico.

### Resultado

El equipo aprende de la experiencia.

Los procesos mejoran.

El próximo incidente podrá resolverse con mayor rapidez y eficiencia.

---

# ⚙️ Lo que DEBES Memorizar

## Las cuatro fases (en orden)

1. **PREPARACIÓN**
   - Construcción de herramientas.
   - Capacitación del equipo.
   - Creación de playbooks.

2. **DETECCIÓN Y ANÁLISIS**
   - Detectar.
   - Confirmar.
   - Investigar.

3. **CONTENCIÓN → ERRADICACIÓN → RECUPERACIÓN**
   - Detener.
   - Eliminar.
   - Restaurar.

4. **ACTIVIDADES POSTERIORES AL INCIDENTE**
   - Aprender.
   - Mejorar.
   - Documentar.

---

## Regla mnemotécnica

```text
PREPARAR
        ↓
DETECTAR
        ↓
CONTENER
        ↓
ERRADICAR
        ↓
RECUPERAR
        ↓
REVISAR
```

O más fácil aún:

```text
Planificar
↓
Detectar
↓
Contener
↓
Erradicar
↓
Recuperar
↓
Revisar
```

---

## Las tres subfases que nunca debes confundir

### Contención

Detiene el ataque.

Preserva evidencia.

---

### Erradicación

Elimina completamente al atacante.

---

### Recuperación

Restaura el funcionamiento normal del negocio.

---

# 📚 Lo que DEBES Comprender

No basta con memorizar las fases.

Debes entender:

- Por qué el orden es importante.
- La diferencia entre Contención y Erradicación.
- Cuándo preservar evidencia.
- Cuándo escalar un incidente.
- Cómo afecta cada fase al negocio.
- Cómo planificar una recuperación segura.
- Cómo mejorar continuamente mediante la revisión posterior al incidente.

---

# 🚨 Aplicación en el mundo real: Cómo se conectan las fases

## Escenario: Infección por Ransomware

### PREPARACIÓN (El mes pasado)

- Se implementó una herramienta de detección de ransomware.
- Se creó un playbook específico.
- El equipo fue capacitado.
- Se probaron los respaldos.

---

### DETECCIÓN Y ANÁLISIS (Hoy — 8:00 AM)

El antivirus detecta ransomware en **CORP-FINANCE-SVR**.

El analista confirma:

- 150 archivos cifrados.
- Nota de rescate presente.

Alcance:

- Un único servidor comprometido.

Clasificación:

**Incidente Crítico.**

---

### CONTENCIÓN (8:30 AM)

- Aislar el servidor.
- Obtener memoria RAM.
- Obtener imagen del disco.
- Bloquear comunicaciones C2.
- Mantener el servidor encendido.

**Decisión:**

No pagar el rescate.

---

### ERRADICACIÓN (10:00 AM)

- Ejecutar análisis antimalware.
- Eliminar ransomware.
- Buscar persistencia.
- Aplicar el parche correspondiente.
- Confirmar que no quedan indicadores de compromiso.

---

### RECUPERACIÓN (1:00 PM)

- Restaurar desde un respaldo limpio.
- Verificar que el respaldo no contiene ransomware.
- Restaurar archivos.
- Ejecutar nuevamente el antivirus.
- Reconectar el servidor.
- Validar junto al área financiera.

---

### POST-INCIDENT (Al día siguiente — 9:00 AM)

- Verificar estado de la infraestructura.
- Realizar Threat Hunting.
- Reunión de Lessons Learned.

Conclusiones:

**Funcionó bien**

- Detección rápida.
- Contención inmediata.
- Recuperación exitosa.

**Falló**

La vulnerabilidad explotada no había sido parchada.

**Recomendaciones**

- Automatizar el proceso de Patch Management.
- Validar respaldos mensualmente.
- Capacitación continua contra ransomware.

Actualizar el playbook incorporando las lecciones aprendidas.

---

Tiempo total de recuperación:

**24 horas**, no semanas.

---

El incidente ha sido confirmado como actividad maliciosa.

# ❌ Errores comunes que cometen los estudiantes

## Error 1: Pensar que las fases son completamente secuenciales

### Incorrecto

> "Terminé la fase de Preparación y ahora siempre estaré en Detección."

### Correcto

La preparación nunca termina.

Incluso mientras respondes a un incidente, continúas mejorando:

- Herramientas.
- Procedimientos.
- Playbooks.
- Capacitación.

Cada incidente hace que la organización esté mejor preparada para el siguiente.

---

## Error 2: Confundir Contención con Erradicación

### Incorrecto

> "Contener y erradicar significan lo mismo."

### Correcto

No.

**Contención**

- Detiene el ataque.
- El sistema continúa comprometido.
- La evidencia se preserva.

**Erradicación**

- Elimina completamente al atacante.
- El sistema queda limpio.
- Ya pueden realizarse cambios permanentes.

### ¿Por qué importa?

Si intentas eliminar el malware antes de detener al atacante, éste puede seguir propagándose mientras limpias el sistema.

---

## Error 3: Destruir evidencia demasiado pronto

### Incorrecto

> "Reinicio el equipo y el problema desaparece."

### Correcto

Reiniciar destruye evidencia muy importante.

Por ejemplo:

- Procesos activos.
- Conexiones de red.
- Malware residente únicamente en memoria.
- Claves de cifrado.
- Sesiones del atacante.

Primero se recolecta evidencia.

Después se realizan modificaciones.

### Consecuencias reales

- Incumplimiento regulatorio.
- Investigación incompleta.
- Imposibilidad de reconstruir el ataque.
- El atacante podría quedar sin ser identificado.

---

## Error 4: No documentar durante la respuesta

### Incorrecto

> "Después recordaré todo."

### Correcto

Documenta mientras respondes.

Incluye:

- Hora.
- Acción realizada.
- Evidencia obtenida.
- Resultado.

### ¿Por qué?

Los incidentes son caóticos.

La memoria falla.

La documentación constituye evidencia.

---

## Error 5: Pensar que Recuperación significa "todo terminó"

### Incorrecto

> "El usuario ya puede trabajar. Caso cerrado."

### Correcto

La recuperación implica:

- Restaurar operaciones.
- Continuar monitoreando.
- Buscar reinfecciones.
- Detectar persistencia.

Es habitual mantener monitoreo reforzado durante varios días o incluso semanas.

---

## Error 6: Omitir la revisión posterior al incidente

### Incorrecto

> "Ya resolvimos el incidente. Pasemos al siguiente."

### Correcto

La revisión posterior es donde ocurre el verdadero aprendizaje.

Aquí se identifican:

- Errores.
- Procesos lentos.
- Falta de herramientas.
- Oportunidades de automatización.

Si omites esta fase, probablemente repetirás los mismos errores.

---

## Error 7: No saber cuándo escalar

### Incorrecto

> "Yo puedo resolver todo."

### Correcto

Algunos incidentes requieren la participación de otras áreas.

### Debes escalar cuando exista:

✅ Posible fuga de información.

→ Área Legal.

---

✅ Ransomware.

→ Dirección, CISO y, dependiendo del caso, autoridades.

---

✅ Actividad de un APT.

→ Equipo de Threat Intelligence.

---

✅ Infraestructura crítica comprometida.

→ CISO.

---

✅ Dudas técnicas importantes.

→ Analista Senior.

---

# 🎯 Preguntas de entrevista que podrían hacerte

## Nivel Básico (SOC L1)

### Pregunta

**¿Cuáles son las cuatro fases de la Respuesta a Incidentes?**

### Respuesta

> Preparación, Detección y Análisis, Contención/Erradicación/Recuperación y Actividades Posteriores al Incidente.

---

### Pregunta

**¿Cuál es la diferencia entre Contención y Erradicación?**

### Respuesta

> La Contención detiene el ataque y preserva la evidencia.

> La Erradicación elimina completamente al atacante y corrige la vulnerabilidad.

Siempre se contiene antes de erradicar.

---

### Pregunta

**¿Por qué la preparación es importante si no podemos evitar todos los incidentes?**

### Respuesta

Porque determina la velocidad y calidad de la respuesta.

Una organización preparada responde mediante procedimientos definidos.

Una organización no preparada improvisa.

---

# Nivel Intermedio

### Pregunta

**Explícame cómo responderías a una infección de malware en un servidor crítico.**

### Respuesta

1. Confirmar el incidente.

2. Evaluar el alcance.

3. Aislar el servidor.

4. Obtener evidencia.

5. Bloquear la comunicación del atacante.

6. Eliminar el malware.

7. Corregir la vulnerabilidad.

8. Restaurar desde un respaldo limpio.

9. Validar el funcionamiento.

10. Realizar la revisión posterior al incidente.

---

### Pregunta

**¿Qué es más importante: preservar evidencia o contener rápidamente?**

### Respuesta

Depende del impacto al negocio.

Si el atacante está comprometiendo múltiples sistemas críticos, la prioridad será detener la propagación.

Si el riesgo inmediato es bajo, puede priorizarse una preservación de evidencia más completa.

La decisión siempre debe equilibrar continuidad operativa e investigación forense.

---

### Pregunta

**¿Qué harías si descubrieras una fuga de datos durante la revisión posterior al incidente?**

### Respuesta

1. Escalar inmediatamente.

2. Notificar al área legal.

3. Determinar el alcance de la fuga.

4. Cumplir las obligaciones regulatorias.

5. Preparar las notificaciones correspondientes si fueran necesarias.

---

# Nivel Avanzado (Analista Senior)

### Pregunta

**Estás respondiendo a un incidente que podría terminar en un proceso legal. ¿Cómo cambia eso tu estrategia de contención?**

### Respuesta

Cuando existe la posibilidad de una investigación legal, la preservación de evidencia se vuelve una prioridad crítica.

En este escenario:

- Toda acción debe quedar documentada.
- Debe mantenerse una **Cadena de Custodia (Chain of Custody)** completa.
- Debe registrarse quién manipuló la evidencia, cuándo y por qué.
- El área legal debe involucrarse desde las primeras etapas.
- En algunos casos también se notificará a las autoridades competentes.

La contención puede ser más cuidadosa para evitar alterar evidencia que posteriormente pueda utilizarse en un proceso judicial.

---

### Pregunta

**Describe una situación donde tuviste que elegir entre recuperar rápidamente un sistema o realizar una investigación más profunda.**

### Respuesta

En una entrevista puedes responder utilizando una experiencia real o un laboratorio.

Una estructura sólida sería:

- Explicar el contexto.
- Describir el riesgo para el negocio.
- Explicar qué alternativas existían.
- Justificar la decisión tomada.
- Explicar qué aprendiste.

Lo importante es demostrar que comprendes el equilibrio entre:

- Continuidad del negocio.
- Investigación forense.
- Gestión del riesgo.

---

### Pregunta

**¿Cómo sabes que un incidente realmente fue erradicado?**

### Respuesta

Nunca existe una garantía absoluta, pero puedes alcanzar un alto nivel de confianza cuando:

- Los análisis antivirus ya no detectan amenazas.
- No existen mecanismos de persistencia.
- La vulnerabilidad explotada fue corregida.
- No aparecen nuevos Indicadores de Compromiso (IoCs).
- El monitoreo durante las siguientes 48–72 horas no detecta actividad sospechosa.

Aun así, es recomendable mantener un monitoreo reforzado durante varias semanas.

---

# 🔗 Cómo se relaciona este Framework con otros conocimientos

Comprender NIST SP 800-61 facilita aprender prácticamente todas las demás áreas del Blue Team.

---

## MITRE ATT&CK

Se utiliza principalmente durante la fase de **Detección y Análisis**.

Permite:

- Clasificar el ataque.
- Comprender el comportamiento del adversario.
- Anticipar los siguientes movimientos del atacante.

---

## Windows Event IDs

Los Event IDs representan parte de la evidencia que recopilarás durante:

- Detección.
- Análisis.
- Contención.

Ejemplos:

- 4624
- 4625
- 4672
- 4768
- 4769
- 4771

---

## Redes

Comprender conceptos de redes permite:

- Detectar movimiento lateral.
- Identificar tráfico sospechoso.
- Aplicar contención correctamente.
- Diseñar segmentación de red.

---

## Active Directory

La mayoría de los incidentes empresariales involucran Active Directory.

Los ataques contra AD suelen aparecer en todas las fases:

- Detección.
- Contención.
- Erradicación.
- Recuperación.

---

## DFIR (Digital Forensics & Incident Response)

DFIR complementa especialmente:

- Contención.
- Erradicación.
- Actividades posteriores al incidente.

Aquí se realiza el análisis forense más profundo.

---

## Threat Hunting

El Threat Hunting aparece principalmente durante la fase posterior al incidente.

Su objetivo es descubrir amenazas que aún permanecen ocultas dentro del entorno.

---

## SIEM

El SIEM participa durante todo el ciclo.

- Detecta incidentes.
- Genera alertas.
- Ejecuta playbooks automatizados.
- Proporciona evidencia para la investigación.
- Alimenta las revisiones posteriores.

---

# 💾 Resumen rápido (TL;DR)

| Aspecto | Resumen |
|----------|---------|
| **¿Qué es?** | Framework de cuatro fases para responder a incidentes de seguridad definido por NIST SP 800-61. |
| **¿Por qué importa?** | Es el proceso utilizado por prácticamente todos los SOC empresariales. |
| **Las cuatro fases** | Preparación → Detección y Análisis → Contención / Erradicación / Recuperación → Actividades Posteriores al Incidente |
| **Lo más importante** | Siempre contener antes de erradicar. Preservar evidencia durante la contención. Erradicar únicamente cuando el incidente esté controlado. |
| **Si haces el proceso incorrectamente** | Puedes perder evidencia, permitir que el atacante continúe propagándose o prolongar el incidente durante días o semanas. |
| **Cómo aplicarlo en un SOC** | Cuando recibas una alerta, identifica inmediatamente en qué fase del framework te encuentras y sigue el playbook correspondiente. |

---

# 📌 Cómo ocurre realmente en Producción

## Lunes — 9:00 AM

El SIEM genera una alerta.

**Fase actual: Detección y Análisis**

La alerta indica:

> Posible ataque de Credential Stuffing.

El analista investiga.

Descubre:

- Más de 2,000 intentos fallidos de autenticación.
- Un inicio de sesión exitoso.

El incidente se confirma.

Se escala al Analista Senior.

---

## 9:30 AM

**Fase actual: Contención**

Se realizan las siguientes acciones:

- Restablecer la contraseña comprometida.
- Bloquear la IP atacante.
- Monitorear actividad adicional.
- Preservar evidencia.

El sistema permanece operativo.

---

## 2:00 PM

**Fase actual: Erradicación**

- Buscar persistencia.
- No se encuentra evidencia.
- Aplicar el parche correspondiente.
- Actualizar reglas del firewall.

---

## 5:00 PM

**Fase actual: Recuperación**

- El usuario valida el acceso.
- Todos los servicios funcionan correctamente.
- Se mantiene monitoreo reforzado.

---

## Martes — 10:00 AM

**Fase actual: Actividades Posteriores al Incidente**

El equipo realiza una reunión.

### Causa raíz

Política de contraseñas demasiado débil.

### Mejoras

- Contraseñas complejas.
- MFA obligatorio.
- Actualización del playbook.

Tiempo total:

Aproximadamente **25 horas**, incluyendo monitoreo y validaciones.

---

# 📌 Tu rol en cada fase (Analista SOC L1)

| Fase | Tu responsabilidad | Lo que normalmente NO harás |
|------|---------------------|------------------------------|
| **Preparación** | Aprender herramientas, probar playbooks y conocer los procedimientos. | Crear políticas corporativas o tomar decisiones estratégicas. |
| **Detección y Análisis** | Investigar alertas, confirmar incidentes y escalar cuando corresponda. | Tomar decisiones críticas de negocio sin aprobación. |
| **Contención / Erradicación / Recuperación** | Ejecutar procedimientos establecidos y documentar cada acción. | Definir estrategias de recuperación sin supervisión. |
| **Actividades Posteriores** | Participar en la revisión y aportar mejoras. | Aprobar cambios organizacionales por cuenta propia. |

---

# 📚 Lecturas y recursos recomendados

- **NIST SP 800-61 Rev. 2** — Guía oficial de Respuesta a Incidentes.
- **Windows Security Event IDs** — 4624, 4625, 4672, 4768, 4769 y 4771.
- **MITRE ATT&CK** — Técnicas relacionadas con Impact, Lateral Movement, Credential Access, Persistence, entre otras.
- **Casos reales de incidentes** — Analiza informes publicados por organizaciones de seguridad para comprender cómo se aplican estas fases en entornos reales.
- **Práctica recomendada** — Construye un laboratorio personal y simula incidentes siguiendo este framework de principio a fin.

---

*Última actualización: 2024*

**Nivel de dificultad:** L1

**Relevancia para entrevistas:** ⭐⭐⭐⭐⭐

**Aplicabilidad laboral:** Conocimiento obligatorio para cualquier puesto dentro de un SOC.
