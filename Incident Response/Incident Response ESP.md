# Marco de Respuesta a Incidentes (NIST SP 800-61)

## 📖 ¿Qué Es Esto? (En 30 Segundos)

El Marco de Respuesta a Incidentes es la metodología estructurada que siguen las organizaciones cuando ocurre un incidente de seguridad. NIST SP 800-61 define **cuatro fases** que guían a los investigadores desde el momento en que se detecta un incidente hasta que se aprenden las lecciones y se restauran los sistemas. Piénsalo como el "manual de juego" que te dice qué hacer, en qué orden, y cuándo escalar.

---

## 🎯 ¿Por Qué Un Analista De SOC Necesita Esto?

### En Entrevistas De Trabajo, Escucharás:
- "Camina a través del proceso de respuesta a incidentes desde la detección hasta la recuperación"
- "¿Cuál es la diferencia entre contención y erradicación?"
- "¿Cuándo escalarías? ¿A quién?"
- "¿Cómo preservas evidencia mientras detienes un ataque?"
- "¿Cuál es tu metodología para investigar incidentes?"

### En Tu Primer Mes En Un SOC:
- Recibirás una alerta y pensarás: "¿Ahora qué?"
- Tu SIEM se dispara. Sigues el marco.
- Manejarás decisiones de contención que afectan el negocio
- Preservarás evidencia mientras respondes a ataques
- Reportarás hallazgos a la gerencia usando esta estructura
- Participarás en revisiones posteriores al incidente
- **Hacerlo mal = perder evidencia, propagar infección, o perder al atacante**

---

## 🔍 Las Cuatro Fases (Expandidas)

### **FASE 1: PREPARACIÓN**

#### Qué Es:
Antes de que ocurra un incidente, te preparas. Este es el trabajo de "tiempo de paz" que determina si tendrás éxito cuando llega la crisis.

#### Qué Haces:
- **Construir el equipo:** Entrenar analistas, definir roles, crear rutas de escalada
- **Desplegar herramientas:** SIEM, detección de puntos finales, infraestructura de logging, firewalls
- **Crear manuales:** Procedimientos paso a paso para ataques comunes
- **Establecer políticas:** Cómo preservar evidencia, quién tiene autoridad para tomar decisiones
- **Probar todo:** Realizar simulacros de respuesta a incidentes, probar copias de seguridad, probar canales de comunicación
- **Documentar líneas base:** Saber qué se ve "normal" en tu red

#### Por Qué Importa:
Si no estás preparado, cuando ocurre un incidente estarás improvisando. Los atacantes no esperan mientras figura tu proceso.

#### Ejemplo En SOC Real:
- Lunes: Instalas un nuevo SIEM y configuras reglas de alerta
- Martes: Creas un manual: "Si vemos 1000+ intentos de inicio de sesión fallidos desde una fuente, haz X"
- Jueves: Se dispara la alerta. Sigues tu manual. Funciona.

#### Bandera Roja:
Si tu organización no se ha preparado, los incidentes tardan 3 veces más en resolverse.

---

### **FASE 2: DETECCIÓN Y ANÁLISIS**

#### Qué Es:
Se detecta un incidente (alerta, reporte de usuario, herramienta de seguridad) y determinas qué sucedió realmente.

#### Qué Haces:

**DETECCIÓN (Alguien nota que algo está mal):**
- La herramienta de seguridad genera una alerta
- El administrador del sistema reporta comportamiento sospechoso
- El usuario reporta que no puede acceder a su cuenta
- El firewall bloquea tráfico sospechoso
- El antivirus pone en cuarentena malware

**ANÁLISIS (Investigas para confirmar):**

1. **Clasificar la alerta**
   - ¿Es un verdadero positivo o falso positivo?
   - ¿Cuán crítico es? (Escala: Bajo/Medio/Alto/Crítico)
   - ¿Coincide con un patrón de ataque conocido?

2. **Recopilar evidencia inicial**
   - Recopilar logs del sistema fuente
   - Obtener capturas de tráfico de red
   - Documentar timeline (¿cuándo comenzó esto?)
   - Identificar activos afectados (¿qué fue atacado?)
   - Identificar usuarios afectados (¿quién estuvo involucrado?)

3. **Determinar alcance**
   - ¿Está aislado en una máquina o es en toda la red?
   - ¿Cuántos usuarios están afectados?
   - ¿Cuántos sistemas están comprometidos?
   - ¿Cuánto tiempo ha estado sucediendo?

