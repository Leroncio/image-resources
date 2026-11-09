# Repositorio de imágenes temporal

Videos de previsualización (preview) que se muestran dentro de los mockups de dispositivos (laptop, tablet y teléfono). jsDelivr los sirve directo desde este repo:

```
https://cdn.jsdelivr.net/gh/Leroncio/image-resources@master/preview/<prefijo>_<id>.<ext>
```

`index.html` es una galería que lista todo lo que hay en `preview/` y en la raíz, para revisar y copiar los enlaces.

---

## Cómo carga los videos la página de preview

La página recibe el cliente en el parámetro `serv`:

```
https://<sitio-preview>/?serv=clinicaintegralarauco
```

El script:

1. Toma el valor de `serv` y lo convierte a **base64url** (`btoa`, luego `+` → `-`, `/` → `_` y quita los `=` del final). Ese resultado es el `<id>`.
2. A cada dispositivo de la página le asigna su archivo, con el prefijo según el tipo:
   - `.device.laptop` → `dk_<id>`
   - `.device.tablet` → `tb_<id>`
   - `.device.phone` → `sp_<id>`
3. Intenta cargar `.webm`. Si el navegador no soporta WebM o el archivo da error, prueba `.mp4`. Si eso también falla, muestra el placeholder (`.no-preview`).

Si `serv` viene vacío o no se puede codificar, se muestra el placeholder directamente.

---

## Nombres de archivo (base64url)

Cada cliente lleva **3 archivos** en `preview/`:

| Dispositivo | Prefijo | Ejemplo (`serv=clinicaintegralarauco`)        |
|-------------|---------|-----------------------------------------------|
| Laptop      | `dk_`   | `dk_Y2xpbmljYWludGVncmFsYXJhdWNv.webm`        |
| Tablet      | `tb_`   | `tb_Y2xpbmljYWludGVncmFsYXJhdWNv.webm`        |
| Teléfono    | `sp_`   | `sp_Y2xpbmljYWludGVncmFsYXJhdWNv.webm`        |

### Generar el id

**Git Bash / Linux / macOS:**

```bash
printf '%s' 'clinicaintegralarauco' | base64 | tr '+/' '-_' | tr -d '=\n'
```

**PowerShell:**

```powershell
[Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes('clinicaintegralarauco')).Replace('+','-').Replace('/','_').TrimEnd('=')
```

**Consola del navegador (F12), que es lo mismo que hace la página:**

```js
btoa('clinicaintegralarauco').replace(/\+/g,'-').replace(/\//g,'_').replace(/=+$/,'')
```

Usa `printf '%s'` y no `echo`, porque `echo` agrega un salto de línea que cambia el resultado.

### Reglas para el valor de `serv`

- **Solo minúsculas a-z y números, sin espacios, tildes ni ñ.** `btoa` en el navegador codifica en Latin‑1 y `base64` en la terminal en UTF‑8, así que con tildes o ñ los ids no coinciden. Además, `btoa` falla con caracteres fuera de Latin‑1.
- **Distingue mayúsculas y minúsculas:** `ClinicaX` y `clinicax` dan ids distintos.
- Sin `=` al final y con `-`/`_` en vez de `+`/`/` (base64url). Con nombres en minúsculas ASCII casi nunca aparecen `+` o `/`, pero el `=` de relleno sí aparece seguido y hay que quitarlo.

Ejemplos existentes:

| `serv`                  | id                                |
|-------------------------|-----------------------------------|
| `clinicaintegralarauco` | `Y2xpbmljYWludGVncmFsYXJhdWNv`    |
| `funerariasantarosa`    | `ZnVuZXJhcmlhc2FudGFyb3Nh`        |
| `funexpert`             | `ZnVuZXhwZXJ0`                    |
| `motelriviera`          | `bW90ZWxyaXZpZXJh`                |
| `montecordillera`       | `bW9udGVjb3JkaWxsZXJh`            |

---

## Dimensiones por dispositivo

Graba cada video al tamaño de la pantalla del mockup correspondiente. Estas son las medidas que usan los previews actuales:

| Dispositivo | Prefijo | Ancho × Alto (px) | Orientación |
|-------------|---------|-------------------|-------------|
| Laptop      | `dk_`   | **1198 × 764**    | Horizontal  |
| Tablet      | `tb_`   | **456 × 606**     | Vertical    |
| Teléfono    | `sp_`   | **356 × 768**     | Vertical    |

Una forma práctica de grabar es usar el modo responsive de DevTools (Ctrl+Shift+M) con esas medidas exactas y capturar solo el viewport.

### Formato recomendado

- **Contenedor/códec:** WebM con VP9 (`yuv420p`). Opcionalmente, también un `.mp4` H.264 con el mismo nombre como respaldo para navegadores sin WebM.
- **Duración:** unos 20–30 s (el video se reproduce en loop).
- **FPS:** 15–30.
- **Audio:** no hace falta. Se recomienda quitarlo para que el archivo pese menos.
- **Peso:** idealmente menos de 5 MB por archivo.

Convertir o redimensionar con ffmpeg (ejemplo para laptop):

```bash
ffmpeg -i grabacion.mp4 -vf "scale=1198:764,fps=30" -c:v libvpx-vp9 -b:v 0 -crf 34 -pix_fmt yuv420p -an dk_<id>.webm
```

Respaldo MP4 opcional:

```bash
ffmpeg -i grabacion.mp4 -vf "scale=1198:764,fps=30" -c:v libx264 -crf 26 -pix_fmt yuv420p -movflags +faststart -an dk_<id>.mp4
```

---

## Pasos para subir un cliente nuevo

1. Genera el id base64url del valor de `serv`.
2. Graba o exporta los 3 videos con las dimensiones de la tabla.
3. Guárdalos en `preview/` como `dk_<id>.webm`, `tb_<id>.webm` y `sp_<id>.webm`.
4. Haz commit y push a `master`.
5. Prueba con `?serv=<valor>` en la página de preview.

## Reemplazar un video existente (caché de jsDelivr)

jsDelivr guarda en caché las URLs que apuntan a `@master` (puede durar horas o días). Si reemplazas un archivo **con el mismo nombre**, el CDN puede seguir entregando la versión anterior. Después del push, purga cada archivo:

```bash
for p in dk tb sp; do curl -s "https://purge.jsdelivr.net/gh/Leroncio/image-resources@master/preview/${p}_<id>.webm"; done
```

Luego recarga el navegador con Ctrl+F5. Para confirmar que el CDN ya tiene la versión nueva, compara el `Content-Length` con el tamaño del archivo local:

```bash
curl -sI "https://cdn.jsdelivr.net/gh/Leroncio/image-resources@master/preview/dk_<id>.webm" | grep -i content-length
```
