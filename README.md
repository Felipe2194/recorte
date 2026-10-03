# UTN CORRE — Procesador de fotos

Página estática (un solo `index.html`, sin backend) alojada en Vercel. Le pegás el link de una carpeta de Drive y:

1. Descarga cada foto, la redimensiona (opcional) y le pone el logo encima con `<canvas>`.
2. La guarda en el formato elegido (JPG / WebP / PNG, con calidad configurable).
3. La sube a una carpeta de resultados dentro de la carpeta de origen.
4. Arma el ZIP leyendo esa carpeta de resultados (en partes de 150 fotos si son muchas).

Todo el procesamiento corre en el navegador de quien usa la página; Vercel solo sirve el HTML. No hay Apps Script, ni `Code.gs`, ni servidor propio.

## Puesta en marcha (una sola vez)

### 1. Desplegar en Vercel

```bash
npm i -g vercel
vercel          # preview
vercel --prod   # producción
```

O subí la carpeta a GitHub e importala desde vercel.com/new (sin framework, sin build command). Anotá la URL de producción, ej. `https://utn-corre-fotos.vercel.app`.

### 2. Client ID de OAuth en Google Cloud Console

1. [console.cloud.google.com](https://console.cloud.google.com/) → proyecto nuevo.
2. "APIs y servicios" → "Biblioteca" → habilitar **Google Drive API**.
3. "Pantalla de consentimiento de OAuth" → tipo **Externo** → agregá como **usuarios de prueba** los correos que van a usar la herramienta (en modo Testing solo ellos pueden entrar).
4. "Credenciales" → "Crear credenciales" → "ID de cliente de OAuth" → **Aplicación web**.
5. En **Orígenes de JavaScript autorizados** agregá la URL de Vercel (sin barra final), ej. `https://utn-corre-fotos.vercel.app`. Si querés probar en local, agregá también `http://localhost:3000`.
6. Copiá el Client ID (`....apps.googleusercontent.com`).

### 3. Pegar el Client ID en la página

En `index.html`, completá la constante:

```js
const GOOGLE_CLIENT_ID = '1234567890-xxxx.apps.googleusercontent.com';
```

y volvé a desplegar. Con eso el campo del paso 1 desaparece y solo queda el botón "Conectar con Google". El Client ID no es un secreto: Google solo lo acepta desde los orígenes autorizados.

> Al conectar por primera vez aparece "Google no verificó esta app": "Avanzado" → "Ir a … (no seguro)". Es normal para una app en modo Testing que pide acceso a Drive.

## Uso

1. Conectar con Google.
2. Pegar el link (o ID) de la carpeta de Drive → "Buscar fotos".
3. Subir el logo PNG.
4. Configurar: tamaño / opacidad / margen del logo, formato de salida, calidad, lado más largo en px (0 = original), nombre de la carpeta de resultados. La **vista previa** muestra en vivo cómo queda cualquier foto de la carpeta (con ‹ › se recorren) y cuánto pesará cada resultado.
5. "Procesar fotos". Se puede detener y retomar: las fotos que ya tienen resultado en Drive (mismo nombre y extensión) se saltean.
6. "Generar y descargar ZIP" (funciona también otro día sin reprocesar: busca la carpeta de resultados por nombre).

Las fotos originales nunca se tocan.

## Notas técnicas

- **Alcance OAuth `drive` (completo)**: necesario para listar una carpeta pegando su link. La alternativa más acotada (`drive.file` + Google Picker) obliga a elegir archivos con el selector de Google.
- **Concurrencia**: 3 fotos a la vez (`CONCURRENCY` en `index.html`). Si la máquina se queda sin memoria con fotos muy grandes, bajala a 1.
- **Token**: dura ~1 h. Si un lote largo falla con `token_expired`, volvé a "Conectar" y usá "Reintentar fallidas".
- **Formatos**: WebP se genera bien en Chrome/Edge/Firefox; Safari puede no soportarlo para exportar (la foto queda como fallida con el motivo).