4. **Clasificar el incidente**
   - **¿Malware?** Usa firmas de antivirus y análisis de comportamiento
   - **¿Acceso no autorizado?** Verifica logs de acceso y escaladas de privilegios
   - **¿Exfiltración de datos?** Monitorea volúmenes de tráfico de red a IPs externas
   - **¿Denegación de Servicio?** Analiza patrones de tráfico
   - **¿Amenaza interna?** Revisa comportamiento del usuario y acceso a datos

5. **Mapear a MITRE ATT&CK**
   - ¿Qué técnicas está usando el atacante?
   - Esto te ayuda a predecir qué harán a continuación

#### Decisiones Críticas En Esta Fase:
- **¿Debemos escalar?** (Sí, si se confirma malicioso)
- **¿Debemos preservar evidencia?** (Sí, siempre)
- **¿Debemos aislar el sistema?** (No todavía—espera a la Fase 3)

#### Errores Comunes:
- ❌ Actuar demasiado rápido sin confirmar que es real
- ❌ No documentar lo que encuentras
- ❌ Eliminar logs antes de revisarlos
- ❌ Asumir que una alerta = cuadro completo

#### Ejemplo En SOC Real:

**10:35 AM** — Alerta SIEM: "20 intentos de inicio de sesión fallidos en CORP-SQL-SERVER desde 192.168.1.105"

**Tus acciones:**
1. Verificar: ¿Es 192.168.1.105 un servidor conocido? (No = sospechoso)
2. Recopilar: ¿Qué usuario estaba siendo atacado? (SA_Admin = muy sospechoso)
3. Timeline: ¿Cuándo comenzó? (9:47 AM = 48 minutos de ataques)
4. Alcance: ¿Otros servidores bajo ataque? (Verificar logs = Sí, 3 servidores más)
5. Clasificar: Esto parece un ataque de fuerza bruta o credential spraying
6. Mapear a MITRE: T1110 (Fuerza Bruta) o T1110.003 (Password Spraying)
7. Decisión: **ESCALAR AL ANALISTA SENIOR** — Esto se confirma como malicioso

---

### **FASE 3: CONTENCIÓN, ERRADICACIÓN Y RECUPERACIÓN**

Esta fase tiene **tres subfases** que suceden en orden. Entender las diferencias es crítico.

#### SUBFASE 3A: CONTENCIÓN

#### Qué Es:
**DETÉN LA HEMORRAGIA.** Impide que el atacante cause más daño mientras preservas la evidencia.

#### La Estrategia De Contención:
```
Contención a Corto Plazo (0-30 minutos):
↓
Prevenir propagación, mantener evidencia, mantener sistema ejecutándose si es posible

Contención a Mediano Plazo (30 minutos - horas):
↓
Aislar sistemas comprometidos, bloquear comunicación del atacante

Contención a Largo Plazo (horas - días):
↓
Fortalecer defensas, monitorear persistencia
```

#### Cómo Funciona La Contención:

**Paso 1: Aislar el sistema afectado**
- Desconectar de la red (pero mantener encendido para recopilar memoria viva)
- O cambiar políticas de red para restringir movimiento
- Objetivo: El atacante no puede propagarse a otros sistemas

**Paso 2: Preservar la evidencia**
- Recopilar dump de memoria (RAM contiene conexiones activas, procesos)
- Copiar disco duro antes de hacer cambios
- Documentar lo que ves (capturas de pantalla, logs, timestamps)
- Cadena de custodia: Registra quién tocó qué y cuándo

**Paso 3: Bloquear comunicación del atacante**
- Si el atacante tiene acceso shell, mata la conexión
- Bloquea IPs del servidor C2 en el firewall
- Reinicia credenciales para que el atacante no pueda volver a iniciar sesión
- PERO: No hagas esto todavía si aún estás investigando

**Paso 4: Mantener la integridad del sistema**
- No reinicies el sistema todavía (podrías perder evidencia)
- No elimines archivos o logs
- No apliques parches de vulnerabilidades todavía (podría ocultar cómo entraron)
- **Mantén el sistema tal como está para análisis forense**

#### Decisión Crítica En Contención:
**"¿Cuánto daño puede hacer el atacante AHORA?"**
- ¿Alto riesgo de daño? → Contención agresiva (desconectar sistema)
- ¿Bajo riesgo de daño? → Contención suave (restringir red, mantener ejecutándose)

#### Ejemplo En SOC Real:

