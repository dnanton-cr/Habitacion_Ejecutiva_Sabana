# Habitación Ejecutiva Sabana Norte

Sitio de reserva directa (una sola página HTML) con Firebase Firestore.

## Galería de fotos con carga desde el teléfono

La sección **Fotos y video** ya no tiene imágenes fijas: se alimenta de la
colección `galeria` de Firestore. Desde el panel de administración puedes subir
fotos tomadas o elegidas desde la galería del teléfono (Android / iPhone).

### Cómo entrar al panel

1. Abre el sitio y agrega `#panel` al final de la URL:
   `https://habitacionejecutivasabana.vercel.app/#panel`
   (también hay un enlace discreto «Administración de fotos» en el pie).
2. Escribe el PIN. **La primera vez**, el PIN que escribas queda registrado como
   el PIN del panel (se guarda solo su hash en `config/panel`).
3. Ya dentro puedes:
   - **Elegir fotos**: selección múltiple desde la galería del teléfono.
   - **Tomar foto**: abre la camara directamente.
   - Editar la descripción de cada foto (se muestra sobre la imagen).
   - Reordenar con flechas arriba/abajo. **La primera foto es la portada**.
   - Ocultar / mostrar una foto sin borrarla.
   - Borrar.

Las fotos se reducen en el propio teléfono (máx. 1600 px, JPEG) antes de
subirse, así que no consumen datos de más ni exigen pantalla de escritorio.

### Cómo se guarda

| Colección | Contenido |
|---|---|
| `galeria` | miniatura (520 px), descripción, orden, visibilidad |
| `galeria_full` | imagen optimizada (1600 px) que se carga solo al ampliar |

Se usa Firestore y no Firebase Storage a propósito: Storage en proyectos nuevos
exige plan Blaze (tarjeta). Con Firestore la galería funciona en el plan
gratuito. Límite por documento: 1 MiB, por eso la imagen se comprime a ~500 KB.

## Paso obligatorio: reglas de Firestore

El proyecto tenía las reglas en modo cerrado (`allow read, write: if false`),
lo que bloquea **también** el formulario de reserva y la galería. Hay que
publicar el archivo `firestore.rules`:

1. Firebase Console -> Firestore Database -> pestaña **Reglas**.
2. Borra el contenido y pega el de `firestore.rules`.
3. **Publicar**.

> El PIN del panel es una barrera de interfaz, no seguridad real: en esta fase
> las reglas permiten escribir a cualquiera que conozca la URL. La restriccion
> por usuario llega en la fase 2 (bloque comentado al final del archivo de
> reglas).

## Fase 2 - Módulo de seguridad (pendiente)

1. Activar **Authentication** (correo/contraseña o Google).
2. Crear la colección `usuarios/{uid}` con el campo `rol`:
   `admin` (todo), `mantenimiento` (subir/editar fotos), `lectura` (solo ver).
3. Descomentar el bloque FASE 2 de `firestore.rules` y publicar.
4. En el panel, reemplazar el PIN por `signInWithEmailAndPassword` y mostrar el
   panel solo si el rol lo permite.

## Otros cambios

- Se corrigió el bloque de Firebase: llamaba `getAnalytics` sin importarlo y
  ejecutaba `initializeApp` dos veces, lo que rompía el envío del formulario de
  reserva.
- El formulario mantiene el respaldo por WhatsApp si Firestore falla.

## Publicar

```bash
git add .
git commit -m "Galeria con subida de fotos desde el telefono"
git push
```

Vercel despliega automáticamente desde la rama `main`.
