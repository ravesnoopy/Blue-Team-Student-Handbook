# ☁️ Seguridad en la Nube (Cloud Security): Una Inmersión Profunda para Blue Team

## 📖 ¿Qué es esto? (Definición de 30 segundos)

Seguridad en la nube es la práctica de proteger datos, aplicaciones e infraestructura alojados en plataformas en la nube (AWS, Azure, Google Cloud, etc.) contra acceso no autorizado, robo y compromiso. Combina seguridad de red, gestión de identidad y acceso, protección de datos, y respuesta a incidentes—adaptados a un modelo de responsabilidad compartida donde el proveedor de nube asegura la infraestructura mientras los clientes aseguran sus configuraciones, aplicaciones y controles de acceso. Los Blue Teams deben detectar, investigar y responder a incidentes basados en la nube mientras mantienen seguridad híbrida (on-premise + nube).

---

## 🎯 ¿Por qué un profesional de Blue Team/SOC necesita saber esto?

### Escenarios reales en el trabajo:
- **Infraestructura Híbrida:** Tu organización ejecuta servidores on-premise Y en AWS. Un incidente ocurre en la nube. Tu SOC debe detectarlo, pero los logs de AWS son diferentes a los logs on-premise.
- **Responsabilidad Compartida:** AWS está comprometido. ¿Es culpa de AWS o tuya? Necesitas entender el límite.
- **Brecha de IAM:** Un atacante obtiene credenciales de AWS. ¿Qué pueden acceder? ¿Cómo detectas acceso no autorizado? ¿Cómo lo contendrías?
- **Exfiltración de Datos desde la Nube:** Un insider malicioso o app comprometida en tu entorno de nube está descargando datos sensibles. Tu SIEM necesita reglas de detección específicas de nube.
- **Explotación de Costos:** Los atacantes lanzan mineros de criptomonedas en tu infraestructura de nube. Tu factura de nube se triplica durante la noche. ¿Cómo lo detectas?

### Preguntas de Entrevista que Enfrentarás:
- "Explica el modelo de responsabilidad compartida"
- "¿Cómo es diferente la respuesta a incidentes en la nube vs. on-premise?"
- "¿Qué buscarías para detectar un compromiso de AWS?"
- "¿Cuál es la diferencia entre IAM, RBAC y ABAC en nube?"

---

## 🔍 El Concepto Desglosado

### **Parte 1: Fundación - Nube vs. On-Premise (La Diferencia Core)**

#### **Seguridad On-Premise (Tradicional)**
```
Tu Organización
├─ Servidores Físicos (los posees)
├─ Infraestructura de Red (la gestionas)
├─ Sistema Operativo (lo parches)
├─ Aplicaciones (las aseguras)
├─ Datos (los proteges)
└─ Todo = TU responsabilidad
```

**Enfoque de Seguridad:**
- Firewall en perímetro de red
- Controles de acceso físico al data center
- Parches directos del OS
- Seguridad directa de aplicación
- Visibilidad total en todo

**Respuesta a Incidentes:**
- Acceso a todos los logs (en tus servidores)
- Aislamiento físico posible
- Control forense completo

---

#### **Seguridad en la Nube (AWS/Azure/GCP)**
```
Tu Organización (Cliente de Nube)
├─ Aplicaciones (construyes)
├─ Datos (subes)
├─ Control de Acceso de Usuario (configuras IAM)
└─ Configuración (gestionas)

Proveedor de Nube (AWS/Azure/GCP)
├─ Infraestructura Física (aseguran)
├─ Red (gestionan)
├─ Sistema Operativo (parchean)
├─ Capa de Virtualización (mantienen)
└─ Seguridad Física (la hacen cumplir)
```

**Diferencia Clave:** Compartes responsabilidad. Esto cambia TODO sobre respuesta a incidentes.

---

### **Parte 2: El Modelo de Responsabilidad Compartida (Esto es Crítico)**

#### **De qué AWS/Azure/GCP es Responsable:**
- Seguridad de data center físico
- Infraestructura de red
- Hypervisor (capa de virtualización)
- Sistemas de almacenamiento
- **Protegen la nube**

#### **De qué TÚ (Cliente) Eres Responsable:**
- Parches de instancia EC2/VM
- Seguridad de código de aplicación
- Configuración de base de datos
- Gestión de Identidad y Acceso (IAM)
- Claves de encriptación (si las gestionas)
- Reglas de firewall que creas
- Monitoreo de tus propios recursos
- **Proteges tu stuff EN la nube**

---

#### **Ejemplo Real: Ransomware WannaCry**

**Escenario 1: On-Premise**
```
WannaCry ataca tu servidor
↓
Accedes directamente a logs del servidor
↓
Parches inmediatamente
↓
Ves el ataque en progreso (visibilidad total)
↓
Bloqueas acceso de red
↓
Recuperas desde backup
```

**Escenario 2: Nube (AWS)**
```
WannaCry ataca tu instancia EC2
↓
Necesitas revisar AWS CloudTrail (servicio de logging de AWS)
↓
Infraestructura de AWS protegida, pero TU instancia necesita parches
↓
DEBES parchear TU AMI (Amazon Machine Image)
↓
La visibilidad depende de qué configuraste (CloudTrail, VPC Flow Logs)
↓
Aíslas modificando security groups
↓
Recuperas desde snapshots de EBS que creaste
```

