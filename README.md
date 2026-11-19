# 🔐 Portafolio de Ciberseguridad — Deivid

Bitácoras (*writeups*) y notas de mi laboratorio de ciberseguridad en casa. Aquí documento, paso a paso, las prácticas ofensivas y defensivas que voy realizando mientras me formo para trabajar en seguridad informática.

Todo lo que ves aquí está hecho en **mi propio laboratorio aislado**. No hay datos reales, ni sistemas de terceros, ni nada fuera de mi entorno controlado.

---

## 👤 Sobre mí

Estoy construyendo mi camino en ciberseguridad de forma práctica: monto mi propia infraestructura, rompo cosas en un entorno seguro, entiendo *por qué* fallan y aprendo a defenderlas. Me interesa tanto el lado ofensivo (pentesting) como el defensivo (análisis SOC).

- 📍 Boston, MA
- 🎓 En formación: ruta **Pre Security** en TryHackMe
- 🧪 Laboratorio propio montado en casa (ver abajo)

---

## 🧰 Mi laboratorio (home lab)

| Componente | Detalle |
|---|---|
| **Servidor de laboratorio** | Raspberry Pi 5 (4 GB RAM) |
| **Contenedores** | Docker |
| **Filtrado DNS / red** | Pi-hole |
| **Objetivo vulnerable** | DVWA (Damn Vulnerable Web Application) en Docker |
| **Red** | Switch administrable NETGEAR GS108E |
| **Estación de trabajo** | MacBook Pro (Apple Silicon) |
| **Herramientas** | nmap, sqlmap, hashcat, gobuster, whatweb, Burp Suite, Splunk (SIEM), Kali (VM) |

---

## 📓 Writeups

| # | Título | Categoría | Herramientas |
|---|--------|-----------|--------------|
| 01 | [SQL Injection en DVWA + crackeo de hashes](writeups/01-sql-injection-dvwa.md) | Web / Ofensivo | sqlmap, hashcat |
| 02 | [Command Injection en DVWA (Low/Medium/High)](writeups/02-command-injection-dvwa.md) | Web / Ofensivo | nmap, whatweb, gobuster |
| 03 | [Análisis de alertas SOC (falso positivo vs. verdadero positivo)](writeups/03-soc-analisis-alertas-tryhackme.md) | SOC / Defensivo | Splunk (SIEM) |

> Más writeups en camino conforme avanzo en la ruta.

---

## 🎯 Qué demuestran estas prácticas

- Montaje de infraestructura de laboratorio desde cero (hardware, red, contenedores)
- Explotación de vulnerabilidades web comunes (OWASP Top 10)
- Análisis e investigación de alertas de seguridad en un SIEM (triaje SOC)
- Uso de herramientas estándar de la industria
- Capacidad de **documentar** con claridad — una habilidad clave y a menudo olvidada en seguridad

---

## ⚖️ Nota ética

Todas las prácticas se realizan exclusivamente contra sistemas de mi propiedad, en una red aislada, con fines educativos. Nunca contra sistemas de terceros sin autorización explícita.
