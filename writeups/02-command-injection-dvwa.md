# Command Injection — DVWA

**Autor:** deividcuello9-wq
**Entorno:** Home lab — Raspberry Pi 5 (DVWA en Docker) atacada desde Kali Linux (VM en UTM)
**Objetivo:** `TARGET_IP:8080` (DVWA)
**Fecha:** Noviembre 2026

---

## Resumen

La función "Ping a device" de DVWA toma una IP del usuario y la pasa directamente al comando `ping` del sistema operativo. Como la entrada no se valida correctamente, es posible encadenar comandos propios y lograr **ejecución remota de comandos (RCE)** en el servidor.

Se explotó la vulnerabilidad en los tres niveles de seguridad (Low, Medium, High), demostrando que cada filtro basado en lista negra puede evadirse.

---

## Reconocimiento previo

Antes de atacar la aplicación se hizo reconocimiento desde Kali:

```
sudo nmap -sV TARGET_IP
```

Puertos abiertos relevantes:

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22 | SSH | OpenSSH 10.0 |
| 53 | DNS | dnsmasq 2.93 (Pi-hole) |
| 8080 | HTTP | Apache 2.4.68 (DVWA) |

Identificación de la aplicación web:

```
whatweb http://TARGET_IP:8080
```

Resultado: **DVWA**, PHP 8.5.11, Apache 2.4.68 sobre Debian. Redirige a `login.php`.

---

## Explotación

### Nivel LOW

Sin ningún filtro. El separador `;` ejecuta un segundo comando tras el ping.

**Payload:**
```
TARGET_IP; whoami
```

**Resultado:** `www-data`

Comandos adicionales ejecutados para medir el alcance:

```
TARGET_IP; id          -> uid=33(www-data) gid=33(www-data) groups=33(www-data)
TARGET_IP; uname -a    -> Linux 142a48145d73 6.18.50 ... Debian aarch64
TARGET_IP; cat /etc/passwd  -> lista completa de usuarios del sistema
```

**Observación clave:** el hostname `142a48145d73` (formato de ID aleatorio) confirma que DVWA corre dentro de un **contenedor Docker**, aislado de la Raspberry Pi anfitriona.

---

### Nivel MEDIUM

El código filtra `;` y `&&` mediante una lista negra. Sin embargo, no bloquea el pipe `|`.

**Payload fallido:** `TARGET_IP; whoami` (el `;` se elimina)

**Payload exitoso:**
```
TARGET_IP| whoami
```

**Resultado:** `www-data`

**Por qué funciona:** la lista negra solo contempla `;` y `&&`. El pipe `|` no está en la lista, así que pasa sin filtrar.

---

### Nivel HIGH

Lista negra más amplia: bloquea `;`, `&`, `&&`, `||` e incluso `| ` (pipe seguido de espacio). Error: solo se bloquea el pipe **con** espacio.

**Payload exitoso:**
```
TARGET_IP|whoami
```
(pipe pegado al comando, sin espacio)

**Resultado:** `www-data`

**Por qué funciona:** el filtro elimina la cadena `'| '` (con espacio final). Un pipe sin espacio después no coincide con el patrón y se ejecuta.

---

## Lección y mitigación

El patrón se repite en los tres niveles: **las listas negras son incompletas por naturaleza**. El desarrollador bloquea los caracteres que recuerda, pero siempre queda una variante sin cubrir (`|`, `|` sin espacio, etc.).

**Defensa correcta (lista blanca):** validar que la entrada tenga exactamente forma de dirección IP y rechazar todo lo demás, por ejemplo con una expresión regular que solo permita dígitos y puntos, o usando funciones del lenguaje que separen argumentos sin invocar una shell.

| Nivel | Filtro | Evasión |
|-------|--------|---------|
| Low | ninguno | `;` |
| Medium | `;` y `&&` | `\|` |
| High | casi todos los separadores | `\|` sin espacio |

---

## Impacto

Con un usuario `www-data` y RCE confirmado, un atacante real podría leer archivos de la aplicación, buscar credenciales, enumerar el sistema e intentar escalar privilegios. En este caso el aislamiento por Docker limita el daño al contenedor, lo que ilustra por qué los entornos vulnerables se montan en contenedores.
