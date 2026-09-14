# Recetario — app web

App de recetas propias: añadir, editar, archivar y quitar, con lectura rápida de nutrientes
(alto / medio / bajo) y filtros combinables por momento, elaboración y nutriente.

Archivos:

- `index.html` — la app completa en **un solo archivo**. Es lo que se publica.
- `manifest.json` — permite instalarla en el móvil como app.
- `Recetario.dc.html` — el fuente editable del diseño. Si lo cambias, hay que regenerar `index.html`.

## Dos modos de guardado

1. **Local (por defecto).** Las recetas viven en el almacenamiento del navegador
   (`localStorage`, clave `recetario.v1`). Funciona sin conexión y sin cuentas, pero solo en ese
   dispositivo.
2. **Nube (Supabase).** En cuanto pegas las dos claves de tu proyecto, la app pide entrar con tu
   correo y a partir de ahí cada cambio se guarda en la base de datos **y** en el dispositivo (la
   copia local queda como caché para cocinar sin cobertura).

En los dos modos tienes al final de la lista **Descargar copia** e **Importar copia** (JSON).

---

## Paso 1 — Publicarla en GitHub Pages

1. Crea un repositorio nuevo en github.com: **New**, nombre `recetario`, visibilidad **Public**.
2. **Add file → Upload files** y arrastra `index.html`, `manifest.json` y este `README.md`.
   **Commit changes**.
3. **Settings → Pages**. En *Source* elige **Deploy from a branch**; en *Branch*, `main` y
   carpeta `/ (root)`. **Save**.
4. Espera 1–2 minutos: aparecerá `https://TU-USUARIO.github.io/recetario/`.
5. Ábrela en el móvil → menú del navegador → **Añadir a la pantalla de inicio**. Se abre a pantalla
   completa, como una app.

Para actualizarla más adelante: sube otra vez `index.html` y confirma el reemplazo.

## Paso 2 — Conectar Supabase (base de datos de verdad)

1. Entra en supabase.com, **New project**. Apunta la región más cercana y la contraseña que te pida
   (no la necesitarás para esto, pero guárdala).
2. Abre **SQL Editor → New query**, pega esto y pulsa **Run**:

   ```sql
   create table recetas (
     id text primary key,
     user_id uuid not null default auth.uid() references auth.users on delete cascade,
     datos jsonb not null,
     actualizado timestamptz not null default now()
   );

   alter table recetas enable row level security;

   create policy "cada quien ve lo suyo" on recetas
     for all
     using (auth.uid() = user_id)
     with check (auth.uid() = user_id);
   ```

   Esto crea la tabla y la regla de seguridad: cada cuenta solo puede leer y escribir sus propias
   recetas.

3. Ve a **Authentication → URL Configuration** y en *Site URL* pon la dirección de tu GitHub Pages
   (`https://TU-USUARIO.github.io/recetario/`). Añádela también en *Redirect URLs*. Sin esto, el
   enlace del correo no vuelve a la app.
4. Ve a **Project Settings → API** y copia dos cosas: **Project URL** y la clave **anon public**
   (la anon es pública por diseño; la seguridad la pone la regla del paso 2 — no uses nunca la
   `service_role`).
5. Abre `Recetario.dc.html`, busca `static NUBE = {` (está al principio del bloque de lógica) y
   pega los valores:

   ```js
   static NUBE = {
     url: 'https://xxxxxxxx.supabase.co',
     anonKey: 'eyJhbGciOi…',
   };
   ```

   Dime los valores y lo pego yo, y te devuelvo el `index.html` ya compilado.
6. Vuelve a generar `index.html` y súbelo a GitHub (paso 1.5). Al abrir la app te pedirá el correo,
   recibirás un enlace, y al pulsarlo entras. La primera vez sube automáticamente las recetas que
   tuvieras en ese dispositivo.
7. En el ordenador entra con el mismo correo: verás las mismas recetas.

### Si quieres guardar las fotos mejor

Ahora las fotos viajan dentro del JSON de cada receta (reescaladas a 1100 px). Funciona, pero si
acumulas muchas conviene **Supabase Storage**: creas un bucket `fotos`, subes el archivo y guardas
solo la URL en la receta. Es el siguiente paso natural cuando empieces a notarlo lento.