**Insight Clave:** En la nube, TÚ eres responsable de saber que fuiste atacado. AWS te dice QUÉ sucedió en su infraestructura, pero TÚ debes configurar logging para tus aplicaciones.

---

### **Parte 3: Amenazas en la Nube vs. On-Premise**

#### **Amenazas Tradicionales On-Premise:**
- Malware en servidores
- Intrusiones de red
- Robo físico
- Amenazas internas con acceso físico

#### **Amenazas Específicas de la Nube:**

| Tipo de Amenaza | Definición | Ejemplo | Detección |
|---|---|---|---|
| **Compromiso de Credenciales** | Atacante obtiene claves de acceso de AWS o contraseñas | Desarrollador hace commit de claves de AWS a GitHub | CloudTrail muestra acceso desde IP inusual |
| **Misconfiguration** | S3 bucket público, security groups demasiado abiertos | S3 bucket con backups públicamente legible | AWS Config detecta bucket público |
| **Escalamiento de Privilegios** | Usuario de baja autoridad obtiene acceso de admin vía fallo de IAM | Usuario con permiso "ec2:*" escala a modificar IAM | CloudTrail muestra cambios inesperados de IAM |
| **Exfiltración de Datos** | Atacante descarga datos sensibles de la nube | Instancia EC2 comprometida descarga backups de BD | VPC Flow Logs muestra transferencia saliente grande |
| **Secuestro de Recursos** | Atacante usa tus recursos de nube para minería/DDoS | Minador de cripto deployado en tu instancia | CPU anormalmente alta, costos de AWS altos |
| **Ataque de Cadena de Suministro** | Imagen de contenedor comprometida o código de tercero malicioso | Imagen Docker maliciosa usada en ECS | Escaneo de imagen de contenedor detecta vulnerabilidades |
| **Deployment No Autorizado** | Atacante crea nuevos recursos en tu cuenta | Atacante lanza instancia EC2 en tu cuenta | CloudTrail muestra EC2:RunInstances de usuario desconocido |

---

### **Parte 4: El Modelo de Responsabilidad Compartida - Detallado**

#### **Controles de Seguridad por Capa:**

```
Capa de Aplicación (TÚ)
├─ Seguridad de código
├─ Lógica de autenticación/autorización
└─ Seguridad de API

Capa de Datos (COMPARTIDA)
├─ Encriptación de datos (tú proporcionas claves, proveedor encripta)
├─ Backups de base de datos (tú decides, proveedor almacena)
└─ Clasificación de datos (tú decides, proveedor la hace cumplir)

Capa de Sistema Operativo (TÚ para instancias, PROVEEDOR para servicios gestionados)
├─ Parches de instancia (tú para EC2, AWS para RDS)
├─ Hardening (tú para EC2, AWS para Lambda)
└─ Gestión de configuración (tú para EC2, AWS para contenedores)

Capa de Red (TÚ configuras, PROVEEDOR proporciona)
├─ Security groups (reglas de firewall - TÚ creas)
├─ ACLs de Red (reglas a nivel de red - TÚ configuras)
├─ Setup de VPC (aislamiento de red - TÚ diseñas)
└─ Seguridad de red de AWS (protección DDoS - AWS proporciona)

Capa de Infraestructura (AWS)
├─ Servidores físicos
├─ Hypervisor
├─ Controles de acceso físico
└─ Seguridad de cadena de suministro
```

---

### **Parte 5: IAM - El Vector de Ataque Más Común**

#### **¿Qué es IAM (Gestión de Identidad y Acceso)?**
IAM controla **QUIÉN** puede hacer **QUÉ** en tu cuenta de nube.

#### **Componentes:**

**Usuarios:** Individuos (desarrolladores, admins, cuentas de servicio)
```
Ejemplo: john@company.com tiene permisos de desarrollador
```

**Roles:** Grupos de permisos aplicados a usuarios o servicios
```
Ejemplo: Rol "EC2-Admin" = puede iniciar/detener/terminar instancias EC2
```

**Políticas:** Permisos específicos
```
Ejemplo: "Allow ec2:StartInstances on resource arn:aws:ec2:*:*:instance/*"
```

**Claves de Acceso:** Credenciales programáticas (como contraseñas para código)
```
Ejemplo: AKIAIOSFODNN7EXAMPLE (ID de clave de acceso)
         wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY (clave secreta)
```

#### **El Escenario de Brecha de IAM:**

```
Paso 1: Desarrollador hace commit de claves de acceso de AWS a GitHub
        (Visibilidad pública)
        ↓
Paso 2: Atacante encuentra claves vía herramientas de escaneo de GitHub
        ↓
Paso 3: Atacante usa claves para acceder a tu cuenta de AWS
        ↓
Paso 4: Atacante enumera permisos
        Ejecuta: aws iam get-user
        ↓
Paso 5: Atacante abusa de permisos
        Ejemplo: Si existe permiso "ec2:*", atacante puede:
        - Lanzar nuevas instancias EC2
        - Acceder a datos
        - Crear acceso de puerta trasera
        ↓
Paso 6: Atacante cubre sus rastros
        Elimina logs de CloudTrail (si tienen permisos excesivos)
        ↓
Paso 7: Descubres semanas después cuando AWS envía factura de $50K
```

