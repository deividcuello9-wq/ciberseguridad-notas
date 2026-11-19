# Análisis de Alertas SOC — TryHackMe SOC Simulator

**Autor:** deividcuello9-wq
**Entorno:** TryHackMe SOC Simulator (SIEM: Splunk)
**Rol:** Analista SOC — triaje e investigación de alertas
**Fecha:** Noviembre 2026

---

## Resumen

Investigación de dos alertas en un entorno SOC simulado usando Splunk como SIEM y una herramienta de análisis de reputación de URLs/dominios. El ejercicio demuestra el flujo completo de un analista: **asignar → investigar → verificar → clasificar → documentar**, y la capacidad de distinguir un falso positivo de un verdadero positivo con evidencia.

Metodología aplicada en ambos casos:
1. Asignación de la alerta y lectura de indicadores.
2. Pivote en el SIEM (Splunk) para reunir contexto.
3. Verificación de reputación de dominios/URLs en una herramienta de análisis.
4. Clasificación basada en evidencia y redacción del informe.

---

## Caso 1 — Alerta 8814: Correo con enlace externo (FALSO POSITIVO)

**Tipo:** Suplantación de identidad (phishing) · **Gravedad:** Media

### Indicadores
- Remitente: `onboarding@hrconnex.thm`
- Destinataria: `j.garcia@thetrydaily.thm`
- URL: `https://hrconnex.thm/onboarding/<id>/j.garcia`
- Asunto: "Finaliza tu perfil de incorporación"
- Adjunto: ninguno

### Investigación
La alerta se disparó porque el dominio del remitente (`hrconnex.thm`) es externo y distinto al de la empresa (`thetrydaily.thm`), pese a que el correo se presentaba como de RR.HH. interno — un patrón típico de phishing.

Al pivotar en Splunk buscando el dominio `hrconnex.thm` aparecieron 3 eventos, incluyendo un **correo interno** de un empleado al equipo de IT que indicaba que `hrconnex.thm` era un **proveedor externo de RR.HH. recién contratado** y que la empleada esperaba ese correo de incorporación.

### Verificación
- Reputación del remitente y de la URL: **CLEAN** (sin indicadores maliciosos).
- Registros de firewall para `hrconnex.thm`: **ninguno** → nadie hizo clic; sin impacto.

### Clasificación: FALSO POSITIVO
La detección se activó correctamente por un enlace externo, pero la investigación demostró que era actividad legítima: un proveedor nuevo no reconocido por la regla.

### Recomendación
Verificar con IT/RR.HH. que `hrconnex.thm` es un proveedor autorizado y añadirlo a la lista blanca. Anunciar proveedores nuevos al personal antes de su primer envío para evitar futuras falsas alarmas.

### Lección
Una afirmación dentro de un correo ("es nuestro proveedor") no es prueba; hay que verificar por canales independientes. Ante la duda, no se marca algo como seguro sin confirmarlo.

---

## Caso 2 — Alerta 8816: URL externa bloqueada por firewall (VERDADERO POSITIVO)

**Tipo:** Cortafuegos · **Gravedad:** Alta

### Indicadores
- Acción: `blocked` (bloqueado)
- Host de origen: `10.20.2.17` (puerto 34257)
- IP de destino: `67.199.248.11` (puerto 80)
- URL: `http://bit.ly/<id>` (enlace acortado)
- Regla: Sitios web bloqueados (lista negra)

### Investigación
Alerta por conexión **saliente**: un host interno intentó acceder a una URL acortada en lista negra. A diferencia del phishing entrante, aquí la pregunta es por qué un equipo propio intentó salir a un sitio malicioso.

Pivote en Splunk por el host `10.20.2.17` → solo 2 eventos en total:
1. **15:18:36** — `allowed`: búsqueda en Google "how to set up payroll system for small business".
2. **15:20:58** — `blocked`: intento de abrir el enlace `bit.ly` malicioso.

La secuencia indica comportamiento **humano**: el usuario investigaba software de nómina y pulsó un enlace acortado malicioso (resultado o anuncio). La ausencia de conexiones repetidas descarta un patrón de *beaconing* de malware.

### Verificación
- Reputación de la URL: **MALICIOUS** (confirmado).
- Acción del firewall: **blocked** → conexión impedida; sin impacto.
- Solo 2 eventos del host, sin señales de compromiso.

### Clasificación: VERDADERO POSITIVO
La actividad maliciosa ocurrió realmente (intento confirmado hacia una URL maliciosa) y la detección acertó. El impacto fue nulo gracias al bloqueo del firewall.

### Recomendación
Mantener la URL/dominio en lista negra. Concienciar al usuario del host `10.20.2.17` sobre enlaces acortados y anuncios en buscadores. Monitorear el host por posible actividad posterior.

### Lección
Un bloqueo exitoso sigue siendo un verdadero positivo: la clasificación depende de si la amenaza era real, no de si tuvo impacto. Los enlaces acortados (bit.ly) ocultan el destino y son un indicador de sospecha.

---

## Conclusión

| Caso | Tipo | Veredicto | Clave |
|------|------|-----------|-------|
| 8814 | Phishing (entrante) | Falso positivo | Proveedor legítimo confirmado por contexto en el SIEM |
| 8816 | Firewall (saliente) | Verdadero positivo | URL maliciosa confirmada; bloqueada sin impacto |

El valor del analista no está en cerrar alertas rápido, sino en reunir evidencia suficiente para clasificar correctamente y recomendar acciones. El mismo indicador superficial (un dominio externo) puede ser benigno o malicioso; solo la investigación lo determina.