**11:15 AM** — Confirmado: CORP-LAPTOP-203 está infectado con troyano

**Tus acciones de contención:**
1. ✅ Recopilar dump de memoria (mientras el sistema sigue ejecutándose)
2. ✅ Desconectar de la red (pero mantener encendido)
3. ✅ Bloquear IP del servidor C2 en el firewall (detiene comandos nuevos)
4. ✅ Forzar reinicio de contraseña para el usuario afectado
5. ✅ Documentar todo en el ticket de incidente
6. ⏸️ No reinicies el sistema todavía
7. ⏸️ No ejecutes limpieza de antivirus todavía

**Resultado:** El atacante está contenido. La evidencia está preservada. La investigación continúa.

---

#### SUBFASE 3B: ERRADICACIÓN

#### Qué Es:
**MATA LA INFECCIÓN.** Elimina completamente la presencia del atacante.

#### Cómo Funciona La Erradicación:

**Paso 1: Elimina el malware**
- Ejecuta escaneos de antivirus/malware
- Elimina archivos infectados
- Elimina tareas programadas o entradas de registro creadas por malware
- Desinstala puertas traseras o herramientas de acceso remoto

**Paso 2: Cierra la vulnerabilidad**
- Aplica parches al software/SO donde el atacante entró
- Cambia reglas de firewall para prevenir re-entrada
- Actualiza controles de acceso
- Ejemplo: Si explotaron SQL Server sin parches, aplica parches AHORA

**Paso 3: Elimina mecanismos de persistencia**
- El malware a menudo crea formas alternativas de retorno
- Elimina toda persistencia (puertas traseras, tareas programadas, etc.)
- Verifica la presencia de rootkits o botkits
- Verifica que todo malware esté gone (no solo la carga principal)

**Paso 4: Verifica la erradicación**
- Escanea el sistema nuevamente para confirmar que no quede nada
- Verifica logs en busca de actividad restante del atacante
- Verifica que la vulnerabilidad esté cerrada

#### Diferencia Crítica: Contención Vs. Erradicación

| Contención | Erradicación |
|-------------|------------|
| **Detiene** el ataque | **Elimina** el ataque |
| Preserva evidencia | Puede destruir evidencia (aceptable) |
| Sistema sigue comprometido | Sistema se limpia |
| Reversible | Irreversible |
| Puede suceder rápidamente | Toma más tiempo |
| Ejemplo: Desconectar laptop | Ejemplo: Ejecutar antivirus, aplicar parches, reiniciar |

#### Errores Comunes:
- ❌ Borrar logs antes de entender cómo entraron
- ❌ Aplicar parches antes de completar análisis forense
- ❌ No verificar persistencia (malware vuelve después)
- ❌ Asumir que un reinicio = completamente limpio

#### Ejemplo En SOC Real:

**3:30 PM** — Contención completa. Ahora erradica.

**Tus acciones de erradicación:**
1. ✅ Ejecuta escaneo completo de antivirus → Detecta y elimina troyano
2. ✅ Busca persistencia → Encuentra tarea programada (malware.exe se ejecuta diariamente)
3. ✅ Elimina la tarea programada
4. ✅ Verifica registro de Windows → Elimina entradas de registro maliciosas
5. ✅ Aplica parche a la vulnerabilidad de SQL Server que fue explotada
6. ✅ Escanea nuevamente → No se encuentran infecciones
7. ✅ Documenta: "Erradicación completada en [timestamp]"

**Resultado:** El sistema está limpio. Las herramientas del atacante están gone. La vulnerabilidad está parcheada.

---

#### SUBFASE 3C: RECUPERACIÓN

#### Qué Es:
**DEVUELVE SISTEMAS A LA VIDA.** Restaura sistemas a operaciones normales de forma segura.

#### Cómo Funciona La Recuperación:

**Paso 1: Restaura copias de seguridad limpias**
- Usa copia de seguridad de ANTES de la infección
- Restaura archivos del usuario, configuraciones, aplicaciones
- Verifica la integridad de la copia de seguridad (sin malware)

**Paso 2: Reconstruye sistemas (si es necesario)**
- Si la copia de seguridad es demasiado antigua o no está disponible, reconstruye desde cero
- Reinstala SO, parches, aplicaciones
- Restaura datos solo de copias de seguridad limpias
- Tómate tiempo—la velocidad no es más importante que la seguridad