**Detección:** CloudTrail muestra acceso desde IP desconocida con credenciales expuestas

---

## ⚙️ Lo que DEBES Memorizar

### **Truco de Memoria: CIA-SR (CIA + Responsabilidad Compartida)**
- **C** = Confidentiality (solo personas autorizadas ven datos)
- **I** = Integrity (datos no modificados por atacantes)
- **A** = Availability (servicios están UP y accesibles)
- **SR** = Shared Responsibility (TÚ + Proveedor ambos responsables)

### **Los 5 Pilares de Seguridad en la Nube:**

1. **Identidad y Acceso (IAM)**
   - Quién puede acceder qué
   - Principio de mínimo privilegio
   - Autenticación multifactor (MFA)

2. **Seguridad de Red**
   - Aislamiento VPC (Virtual Private Cloud)
   - Security groups (reglas de firewall)
   - Segmentación de red

3. **Protección de Datos**
   - Encriptación en reposo (datos almacenados)
   - Encriptación en tránsito (datos en movimiento)
   - Gestión de claves

4. **Cumplimiento y Gobernanza**
   - Cumplir estándares de industria (HIPAA, PCI-DSS, SOC2)
   - Logging de auditoría (CloudTrail, etc.)
   - Aplicación de política

5. **Detección de Amenazas y Respuesta**
   - Detectar acceso no autorizado
   - Responder a incidentes
   - Forensia en entorno de nube

### **El Rol del SOC en Híbrido (On-Premise + Cloud):**

```
┌─────────────────────────────────────┐
│ Tu Data Center On-Premise           │
│  ├─ Servidores (Windows, Linux)    │
│  ├─ Firewalls                       │
│  ├─ SIEM (recolectando logs)        │
│  └─ Equipo de Respuesta a Incidentes│
└──────────────┬──────────────────────┘
               │ Conexión Segura (VPN/Private Link)
               │
┌──────────────▼──────────────────────┐
│ Entorno de Nube (AWS/Azure)         │
│  ├─ Instancias EC2                  │
│  ├─ Bases de Datos                  │
│  ├─ Logging de Nube (CloudTrail)    │
│  └─ Monitoreo basado en nube        │
└──────────────────────────────────────┘
               │
┌──────────────▼──────────────────────┐
│ Dashboard SOC Centralizado          │
│  ├─ Logs on-premise                 │
│  ├─ Logs de nube (alimentados a SIEM)
│  ├─ Alertas de ambos                │
│  └─ Respuesta a incidentes unificada│
└──────────────────────────────────────┘
```

---

## 📚 Lo que DEBES Entender

### **Puntos de Comprensión Profunda:**

#### **1. El Logging No es Automático en la Nube**
- **On-Premise:** Los logs del servidor existen por defecto. Ves todo localmente.
- **Nube:** DEBES HABILITAR el logging.
  - CloudTrail (llamadas de API de AWS) - debe habilitarse
  - VPC Flow Logs (tráfico de red) - debe habilitarse
  - Logs de Aplicación - DEBES configurar
  - Logs de Base de Datos - DEBES habilitar

**Consecuencia:** Si el logging no está habilitado, no sabrás que fuiste atacado hasta que datos aparezcan en la dark web.

#### **2. La Visibilidad está Limitada por la Configuración**
```
Escenario 1: Habilitas CloudTrail
  → Ves "cuenta root accedida desde IP 200.1.1.1"
  → Sabes que fuiste comprometido
  
Escenario 2: No habilitaste CloudTrail
  → Ves factura de AWS de $100K
  → Sabes que ALGO sucedió pero no qué
```

#### **3. La Forensia en la Nube es Diferente**
- **On-Premise:** Desconecta cable de red, preserva evidencia
- **Nube:** Si aíslas instancia, pierdes datos forenses (logs aún en AWS, pero logs de aplicación se pierden)
- **Mejor Práctica:** Crea snapshot ANTES de aislar, investiga snapshot después

#### **4. El Costo como Señal de Detección**
- **Factura Normal de AWS:** $5K/mes
- **Con Minador Agregado:** $50K/mes
- **Aumento de costo = posible ataque** (pero detección retrasada)

#### **5. La Paradoja de la Responsabilidad Compartida**
- Eres responsable de la seguridad, pero no posees el hardware
- AWS es responsable de la infraestructura, pero no puede proteger tu misconfiguration
- **Ejemplo:** S3 bucket accidentalmente configurado como público
  - AWS aseguró la infraestructura ✓
  - Misconfiguraste el bucket ✗
  - **Resultado:** Tus datos se filtraron, pero es TU culpa

---

## 🚨 Aplicación Práctica / Ataque y Defensa

### **Escenario 1: Exposición de Credenciales vía Commit de GitHub**

