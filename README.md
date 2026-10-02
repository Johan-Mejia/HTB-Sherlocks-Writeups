# Hack The Box Sherlocks Writeups & DFIR Analysis 🔍🛡️

Bienvenido a mi repositorio dedicado a la resolución e investigación de casos en los laboratorios **Sherlocks** de Hack The Box. Este espacio recopila informes detallados sobre **Análisis Forense Digital y Respuesta a Incidentes (DFIR)**, **Monitoreo SOC**, **Threat Hunting** y **Análisis de Artefactos**.

---

## 📌 Lista de Sherlocks Investigados

| Caso / Sherlock | Categoría | Dificultad | Enfoque de Análisis | Herramientas Utilizadas | Writeup / Reporte |
|---|---|---|---|---|---|
| *(Próximamente)* | DFIR / SOC | Very Easy / Easy | Análisis de logs de eventos / Captura de red | Wireshark, Volatility, EVTX ECM | [Ver Informe](./Sherlock-Nombre) |

---

## 🎯 Metodología de Análisis (DFIR / Blue Team)

En cada informe aplico una metodología estructurada de investigación defensiva:

1. **Triage Inicial y Recolección de Artefactos:** Identificación del alcance del incidente y preservación de evidencias (PCAPs, EVTX, dumps de memoria RAM, registros de auditoría).
2. **Línea de Tiempo del Incidente (Timeline Analysis):** Reconstrucción cronológica de los eventos para determinar la ventana del compromiso.
3. **Análisis de Vectores de Ataque & Mapeo MITRE ATT&CK:** Identificación de las técnicas, tácticas y procedimientos (TTPs) utilizados por el atacante.
4. **Determinación de Indicadores de Compromiso (IoCs):** Extracción de IPs maliciosas, hashes de archivos, dominios C2 y persistencias en el sistema.
5. **Recomendaciones y Remediación:** Medidas de hardening, reglas de detección (YARA/Sigma) y mitigaciones para neutralizar la amenaza.

---

## 🧰 Herramientas de Investigación Frecuentes

* **Análisis de Red:** Wireshark, tshark, Zeek/Bro, Brim.
* **Análisis Forense Windows/Linux:** Eric Zimmerman Tools (KAPE, EZ Tools), EvtxECmd, Volatility 3, Autopsy.
* **Análisis de Malware / Logs:** CyberChef, YARA, Sigma Rules, PowerShell Logging.

---

> ⚠️ **Descargo de Responsabilidad:** Todos los ejercicios e informes publicados en este repositorio corresponden a entornos de prueba simulados en Hack The Box con fines educativos y de desarrollo profesional en defensas de ciberseguridad.