**Paso 3: Restaura acceso y conectividad**
- Reconecta sistema a la red
- Restaura reglas de red y políticas de firewall
- Restaura acceso del usuario y permisos
- PERO: Mantén monitoreo para actividad sospechosa

**Paso 4: Valida funcionalidad completa**
- ¿Funciona el sistema normalmente?
- ¿Pueden los usuarios acceder a sus archivos?
- ¿Están todas las aplicaciones funcionando?
- ¿No hay mensajes de error o problemas de rendimiento?

**Paso 5: Monitorea de cerca post-recuperación**
- Observa señales de re-infección
- Monitorea tráfico de red del sistema recuperado
- Observa intentos de inicio de sesión inusuales
- Este monitoreo dura días/semanas

#### Decisión En Recuperación:
**"¿Restauramos desde copia de seguridad o reconstruimos desde cero?"**
- Riesgo: La copia de seguridad podría contener malware (demasiado riesgoso, reconstruir)
- Seguro: Reconstruir sistema, luego restaurar solo datos del usuario (más lento pero más seguro)

#### Ejemplo En SOC Real:

**5:00 PM** — Erradicación confirmada. Tiempo de recuperación.

**Tus acciones de recuperación:**
1. ✅ Verificar copia de seguridad del 15 de marzo (antes de infección el 17 de marzo)
2. ✅ Verificar que la copia de seguridad no tiene firmas de malware
3. ✅ Restaurar sistema desde copia de seguridad limpia
4. ✅ Reinstalar parches más recientes (actualizaciones de seguridad de marzo)
5. ✅ Restaurar archivos del usuario desde copia de seguridad
6. ✅ Reconectar a la red
7. ✅ El usuario prueba el sistema → Todo funciona
8. ✅ Habilitar monitoreo mejorado en este sistema durante 30 días

**Resultado:** Sistema recuperado. El usuario vuelve al trabajo. La seguridad se mantiene.

---

#### Por Qué Las Tres Subfases Importan:

```
Contención    →    Erradicación    →    Recuperación
(Detenlo)          (Elimínalo)          (Restítuyelo)
(Preserva evidencia) (Destruye evidencia)  (Operaciones normales)
(Urgente)          (Metódico)           (Cuidadoso)
```

Si saltas o arruinas una subfase:
- ❌ ¿Saltas contención? El atacante se propaga a más sistemas
- ❌ ¿Saltas erradicación? El malware vuelve después de la recuperación
- ❌ ¿Saltas recuperación? Los sistemas no funcionan, los usuarios no pueden ser productivos

---

### **FASE 4: ACTIVIDAD POSTERIOR AL INCIDENTE (REVISIÓN Y LECCIONES APRENDIDAS)**

#### Qué Es:
Después de que se resuelve el incidente, te pausas para aprender. ¿Qué sucedió? ¿Por qué? ¿Cómo lo prevenimos la próxima vez?

#### Qué Haces:

**Paso 1: Recopilar reportes de estado del equipo de infraestructura**
- Documenta todos los cambios realizados durante la respuesta
- Verifica que todos los sistemas funcionen completamente
- Confirma que ningún sistema sigue comprometido
- Asegura que las copias de seguridad se están ejecutando normalmente

**Paso 2: Revisa aplicaciones/herramientas nuevas desplegadas**
- ¿Desplegamos alguna herramienta de seguridad nueva durante la respuesta?
- ¿Están funcionando correctamente?
- ¿Están generando alertas útiles?
- Documenta su rendimiento

**Paso 3: Realiza threat hunting (si es necesario)**
- ¿Hay señales de que este atacante estuvo en la red más tiempo de lo que creemos?
- ¿Accedieron a otros sistemas que no encontramos?
- ¿Hay otros compromisos que no hemos encontrado todavía?
- Esta es una búsqueda proactiva, no investigación reactiva

**Paso 4: Realiza reunión posterior al incidente (Lecciones Aprendidas)**
- **¿Qué sucedió?** Timeline factual
- **¿Por qué sucedió?** Causa raíz
- **¿Cómo respondimos?** ¿Qué funcionó? ¿Qué no?
- **¿Qué mejoraremos?** Elementos de acción específicos
- **¿Quién necesita saber?** Gerencia, cumplimiento, otros equipos

**Paso 5: Documenta recomendaciones**
- Cambios de política necesarios
- Herramientas nuevas o actualizaciones
- Capacitación necesaria para personal
- Reglas de firewall a agregar
- Reglas de monitoreo a mejorar
- Actualizaciones de manual