**Flujo de Ataque:**
```
1. Desarrollador escribe código para conectarse a BD RDS de AWS
2. Hard-codea claves de acceso de AWS en el código: 
   AKIAIOSFODNN7EXAMPLE:wJalrXUtnFEMI/K7...
3. Desarrollador hace push del código a GitHub
4. Repo está accidentalmente público (o atacante tiene acceso)
5. Atacante encuentra credenciales vía herramientas de escaneo de GitHub
6. Atacante configura AWS CLI con claves robadas
7. Atacante ejecuta: aws ec2 describe-instances
8. Atacante ve que la compañía tiene BD con datos de cliente
9. Atacante ejecuta: aws s3 ls s3://company-backups/
10. Atacante descarga backups de BD de cliente
11. Atacante elimina backups para cubrir rastros
```

**Detección (Lo que tu SOC debería ver):**

```
Si CloudTrail está habilitado:
  → Acceso a API de AWS desde IP extranjera
  → s3:GetObject en bucket de backups
  → s3:DeleteObject en bucket de backups
  → Todo originándose del mismo usuario (credenciales comprometidas)
  
Si CloudTrail NO está habilitado:
  → No sabrás hasta que:
     - AWS te contacte sobre actividad sospechosa
     - Los atacantes vendan datos en dark web
     - Los clientes reporten que sus datos se filtraron
     - La compañía sea demandada
```

**Defensa (Respuesta del Blue Team):**

```
Fase de Detección:
  1. Alerta de CloudTrail: "Usuario no autorizado accedió a bucket de s3 desde 200.1.1.1"
  2. Verifica: ¿Es IP propiedad de la compañía? No → sospechoso
  3. Investiga: ¿Qué fue accedido/descargado?
  
Fase de Contención:
  1. Inmediatamente deshabilita claves de acceso expuestas
  2. Revoca tokens de sesión temporal
  3. Si datos fueron descargados, inicia investigación de brecha de datos
  
Fase de Investigación:
  1. Revisión de logs de CloudTrail para línea de tiempo
  2. ¿Qué datos fueron accedidos?
  3. ¿Por cuánto tiempo tuvo acceso el atacante?
  4. ¿Qué más accedieron?
  5. ¿Este usuario específico de IAM tiene otros entornos?
  
Fase de Erradicación:
  1. Rota todas las credenciales
  2. Escanea repositorio de código fuente para claves expuestas
  3. Implementa gestión de secretos (AWS Secrets Manager)
  4. Habilita MFA para todas las cuentas de AWS
  5. Implementa escaneo de credenciales en pipeline de CI/CD
  
Fase de Recuperación:
  1. Notifica a clientes de posible brecha
  2. Fuerza reinicio de contraseña para cuentas afectadas
  3. Implementa monitoreo en ese bucket de S3
  4. Revisa todos los permisos de usuario de IAM (mínimo privilegio)
```

---

### **Escenario 2: Política de IAM Excesivamente Permisiva**

**Misconfiguration:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    }
  ]
}
```

Esta política dice: "Este usuario puede hacer CUALQUIER COSA con instancias EC2, en cualquier lugar."

**Flujo de Ataque:**
```
1. Desarrollador junior comprometido (malware en laptop)
2. Atacante obtiene credenciales de AWS del desarrollador
3. Atacante verifica permisos: aws iam get-user-policy
4. Atacante ve permiso "ec2:*"
5. Atacante lanza 1000 instancias EC2 para minería de cripto
   Comando: aws ec2 run-instances --image-id ami-12345 --count 1000
6. Cada instancia minería criptomonedas por 1 semana
7. Recibes factura de AWS de $100K
```

**Detección:**
```
Qué debería disparar alertas:
  → CloudTrail muestra ec2:RunInstances x 1000 de un usuario
  → Utilización de CPU de EC2 pico
  → Alto tráfico de red saliente (comunicación de piscina de minería)
  → Comportamiento inusual del usuario (desarrollador ejecutando comandos de producción)
  
Si las alertas están configuradas:
  → El SOC atrapa en 1-5 minutos
  
Si las alertas NO están configuradas:
  → Descubres a fin de mes cuando la factura llega
```

**Defensa:**
```
Prevención:
  1. Implementa mínimo privilegio:
     Solo permite acciones específicas necesarias
  2. Etiquetado de recursos + límites
     Limita instancias EC2 creadas por día
  3. MFA requerida para acciones sensibles
  
Detección:
  1. Monitorea ec2:RunInstances para volumen inusual
  2. Alerta en CPU > 90% por tiempo extendido
  3. Monitorea tráfico saliente a piscinas de minería (inteligencia de amenazas)
  
Respuesta:
  1. Termina inmediatamente todas las instancias maliciosas
  2. Deshabilita credenciales comprometidas
  3. Fuerza reinicio de contraseña
  4. Análisis forense de laptop del desarrollador
  5. Implementa controles de acceso mejorados
```

---

### **Escenario 3: S3 Bucket Dejado Público (Misconfiguration)**

**Configuración:**
```
La compañía almacena PII de cliente en bucket S3: 
s3://company-customers/

Un desarrollador configura bucket con:
  - Block Public Access = OFF
  - Bucket Policy permite acceso público
  
Resultado: Cualquiera puede acceder datos de cliente
```

**Descubrimiento:**
```
Opción 1 (Proactivo - Tú lo atrapas):
  Regla de AWS Config verifica permisos de bucket
  Alerta dispara: "S3 bucket es públicamente accesible"
  El SOC investiga y arregla
  
