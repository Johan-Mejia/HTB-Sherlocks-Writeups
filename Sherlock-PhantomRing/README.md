# Hack The Box Sherlock - PhantomRing Writeup 🔍🛡️

| Campo | Detalle |
|---|---|
| **Categoría** | Reverse Engineering / Malware Analysis / SOC |
| **Dificultad** | Very Easy |
| **Artefacto Analizado** | Binario ejecutable ELF 64-bit (`agent`) |
| **Escenario** | Análisis estático de agente de post-explotación / C2 en Linux |

---

## 📖 Escenario de la Investigación
Durante una operación de Threat Hunting en un servidor Linux, el equipo de SOC interceptó un binario sospechoso ubicado en `/var/tmp` que intentaba establecer conexiones salientes[cite: 20]. El objetivo de este laboratorio consiste en realizar un **análisis estático** sobre el archivo ejecutable (`agent`) para identificar sus capacidades, extraer Indicadores de Compromiso (IoCs), entender la infraestructura Command & Control (C2) del atacante y descubrir sus técnicas de evasión de seguridad[cite: 20].

---

## 🔍 Análisis de Tareas e Investigación del Binario

### Task 1: Hash SHA256 del Malware
* **Pregunta:** ¿Cuál es el hash SHA256 del binario malicioso?
* **Respuesta:** `2d7b1b2178f76c26893b2a56cbf9b36700235259e76b893d53817d5b66b634a5`[cite: 21]
* **Análisis:** Se calculó el digesto SHA256 del archivo mediante la utilidad de sistema:
  ```bash
  sha256sum agent
  ```

---

### Task 2: Dirección IP del Servidor C2
* **Pregunta:** ¿Cuál es la dirección IP quemada (hardcoded) en el binario para la comunicación C2?
* **Respuesta:** `192.168.56.1`[cite: 21]
* **Análisis:** Durante el análisis de cadenas de texto (*strings*) y desensamblado del binario, se identificó la dirección IP estática utilizada para establecer la conexión hacia la infraestructura C2[cite: 21].

---

### Task 3: Puerto de Conexión C2
* **Pregunta:** ¿A qué puerto se conecta el agente en el servidor C2?
* **Respuesta:** `4445`[cite: 21]
* **Análisis:** El binario configura la estructura de socket apuntando al puerto TCP `4445` para las comunicaciones salientes[cite: 21].

---

### Task 4: Intervalo de Reconexión
* **Pregunta:** ¿Cuántos segundos espera el agente antes de intentar reconectarse tras una conexión fallida?
* **Respuesta:** `120`[cite: 22]
* **Análisis:** El código fuente/desensamblado implementa un ciclo de espera (*sleep*) de 120 segundos (2 minutos) previo a reintentar el socket de red[cite: 22].

---

### Task 5: Comandos Soportados
* **Pregunta:** ¿Cuántos comandos diferentes soporta el agente (excluyendo comandos inválidos)?
* **Respuesta:** `11`[cite: 22]
* **Análisis:** Mediante la inspección de la función de parseo de comandos (vía Ghidra/IDA Pro/strings), se contabilizaron 11 instrucciones funcionales soportadas por el agente[cite: 22].

---

### Task 6: Evasión de Monitoreo EDR
* **Pregunta:** ¿Qué interfaz del kernel de Linux abusa este malware para evadir el monitoreo de llamadas al sistema (syscalls) por parte del EDR?
* **Respuesta:** `io_uring`[cite: 22]
* **Análisis:** El binario utiliza la interfaz asíncrona de E/S del kernel `io_uring`, una técnica avanzada para realizar operaciones de red y archivos evitando los hooks tradicionales de las syscalls monitoreadas por agentes EDR[cite: 22].

---

### Task 7: Enumeración de Usuarios Activos
* **Pregunta:** ¿Qué archivo lee el agente para enumerar los usuarios con sesión iniciada?
* **Respuesta:** `/var/run/utmp`[cite: 23]
* **Análisis:** El agente accede directamente al archivo de registro de sesiones en tiempo real del sistema `/var/run/utmp`[cite: 23].

---

### Task 8: Búsqueda de Binarios SUID
* **Pregunta:** ¿Qué directorio escanea el agente al buscar binarios SUID para escalación de privilegios?
* **Respuesta:** `/usr/bin`[cite: 23]
* **Análisis:** El módulo de enumeración local del malware analiza los permisos del directorio `/usr/bin` en busca de ejecutables con el bit SUID activo[cite: 23].

---

### Task 9: Detección de Herramientas de Seguridad eBPF
* **Pregunta:** ¿Qué cadena de texto busca el agente en `/proc/[pid]/maps` para identificar herramientas de seguridad que utilicen eBPF?
* **Respuesta:** `anon_inode:bpf-map`[cite: 23]
* **Análisis:** Para detectar la presencia de agentes de seguridad basados en eBPF, el binario inspecciona los mapas de memoria de los procesos buscando la firma `anon_inode:bpf-map`[cite: 23].

---

### Task 10: Desactivación de Tracing de Kernel
* **Pregunta:** ¿Cuál es la ruta completa del primer archivo de rastreo (tracing) que el agente intenta deshabilitar?
* **Respuesta:** `/sys/kernel/debug/tracing/tracing_on`[cite: 24]
* **Análisis:** Como medida de anti-análisis y evasión, el agente intenta escribir en `/sys/kernel/debug/tracing/tracing_on` para desactivar el sistema de ftrace/tracing del kernel Linux[cite: 24].

---

### Task 11: Auto-Ubicación para Auto-Destrucción
* **Pregunta:** ¿Qué ruta de `procfs` lee el agente para encontrar la ubicación de su propio ejecutable antes de auto-destruirse?
* **Respuesta:** `/proc/self/exe`[cite: 24]
* **Análisis:** Lee el enlace simbólico `/proc/self/exe` para obtener la ruta absoluta donde se está ejecutando su propio proceso en el sistema de archivos[cite: 24].

---

### Task 12: Comando de Auto-Destrucción
* **Pregunta:** ¿Qué cadena de comando compara el agente para activar la eliminación de su propio binario?
* **Respuesta:** `sdestruct`[cite: 24]
* **Análisis:** Al recibir el comando `sdestruct` desde el servidor C2, el agente ejecuta la rutina de borrado de su ejecutable en disco[cite: 24].

---

## 🎯 Indicadores de Compromiso (IoCs)

* **Hash SHA256:** `2d7b1b2178f76c26893b2a56cbf9b36700235259e76b893d53817d5b66b634a5`[cite: 21]
* **C2 IP & Port:** `192.168.56.1:4445`[cite: 21]
* **Ruta de Ejecución:** `/var/tmp/agent`[cite: 20]
* **Firma eBPF Monitoreada:** `anon_inode:bpf-map`[cite: 23]
* **Archivo de Kernel Alterado:** `/sys/kernel/debug/tracing/tracing_on`[cite: 24]

---

## 🛡️ Técnicas MITRE ATT&CK Identificadas

* **T1071.001 - Application Layer Protocol:** Comunicación C2 vía sockets personalizados[cite: 21].
* **T1070.004 - File Deletion:** Mecanismo de auto-destrucción (`sdestruct`)[cite: 24].
* **T1562.001 - Impair Defenses: Disable or Modify Tools:** Desactivación de ftrace (`tracing_on`) y evasió de EDR vía `io_uring`[cite: 22, 24].
* **T1082 - System Information Discovery:** Inspección de eBPF mapas y usuarios en `/var/run/utmp`[cite: 23].