**Paso 6: Actualiza tus manuales**
- Si este es un tipo de ataque conocido, mejora el manual
- ¿Qué tomó demasiado tiempo? Simplíficalo
- ¿Qué fue confuso? Documéntalo mejor
- ¿Qué funcionó bien? Hazlo estándar

#### Decisiones Críticas Posteriores Al Incidente:
- **¿Necesitamos reporte de cumplimiento?** (PCI DSS, HIPAA, GDPR, etc.)
- **¿Necesitamos revisión legal?** (Posible brecha de datos)
- **¿Necesitamos notificar a usuarios?** (Se expusieron datos)
- **¿Qué le decimos a la gerencia?** (Impacto comercial)

#### Ejemplo En SOC Real:

**18 de marzo, 10:00 AM** — Incidente resuelto. Tiempo para lecciones aprendidas.

**Tus acciones posteriores al incidente:**
1. ✅ Recopilar estado de infraestructura → Todos los sistemas operativos
2. ✅ Revisar nuevas reglas de firewall desplegadas → Funcionando correctamente, bloqueando intentos C2
3. ✅ Threat hunting → Encontramos 2 intentos adicionales de movimiento lateral (los contuvimos)
4. ✅ Reunión posterior al incidente con equipo:
   - Qué sucedió: Troyano mediante correo de phishing
   - Por qué: El usuario hizo clic en el enlace, descargó malware
   - Cómo respondimos: Contuvimos, erradicamos, recuperamos (2 horas totales)
   - Qué funcionó: La alerta SIEM lo detectó, la contención fue rápida
   - Qué falló: El filtrado de correo no lo detectó
5. ✅ Recomendaciones:
   - Fortalecer reglas de filtrado de correo
   - Capacitación obligatoria de conciencia de phishing para todo el personal
   - Desplegar protección de punto final adicional
6. ✅ Actualizar manual: "Respuesta a Troyano" ahora incluye procedimientos de bypass de filtrado de correo

**Resultado:** El equipo aprende, los procesos mejoran, el siguiente incidente se maneja más rápido.

---

## ⚙️ Lo Que DEBES Memorizar

**Las Cuatro Fases (En Orden):**
1. **PREPARACIÓN** — Antes del incidente (construir herramientas, entrenar equipo, crear manuales)
2. **DETECCIÓN Y ANÁLISIS** — Ocurre incidente (detectar, confirmar, investigar)
3. **CONTENCIÓN, ERRADICACIÓN, RECUPERACIÓN** — Detener, eliminar, restaurar (en ese orden)
4. **ACTIVIDAD POSTERIOR AL INCIDENTE** — Después de la resolución (aprender, mejorar, documentar)

**Dispositivo De Memoria:**
```
PREPARACIÓN → DETECTAR → CONTENER → ERRADICAR → RECUPERAR → REVISAR

P → D → C → E → R → R

"Planifica, Detecta, Contiene, Erradica, Recupera, Revisa"
```

**Subfases (Importante):**
- **Contención** = Detén la hemorragia (preserva evidencia)
- **Erradicación** = Mata la infección (destruye evidencia está OK)
- **Recuperación** = Restaura a normal (verifica funcionalidad)

---

## 📚 Lo Que DEBES Entender

- [ ] **Por qué el orden importa** — No puedes erradicar antes de contener (el atacante se propaga)
- [ ] **Contención vs. Erradicación** — Diferentes objetivos, diferentes acciones, diferente cronograma
- [ ] **Preservación de evidencia** — Dónde y cuándo importa más (fase de Contención)
- [ ] **Puntos de escalada** — Cuándo llamar analista senior, gerente, CEO, abogados
- [ ] **Impacto comercial** — Cada fase afecta cuánto tiempo los usuarios están sin sistemas
- [ ] **Planificación de recuperación** — Procedimientos de copia de seguridad, reconstrucción, prueba
- [ ] **Mejora continua** — La revisión posterior al incidente es cómo mejoras

---

## 🚨 Aplicación Del Mundo Real: Cómo Se Conectan Las Fases

### Escenario: Infección De Ransomware

**PREPARACIÓN (Sucedió el mes pasado):**
- Desplegamos herramienta de detección de ransomware
- Creamos manual de respuesta a ransomware
- Entrenamos al equipo en procedimientos de recuperación
- Probamos proceso de restauración de copia de seguridad