Opción 2 (Reactivo - Ellos lo atrapan):
  Investigador de seguridad encuentra bucket vía escaneo
  Publica en Twitter: "Compañía XYZ filtró datos de cliente"
  Cobertura de medios, multa regulatoria, demandas
```

**Detección y Respuesta:**
```
Paso 1: Alerta de AWS Config o auditoría manual
  → Identifica bucket público de S3
  
Paso 2: Determina qué contiene
  → AWS Access Analyzer muestra: "10 millones de objetos públicamente legibles"
  
Paso 3: Evalúa exposición
  → Verifica CloudTrail: ¿Ha accedido alguien?
  → Verifica logs de acceso de S3 (si habilitados)
  
Paso 4: Contiene
  → Bloquea acceso público inmediatamente
  → Habilita versionado para preservar estado actual
  
Paso 5: Investiga
  → Revisa quién creó el bucket
  → ¿Por qué era público?
  → ¿Qué datos fueron expuestos?
  
Paso 6: Notifica
  → Si PII expuesto: notifica a clientes
  → Si datos de tarjeta de pago: notifica procesador de pago
  → Si datos de salud: notificación HIPAA
  
Paso 7: Previene
  → Hace cumplir políticas de bucket vía SCPs (Service Control Policies)
  → Auditorías regulares
  → Capacitación de desarrolladores
```

---

## ❌ Errores Comunes que Cometen los Estudiantes

### **Error 1: Pensar que el Proveedor de Nube Asegura Tus Datos**
- ❌ **Incorrecto:** "AWS asegura mis bases de datos, así que están seguras"
- ✅ **Correcto:** "AWS asegura la infraestructura de RDS. Debo asegurar: credenciales, acceso de red, claves de encriptación, backups"
- **Consecuencia Real:** Credenciales de BD sin asegurar, cualquiera en la compañía puede acceder BD de producción

### **Error 2: Olvidar que el Logging Debe Ser Habilitado**
- ❌ **Incorrecto:** "AWS registra automáticamente todo"
- ✅ **Correcto:** "AWS proporciona servicios de logging. Debo habilitar CloudTrail, VPC Flow Logs, etc."
- **Consecuencia Real:** Incidente ocurre, no tienes evidencia de qué sucedió

### **Error 3: Políticas de IAM Excesivamente Permisivas**
- ❌ **Incorrecto:** "Dale al desarrollador ec2:* para que pueda hacer su trabajo"
- ✅ **Correcto:** "Dale al desarrollador solo ec2:DescribeInstances y ec2:StartInstances"
- **Consecuencia Real:** Credenciales comprometidas del desarrollador = compromiso total de cuenta

### **Error 4: Ignorar el Costo como Señal de Detección**
- ❌ **Incorrecto:** "¿Factura inesperada de $100K? Solo cárgala al departamento"
- ✅ **Correcto:** "Aumento de costo = posible uso no autorizado de recursos. Investiga inmediatamente."
- **Consecuencia Real:** Minero de cripto corre durante semanas antes de descubrirse

### **Error 5: No Implementar MFA en Cuentas de Nube**
- ❌ **Incorrecto:** "La contraseña es suficientemente fuerte"
- ✅ **Correcto:** "Incluso contraseñas fuertes pueden ser phishing. MFA previene secuestro de cuenta."
- **Consecuencia Real:** Compromiso de credencial única = compromiso total de cuenta

---

## 🧪 Práctica/Análisis

### **Caso a Analizar:**

**Caso:** Tu SOC recibe alerta de CloudTrail a las 2:00 AM

```
Evento: iam:AttachUserPolicy
Usuario: dev-user@company.com
Tiempo: 2024-01-15 02:00:15 UTC
IP Fuente: 203.0.113.45 (conocida por estar en Rusia)
Acción: Adjuntó política "AdministratorAccess" a dev-user

Eventos relacionados (misma sesión):
  1. iam:CreateAccessKey (creó nuevas credenciales)
  2. s3:GetObject (accedió a backup de BD de cliente)
  3. s3:GetObject (accedió a archivo de lote de tarjeta de pago)
  4. iam:ListUsers (enumeró todos los usuarios)
  5. ec2:DescribeInstances (enumeró todas las instancias)
