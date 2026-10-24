# 📤 Cómo subir este portafolio a GitHub

Guía rápida. Hay dos formas: por la **web** (la más fácil, sin comandos) o por la **terminal** (más profesional, se nota en un portafolio de cyber). Empieza por la web si quieres algo rápido.

---

## Paso 1 — Crear tu cuenta de GitHub (una sola vez)

1. Entra a **https://github.com/signup**
2. Usa tu correo, elige un usuario y contraseña. Es gratis.
3. Elige un **nombre de usuario profesional** — aparecerá en la URL de tu portafolio (ej. `github.com/deivid-cyber`). Que sea serio, porque lo verán reclutadores.

> 💡 Tu usuario es parte de tu marca. Piénsalo 30 segundos.

---

## Paso 2 — Crear el repositorio

1. Arriba a la derecha, clic en **＋ → New repository**
2. **Repository name:** `ciberseguridad-notas` (o `homelab-writeups`)
3. **Description:** `Writeups y notas de mi laboratorio de ciberseguridad`
4. Déjalo en **Public** (quieres que lo vean).
5. Marca ✅ **Add a README file** (lo reemplazaremos enseguida).
6. Clic en **Create repository**.

---

## OPCIÓN A — Subir por la web (sin comandos)

La más sencilla para empezar.

1. En tu repo, clic en **Add file → Upload files**.
2. Arrastra estos archivos y carpetas:
   - `README.md`
   - la carpeta `writeups/` completa
3. Abajo, en "Commit changes", escribe un mensaje como `Primer writeup: SQL injection` y clic en **Commit changes**.
4. Para las capturas: entra a la carpeta `writeups/img/`, **Add file → Upload files**, y sube tus imágenes `.png`.

✅ Listo. Tu portafolio ya está público.

---

## OPCIÓN B — Subir por terminal (recomendado para cyber)

Manejar `git` por línea de comandos es parte del oficio, y se ve mejor. En tu MacBook, abre **Terminal** y, dentro de la carpeta del portafolio:

```bash
# 1. Inicializar git en la carpeta
git init

# 2. Añadir todos los archivos
git add .

# 3. Primer commit
git commit -m "Primer writeup: SQL injection en DVWA"

# 4. Conectar con tu repo (cambia la URL por la tuya)
git remote add origin https://github.com/TU-USUARIO/ciberseguridad-notas.git

# 5. Subir
git branch -M main
git push -u origin main
```

> La primera vez, GitHub te pedirá autenticarte. Usa un **Personal Access Token** en lugar de contraseña:
> GitHub → tu foto → **Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**. Marca el permiso `repo`. Copia el token y pégalo cuando la terminal te pida la contraseña.

---

## Paso 3 — Añadir las capturas de pantalla

El writeup 01 tiene marcadas, con `📸`, las partes donde va una captura. Para cada una:
1. Toma la captura mientras repites el ejercicio en tu lab.
2. Guárdala en `writeups/img/` con un nombre claro (ej. `01-formulario.png`).
3. En el writeup, descomenta/añade la línea: `![descripción](img/01-formulario.png)`

Las capturas son lo que convierte un writeup de "creíble" a "contundente". Un reclutador ve de un vistazo que *tú* lo hiciste.

---

## ¿Y ahora qué?

- Rellena los `<IP-DE-MI-PI>` y demás marcadores del writeup 01 con tus valores reales.
- Usa `writeups/_plantilla.md` para cada reto nuevo que hagas (copia, renombra a `02-...`, rellena).
- Cada writeup nuevo = una entrada más en la tabla del `README.md`.