**DETECCIÓN Y ANÁLISIS (Hoy 8:00 AM):**
- El antivirus detecta ransomware en CORP-FINANCE-SVR
- El analista confirma: 150 archivos cifrados, nota de rescate presente
- Alcance: Solo 1 servidor afectado (por ahora)
- Clasificación: Crítico — Los datos de finanzas están cifrados

**CONTENCIÓN (Hoy 8:30 AM):**
- Desconectar servidor de la red (detener propagación de ransomware)
- Recopilar memoria y disco para análisis forense
- Bloquear comunicación C2 del atacante en firewall
- El servidor permanece aislado pero encendido
- **Decisión: NO pagar rescate**

**ERRADICACIÓN (Hoy 10:00 AM):**
- Ejecutar escaneos de malware → Encontrar y eliminar ransomware
- Buscar mecanismos de persistencia → No encontrar ninguno
- Aplicar parche a la vulnerabilidad que fue explotada
- Verificar que la infección esté gone

**RECUPERACIÓN (Hoy 1:00 PM):**
- Restaurar servidor desde copia de seguridad (15 de marzo, antes de infección)
- Verificar que la copia de seguridad no tiene ransomware
- Restaurar archivos del usuario
- Ejecutar antivirus en todo el servidor
- Reconectar a la red
- El equipo de finanzas prueba el sistema

**ACTIVIDAD POSTERIOR AL INCIDENTE (Mañana 9:00 AM):**
- Recopilar estado de infraestructura → Todos los sistemas normales
- Threat hunting → No se encontró otro ransomware
- Reunión posterior al incidente:
  - Qué funcionó: Detección rápida, contención rápida, recuperación exitosa
  - Qué falló: La vulnerabilidad no fue parcheada (IT la tenía en lista de tareas pendientes)
  - Cómo: El ransomware explotó Exchange Server sin parches
- Recomendaciones:
  - El proceso de gestión de parches necesita automatización
  - Las pruebas de copia de seguridad deben suceder mensualmente, no trimestralmente
  - Capacitación de ransomware para usuarios finales
- Actualizar manual con lecciones aprendidas

**Tiempo total: 24 horas (no días o semanas)**

---

## ❌ Errores Comunes Que Cometen Los Estudiantes

### Error 1: Pensar Que Las Fases Son Solo Secuenciales
**Incorrecto:** "Termino Preparación, luego me muevo a Detección para siempre"
**Correcto:** Estás SIEMPRE en Preparación (incluso mientras estás en otras fases). Cada incidente hace mejor la Preparación.

### Error 2: Confundir Contención Con Erradicación
**Incorrecto:** "Contención y erradicación son lo mismo"
**Correcto:** La contención detiene el ataque (sistema sigue comprometido). La erradicación elimina el ataque (sistema se limpia).
**Por qué importa:** Si intentas erradicar antes de contener, el atacante se propaga mientras limpias.

### Error 3: Eliminar Evidencia Demasiado Pronto
**Incorrecto:** "Después de reiniciar el sistema, el ataque se fue"
**Correcto:** Reiniciar destruye evidencia (la memoria se pierde). Recopila evidencia PRIMERO, luego reinicia.
**Consecuencia Real:** Violaciones de cumplimiento, no puedes probar qué sucedió, el atacante queda libre.

### Error 4: No Documentar Durante La Respuesta
**Incorrecto:** "Documentaré todo después de que se resuelva el incidente"
**Correcto:** Documenta MIENTRAS respondes. Los timestamps, las acciones, los hallazgos importan.
**Por qué:** La memoria falla. Los incidentes son caóticos. Olvidarás los detalles.

### Error 5: Pensar Que La Recuperación = Volver Al Trabajo
**Incorrecto:** "Se restauró el sistema, terminamos"
**Correcto:** La recuperación significa operaciones funcionales CON monitoreo. No "terminaste" durante semanas.
**Por qué:** El atacante podría tener persistencia (puerta trasera). Necesitas vigilar re-infección.

### Error 6: Saltarse La Revisión Posterior Al Incidente
**Incorrecto:** "El incidente terminó, tiempo de pasar a la siguiente alerta"
**Correcto:** La revisión posterior al incidente es dónde mejoras. Saltártela y manejarás el siguiente incidente de la misma forma.
**Por qué:** Los incidentes son oportunidades de aprendizaje. Desperdiciar eso = cometer los mismos errores nuevamente.