```

### **Preguntas a Responder:**

1. **¿Es esto un ataque real o falso positivo?**
   - Banderas rojas: Acceso a las 2 AM, IP de Rusia, desarrollador normalmente no trabaja esta hora
   - Pistas: Atacante está enumerando permisos (ListUsers, DescribeInstances)
   - Respuesta: **Alta probabilidad de ataque real**

2. **¿Cuál es el objetivo del atacante?**
   - Adjuntó AdminAccess = quiere acceso total de cuenta
   - Descargó datos sensibles = robo de datos
   - Creó clave de acceso = persistencia
   - Respuesta: **Acceso inicial a infraestructura + exfiltración de datos**

3. **¿Cómo respondes?**
   ```
   Inmediato (0-5 minutos):
     1. Deshabilita credenciales de dev-user
     2. Invalida todas las sesiones activas
     3. Alerta al equipo de respuesta a incidentes
     4. Verifica si datos fueron descargados (logs de acceso de S3)
   
   Corto Plazo (5-30 minutos):
     1. Revisa todos los eventos de CloudTrail para este usuario en últimas 24 horas
     2. Verifica si otros usuarios fueron comprometidos
     3. Asegura todas las claves de acceso
     4. Implementa política de denegación en bucket de datos de cliente
   
   Mediano Plazo (30 min-4 horas):
     1. Análisis forense completo de CloudTrail
     2. Determina si exfiltración de datos ocurrió
     3. Contacta equipo de seguridad de AWS
     4. Prepara notificación de brecha si es necesaria
   ```

4. **¿Qué persistencia creó el atacante?**
   - Nueva clave de acceso (puede acceder cuenta incluso si contraseña se resetea)
   - Política AdminAccess (acceso de alta autoridad)
   - Si no es atrapado: Cuenta de puerta trasera para acceso futuro

5. **¿Cómo lo previene la próxima vez?**
   - MFA en todos los usuarios de IAM (este ataque requiere robo de contraseña)
   - Alerta en cualquier adjunción de política fuera de horario comercial
   - Alerta en acceso desde geografías inusuales
   - Deshabilita claves de acceso no usadas
   - Revisiones regulares de acceso

### **Soluciones:**

**Contención Inmediata:**
```bash
# Comandos de AWS CLI para responder
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE --user-name dev-user
aws iam detach-user-policy --user-name dev-user --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-login-profile --user-name dev-user --password <nueva-contraseña-temporal> --password-reset-required
```

**Investigación:**
```
Revisa CloudTrail para:
  - Línea de tiempo: ¿Cuándo primero accedió el atacante?
  - Alcance: ¿Cuántos usuarios/recursos afectados?
  - Datos: ¿Qué fue descargado?
  - Persistencia: ¿Qué puertas traseras creó?
```

**Comunicación:**
- Notifica: CTO, equipo de Seguridad, Legal, soporte de clientes
- Si brecha de datos: Prepara notificaciones a clientes
- Si datos de cumplimiento (PII, HIPAA, PCI): Notifica reguladores

---

## 🎯 Preguntas de Entrevista que Podrías Recibir

### **Nivel 1 (Principiante):**

**P1:** "Explica el modelo de responsabilidad compartida en seguridad de nube"
- **Respuesta Esperada:**
  - Proveedor de nube asegura infraestructura (data centers, hipervisores, redes)
  - Cliente asegura configuraciones, IAM, protección de datos, aplicaciones
  - Ejemplo: AWS asegura hardware EC2, tú aseguras parches del OS de EC2
  - Diferente para servicios gestionados: AWS RDS = AWS maneja OS; tú manejas configuración de BD

**P2:** "¿Qué es CloudTrail y por qué un SOC lo necesita?"
- **Respuesta Esperada:**
  - CloudTrail = servicio de logging de auditoría de AWS
  - Registra TODAS las llamadas de API hechas a cuenta de AWS
  - Esencial para: Detectar acceso no autorizado, forensia, cumplimiento
  - Sin él: Sin visibilidad en quién hizo qué en AWS

**P3:** "Descubriste $50K en cargos inesperados de AWS. ¿Qué haces?"
- **Respuesta Esperada:**
  - No lo ignores—podría ser ataque (minero de cripto, secuestro de recursos)
  - Verifica CloudTrail para llamadas de API inusuales
  - Revisa instancias EC2 (¿hay instancias desconocidas?)
  - Revisa uso de S3/RDS (¿transferencia de datos inusual?)
  - Investiga y termina recursos maliciosos

---

### **Nivel 2 (Nivel Medio):**

**P1:** "Cuéntame cómo detectarías un compromiso de credencial de IAM en nube"
- **Respuesta Esperada:**
  - Habilita: CloudTrail, VPC Flow Logs, GuardDuty (detección de amenazas de AWS)
  - Busca: Acceso desde IP inusual, llamadas de API inusuales, intentos de escalamiento de privilegios
  - Línea de tiempo: ¿Cuándo se expusieron credenciales? ¿Cuánto tiempo estuvo activo atacante?
  - Impacto: ¿Qué recursos fueron accedidos? ¿Qué datos fueron tocados?
  - Respuesta: Deshabilita credenciales inmediatamente, audita toda actividad, implementa MFA

**P2:** "¿Cuál es la diferencia entre detectar ataque on-premise vs. nube?"
- **Respuesta Esperada:**
  - On-Premise: Acceso directo al servidor, logs de firewall, logs de endpoint
  - Nube: Logs de API (CloudTrail), logs de red (VPC Flow Logs), depende de configuración
  - Desafío: Menos visibilidad por defecto—debe habilitar logging
  - Ventaja: Logs de proveedor de nube centralizados más fáciles de correlacionar
  - Forensia: On-premise = dump de memoria; Nube = preservar snapshots

**P3:** "Diseña una regla de detección para exposición de credenciales en GitHub"
- **Respuesta Esperada:**
  - Monitorea: Pushes de repositorio de GitHub para patrones de credenciales (regex para claves de AWS)
  - Alerta: Si claves de AWS detectadas en commit
  - Respuesta: Inmediatamente revoca claves expuestas
  - Integra: Escaneo de secretos de GitHub + monitoreo de CloudTrail de AWS
  - Prevención: Gestión de secretos (AWS Secrets Manager, HashiCorp Vault)

---

### **Nivel 3 (Senior / Entrevista Avanzada):**

**P1:** "Las credenciales de un desarrollador fueron comprometidas. Cuéntame cómo determinarías el alcance de la brecha en entorno de nube"
- **Respuesta Esperada:**
  - Análisis de CloudTrail: Todas las llamadas de API hechas con credenciales comprometidas
  - Reconstrucción de línea de tiempo: Cuándo expuesto → cuándo usado → cuándo revocado
  - Inventario de recursos: ¿Qué EC2, S3, RDS, bases de datos fueron accedidos?
  - Exposición de datos: Analiza CloudTrail para ver qué datos fueron descargados
  - Verificación de persistencia: ¿Creó atacante puertas traseras? ¿Nuevos usuarios de IAM? ¿Claves de acceso?
  - Análisis de red: VPC Flow Logs para ver tráfico inusual
  - Análisis multi-nube: Si es híbrido, verifica todos los entornos de nube
  - Imagen forense: Crea snapshot de instancias afectadas para análisis profundo

**P2:** "Tu organización se está moviendo de on-premise a nube. ¿Cómo cambian tus procesos de respuesta a incidentes?"
- **Respuesta Esperada:**
  - Fuentes de log cambian: SIEM aún central, pero se alimenta de CloudTrail, VPC Flow Logs
  - Visibilidad: Logging no es automático—debe habilitar
  - Forensia: Basado en snapshots en lugar de acceso directo a disco
  - Aislamiento: Aislamiento de red vía security groups (no aislamiento físico)
  - Recuperación: Snapshots de AMI en lugar de backups de cinta
  - Responsabilidad compartida: Algunos incidentes ahora son problema de AWS, no tuyo
  - Cumplimiento: Requisitos de auditoría diferentes (SOC 2, FedRAMP, etc.)
  - Comunicación: Necesita contactos de respuesta a incidentes del proveedor de nube

**P3:** "Explica cómo detectarías y responderías a una amenaza persistente avanzada (APT) en infraestructura híbrida"
- **Respuesta Esperada:**
  - Capas de detección:
    - EDR de endpoint (on-premise + VMs de nube)
    - Monitoreo de red (interno + red de nube)
    - Logs de nube (CloudTrail, VPC Flow Logs)
    - Logs de aplicación
    - Análisis de comportamiento (patrones de acceso inusual)
  - Características de APT a buscar:
    - Movimiento lateral entre entorno híbrido
    - Living off the Land (usando herramientas legítimas)
    - Mecanismos de persistencia (puertas traseras de nube + on-premise)
    - Preparación de datos para exfiltración
  - Respuesta:
    - Respuesta a incidentes debe coordinar on-premise + equipos de nube
    - Contención: Aislados sistemas on-premise Y de nube
    - Erradicación: Elimina persistencia de TODOS los entornos
    - Forensia: Recopila evidencia de on-premise y nube

---

## 🔗 Cómo Esto se Conecta con Todo lo Demás

- **Respuesta a Incidentes:** La mayoría de incidentes ahora involucran nube. Los playbooks de IR deben cubrir pasos específicos de nube.
- **SIEM y Análisis de Logs:** Logs de CloudTrail deben ser ingeridos en tu SIEM para correlación con logs on-premise.
- **Forensia:** Forensia de nube es diferente (snapshots vs. acceso directo). La cadena de custodia importa.
- **Cumplimiento:** Estándares diferentes para nube (SOC 2, certificaciones de cumplimiento de AWS).
- **IAM/Control de Acceso:** IAM en nube es fundación. Débil IAM = compromiso total de cuenta.
- **Seguridad de Red:** Aislamiento de VPC, security groups, WAF (Web Application Firewall) para apps de nube.
- **Protección de Datos:** Encriptación en reposo + en tránsito en nube es TU responsabilidad (no de AWS por defecto).
- **Análisis de Malware:** Malware en nube = análisis diferente (no puedes acceder directamente a instancia infectada).
- **Threat Hunting:** Threat hunt en nube usando CloudTrail, no análisis de sistema de archivos.

---

## 💾 Resumen para Gente Ocupada

| Concepto | Definición | Método de Detección | Respuesta |
|---|---|---|---|
| **Responsabilidad Compartida** | AWS asegura infraestructura, tú aseguras configuración | Entiende límite de quién es responsable | Playbooks de respuesta a incidentes diferentes |
| **Compromiso de IAM** | Atacante obtiene credenciales de AWS | CloudTrail muestra llamadas de API desde IP desconocida | Deshabilita credenciales, audita toda actividad |
| **CloudTrail** | Servicio de logging de auditoría de AWS | Habilita CloudTrail en todas las regiones | Correlaciona logs en SIEM |
| **VPC Flow Logs** | Logging de tráfico de red en nube | Habilita en todas las subredes | Detecta exfiltración de datos, conexiones inusuales |
| **Misconfiguration de S3** | Bucket dejado públicamente accesible | Verificaciones de AWS Config para políticas de bucket | Restringe acceso, audita qué fue expuesto |
| **Exposición de Credenciales** | Claves de acceso encontradas en código fuente | Escaneo de secretos de GitHub + CloudTrail | Revoca claves inmediatamente, rota |
| **Aumento de Costo** | Factura de AWS inesperada aumentada | Monitorea costo mensual vs. baseline | Investiga minería de cripto, secuestro de recursos |
| **Logging No Habilitado** | Eventos de nube no registrados | Verifica si CloudTrail/Flow Logs habilitados | Habilita inmediatamente, no puedes hacer forensia sin logs |
| **Escalamiento de Privilegios** | Atacante obtiene acceso de admin | CloudTrail muestra iam:AttachUserPolicy | Revoca permisos, audita todos los usuarios |

---

## 📌 Realidad de Producción

### **En un SOC Real con Infraestructura Híbrida:**

**Día 1: Configuración Inicial**
1. **SIEM On-Premise:** Recibe logs de Windows, Linux, firewall
2. **Cuenta de AWS Creada:** Equipo de desarrollo despliega aplicación
3. **Primer Problema:** El SOC no sabe sobre AWS aún (brecha de comunicación)
4. **Acción Necesaria:** Configurar CloudTrail, alimentar a SIEM

**Día 30: Primera Alerta de Nube**
1. **Detección:** CloudTrail muestra llamadas de API inusuales
2. **Investigación:** ¿Fue esto autorizado? Verifica con equipo de dev.
3. **Frecuentemente:** "Oh sí, deployamos eso a las 3 AM"
4. **Lección:** Necesita mejor comunicación entre SOC + equipos de dev

**Día 90: Primer Incidente de Nube**
1. **Alerta:** S3 bucket dejado público con datos sensibles
2. **Descubrimiento:** Vía investigador de seguridad, no tu SOC
3. **Cobertura de Medios:** "Compañía Filtra Datos de Cliente"
4. **Consecuencia:** Implementa reglas de AWS Config, auditorías regulares

**Día 180: Seguridad Híbrida Madura**
1. **Logging Centralizado:** Todos los logs on-premise + nube en SIEM
2. **Detección Automatizada:** Reglas para amenazas específicas de nube
3. **Auditorías Regulares:** Análisis de CloudTrail, revisiones de IAM, verificaciones de bucket de S3
4. **Respuesta a Incidentes:** Playbooks para incidentes de nube
5. **Cumplimiento:** Rastros de auditoría para SOC2, etc.

### **Desafíos Reales que tu SOC Enfrentará:**

1. **Volumen de Logs:** CloudTrail = millones de eventos. SIEM debe escalar.
2. **Costo:** Facturas de AWS aumentan si logging es extenso. Licencias de SIEM aumentan.
3. **Brecha de Habilidad:** No todos los analistas de SOC entienden amenazas específicas de nube.
4. **Confusión de Responsabilidad Compartida:** Los desarrolladores piensan que AWS los protegerá.
5. **Retraso de Detección:** Logs de nube a veces retrasados minutos. Necesita alternativas en tiempo real.

---

## 📚 Lecturas Adicionales

### **Específicas de AWS:**
- Documentación de AWS CloudTrail: https://docs.aws.amazon.com/cloudtrail/
- Mejores Prácticas de IAM de AWS: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Mejores Prácticas de Seguridad de AWS: https://aws.amazon.com/architecture/security-identity-compliance/

### **Específicas de Azure:**
- Log de Actividad de Azure: https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log
- Mejores Prácticas de Seguridad de Azure: https://learn.microsoft.com/en-us/security/

### **Google Cloud:**
- Logs de Auditoría de Google Cloud: https://cloud.google.com/logging/docs/audit
- Seguridad de Google Cloud: https://cloud.google.com/security

### **Seguridad de Nube General:**
- Cloud Security Alliance: https://cloudsecurityalliance.org/
- NIST Cloud Computing Security: https://csrc.nist.gov/projects/cloud-computing/
- OWASP Cloud-Native Security: https://owasp.org/

### **Herramientas para Aprender:**
- AWS CLI (acceso línea de comandos a AWS)
- CloudTrail Insights (detección de amenazas integrada de AWS)
- GuardDuty (detección de amenazas de AWS)
- AWS Config (monitoreo de cumplimiento)
- VPC Flow Logs (monitoreo de red)
- Terraform (Infraestructura como Código - nube)

### **Tus Próximos Pasos:**
1. Solicita entorno de sandbox de AWS de tu organización
2. Habilita CloudTrail en todas las cuentas de AWS
3. Crea reglas de detección para: compromiso de IAM, secuestro de recursos, exfiltración de datos
4. Practica: Playbooks de respuesta a incidentes de AWS
5. Estudia: Modelo de responsabilidad compartida para cada proveedor de nube
6. Implementa: Políticas de IAM de mínimo privilegio
7. Monitorea: Anomalías de costo como señal de detección

---

**Documento Generado:** Julio 2026  
**Para:** Portafolio de Blue Team de Ciberseguridad (GitHub)  
**Nivel:** Analista Principiante a Nivel Medio  
**Estado:** Listo para Uso en Portafolio ✅