### Error 7: No Entender Cuándo Escalar
**Incorrecto:** "Manejaré todo yo mismo"
**Correcto:** Algunos incidentes necesitan analistas senior, gerencia, legal, o cumplimiento del sistema.
**Cuándo escalar:**
- ✅ Posible brecha de datos → Equipo legal
- ✅ Ransomware → Gerencia + posiblemente cumplimiento del sistema
- ✅ Actividad APT → Traer equipo de intel de amenazas
- ✅ Infraestructura crítica → Escalar a CISO
- ✅ Cualquier cosa de la que no estés seguro → Pregunta analista senior

---

## 🎯 Preguntas De Entrevista Que Podrías Recibir

### Fácil (Conocimiento L1)
**P:** "¿Cuáles son las cuatro fases de respuesta a incidentes?"
**R:** "Preparación, Detección y Análisis, Contención/Erradicación/Recuperación, y Actividad Posterior al Incidente."

**P:** "¿Cuál es la diferencia entre contención y erradicación?"
**R:** "La contención detiene el ataque de propagarse mientras preserva la evidencia. La erradicación elimina completamente la presencia del atacante. Contienen primero, luego erradican."

**P:** "¿Por qué es importante la preparación si no podemos detener todos los incidentes?"
**R:** "La preparación determina qué tan rápido y efectivamente respondemos. Sin ella, improvisamos durante crisis, lo que es más lento y propenso a errores."

### Medio (Muestra Entendimiento)
**P:** "Camina a través de cómo responderías a una infección confirmada de malware en un servidor crítico."
**R:** 
1. Detección y Análisis: Confirma la infección, documenta alcance
2. Contención: Aisla el servidor, recopila análisis forense, bloquea C2
3. Erradicación: Elimina malware, aplica parche a vulnerabilidad, verifica que esté gone
4. Recuperación: Restaura desde copia de seguridad limpia, prueba funcionalidad
5. Actividad Posterior al Incidente: Revisa qué sucedió, actualiza manuales

**P:** "¿Qué es más importante: contener rápidamente o preservar evidencia?"
**R:** "Ambos importan, pero la prioridad depende del impacto comercial. Si el atacante se propaga a sistemas críticos, contiene agresivamente aunque signifique perder alguna evidencia. Si está contenido a un sistema de bajo riesgo, preserva evidencia para investigación exhaustiva."

**P:** "¿Qué harías si descubrieras evidencia de brecha de datos durante revisión posterior al incidente?"
**R:** 
1. Escalar a gerencia inmediatamente
2. Involucrar equipo legal para obligaciones de cumplimiento
3. Investigar cuántos datos fueron accedidos
4. Preparar para notificación al usuario si es requerido por ley
5. Trabajar con legal en timeline de divulgación

### Difícil (Nivel Analista Senior)
**P:** "Estás respondiendo a un incidente dónde necesitas evidencia para posible enjuiciamiento. ¿Cómo cambia eso tu estrategia de contención?"
**R:** "La cadena de custodia se vuelve crítica. Cada acción debe estar documentada—quién tocó qué, cuándo, por qué. Involucraría a legal y posiblemente cumplimiento del sistema temprano. La contención podría ser más lenta y deliberada para preservar integridad de evidencia. Evitaría acciones que destruyan evidencia incluso si aceleraría la erradicación."

**P:** "Describe un momento cuando tuviste que elegir entre recuperación rápida e investigación exhaustiva. ¿Qué hiciste?"
**R:** "Si esto sucediera: [Ejemplo específico de tu experiencia o lab]. Aquí está mi proceso de toma de decisión: [Explica prioridad, a quién consultaste, por qué elegiste ese enfoque]. Esto me enseñó que la escalada a gerencia importa—ellos deciden trueques comerciales vs. seguridad, no el analista."

**P:** "¿Cómo sabes cuándo un incidente está verdaderamente erradicado?"
**R:** "Nunca al 100%, pero cerca: Múltiples escaneos limpios con firmas actuales, ningún mecanismo de persistencia encontrado, vulnerabilidad parcheada, monitoreo para re-infección muestra nada sospechoso durante 48-72 horas. Incluso entonces, el monitoreo mejorado continúa durante semanas para detectar persistencia de último momento."

---

## 🔗 Cómo Esto Se Conecta Con Todo Lo Demás

- **MITRE ATT&CK:** Las técnicas se mapean a fase de Detección y Análisis. Clasificas qué sucedió.
- **IDs De Evento Windows:** Evidencia que recopila en fases de Detección y Análisis y Contención.
- **Conceptos De Redes:** Entender movimiento lateral ayuda en Contención (dónde bloquear).
- **Active Directory:** La mayoría de incidentes empresariales involucran AD. Los ataques de AD abarcan todas las cuatro fases.
- **DFIR:** El análisis forense sucede durante Contención/Erradicación (recopila evidencia) y Actividad Posterior al Incidente (análisis detallado).
- **Threat Hunting:** Búsqueda proactiva en fase Posterior al Incidente encuentra amenazas antes de que alerten.
- **SIEM:** Detecta incidentes (Fase 2), dispara manuales automáticamente (Fase 3), alimenta datos a revisión (Fase 4).

---

## 💾 TL;DR Para Personas Ocupadas

| Aspecto | Respuesta |
|--------|--------|
| **¿Qué es?** | Marco de cuatro fases para responder a incidentes de seguridad (NIST SP 800-61) |
| **¿Por qué importa?** | Cada trabajo de SOC usa esto. Cada incidente sigue este orden. |
| **Las cuatro fases:** | Preparación → Detección y Análisis → Contención/Erradicación/Recuperación → Actividad Posterior |
| **Recuerda esto:** | Contén ANTES de erradicar. Preserva evidencia durante contención. Erradica después de contener. |
| **Si lo arruinas:** | El atacante se propaga, la evidencia se pierde, el incidente toma semanas en lugar de horas |
| **Cómo usarlo en el trabajo:** | Cuando se dispara alerta, pregunta "¿En qué fase estamos?" y sigue el manual. |

---

## 📌 Realidad De Producción: Cómo Se Ve Realmente

**Lunes 9:00 AM** — Se dispara alerta. Estás en Fase 2 (Detección y Análisis)
- SIEM dice: "Posible ataque de credential stuffing"
- Investigas: "Sí, confirmado. 2,000 intentos fallidos, 1 exitoso"
- Decisión: Escala a analista senior

**Lunes 9:30 AM** — El analista senior se une. Ahora Fase 3 (Contención)
- "Reinicia la contraseña comprometida"
- "Monitorea acceso adicional"
- "Bloquea la IP del atacante en el firewall"
- La evidencia se recopila pero el sistema sigue ejecutándose

**Lunes 2:00 PM** — Fase 3 (Erradicación)
- "Escanea persistencia" → Nada encontrado
- "Aplica parche a la vulnerabilidad que explotaron"
- "Actualiza regla de firewall"

**Lunes 5:00 PM** — Fase 3 (Recuperación)
- El usuario cambió contraseña (ya lo hizo a las 9:30)
- El usuario confirma que puede acceder a sistemas
- "Listo"

**Martes 10:00 AM** — Fase 4 (Actividad Posterior al Incidente)
- Reunión del equipo
- "¿Cuál fue la causa raíz?" → Política de contraseña débil
- "¿Qué mejoramos?" → Requiere contraseñas complejas, habilita MFA
- Actualizar documentación de política de contraseña

**Tiempo total: ~25 horas (la mayoría es espera + monitoreo)**

---

## 📌 Una Cosa Más: Tu Rol En Cada Fase

**Como Analista L1 De SOC, harás:**

| Fase | Tu Rol | Qué NO Haces |
|-------|-----------|------------------|
| **Preparación** | Prueba manuales, aprende herramientas | Crear política o contratar |
| **Detección y Análisis** | Investigar, confirmar, escalar | Tomar decisiones de contención solo |
| **Contención/Erradicación/Recuperación** | Ejecutar procedimientos, documentar | Tomar decisiones grandes (escala a L3/gerencia) |
| **Actividad Posterior al Incidente** | Contribuye a revisión, sugiere mejoras | Hacer cambios de política solo |

---

## 📚 Lectura Adicional Y Recursos

- **NIST SP 800-61 Rev. 2** — Marco oficial (Googléalo, es gratis)
- **IDs De Evento Windows Relacionados:** 4624, 4625, 4672, 4768, 4769, 4771
- **Técnicas De MITRE ATT&CK:** Impacts (Service Stop, Resource Hijacking, etc.)
- **Writeups De Incidentes Reales:** Revisa HackerNews, Bleeping Computer para postmortems detallados
- **Práctica:** Construye un lab en casa y simula incidentes siguiendo este marco

---

*Última Actualización: 2024*
*Dificultad: L1*
*Relevancia De Entrevista: ⭐⭐⭐⭐⭐*
*Aplicabilidad De Trabajo: Conocimiento Requerido Para Todos Los Roles De SOC*
