# Cómo publicar tu quiz en internet (Supabase + Vercel)

Tu quiz está en la carpeta **`quiz-web`**:

- `index.html`: la página.
- `config.js`: los datos de tu Supabase (los llenas en el paso 4).
- `seed.json`: tus 10 preguntas y fotos, para cargarlas con un toque.

Supabase y Vercel son **gratis y no piden tarjeta**.

---

## Paso 1: Crear tu proyecto en Supabase

1. Entra a **https://supabase.com** y toca **Start your project**.
2. Entra con **GitHub**, la misma cuenta que usas en Vercel.
3. Toca **New project**:
   - **Name:** `quiz-lore`
   - **Database Password:** toca **Generate a password** y guárdala en algún lado. Casi nunca la vas a necesitar.
   - **Region:** la más cercana, por ejemplo *South America (São Paulo)* o *East US*.
   - **Plan:** Free.
4. Toca **Create new project** y espera 1 o 2 minutos mientras se prepara.

## Paso 2: Crear la base de datos (copiar y pegar)

1. En el menú de la izquierda entra a **SQL Editor**.
2. Toca **New query**.
3. Pega todo este código.
4. Tu correo ya está puesto en el código (lorenavfernandezr@gmail.com).
5. Toca **Run**. Debe decir *Success. No rows returned*.

```sql
-- Tablas: cada fila es un documento {id, data}
create table if not exists quiz    (id text primary key, data jsonb not null);
create table if not exists images  (id text primary key, data jsonb not null);
create table if not exists results (id text primary key, data jsonb not null);
create table if not exists wishes  (id text primary key, data jsonb not null);

alter table quiz    enable row level security;
alter table images  enable row level security;
alter table results enable row level security;
alter table wishes  enable row level security;

-- Quién es la dueña del quiz
create or replace function public.is_owner() returns boolean
language sql stable as $$
  select coalesce(lower(auth.jwt() ->> 'email'), '') = lower('lorenavfernandezr@gmail.com')
$$;

-- Borra reglas anteriores (así este código se puede ejecutar varias veces)
drop policy if exists "todos leen"     on quiz;
drop policy if exists "solo dueña"     on quiz;
drop policy if exists "todos leen"     on images;
drop policy if exists "solo dueña"     on images;
drop policy if exists "todos leen"     on results;
drop policy if exists "guardar el mío" on results;
drop policy if exists "dueña borra"    on results;
drop policy if exists "dueña lee"      on wishes;
drop policy if exists "guardar el mío" on wishes;
drop policy if exists "dueña borra"    on wishes;

-- Preguntas y fotos: todos las leen, solo tú las cambias
create policy "todos leen"  on quiz   for select using (true);
create policy "solo dueña"  on quiz   for all using (is_owner()) with check (is_owner());
create policy "todos leen"  on images for select using (true);
create policy "solo dueña"  on images for all using (is_owner()) with check (is_owner());

-- Resultados: todos ven el ranking, cada quien guarda solo el suyo y una sola vez
create policy "todos leen"       on results for select using (true);
create policy "guardar el mío"   on results for insert
  with check (id = auth.uid()::text and length(data->>'name') between 1 and 40);
create policy "dueña borra"      on results for delete using (is_owner());

-- Deseos: cada quien deja el suyo, solo tú los lees
create policy "dueña lee"        on wishes for select using (is_owner());
create policy "guardar el mío"   on wishes for insert
  with check (id = auth.uid()::text and length(coalesce(data->>'wish','')) <= 600);
create policy "dueña borra"      on wishes for delete using (is_owner());

-- Actualización en vivo del ranking
do $$
declare t text;
begin
  foreach t in array array['quiz','images','results','wishes'] loop
    if not exists (select 1 from pg_publication_tables
                   where pubname = 'supabase_realtime' and schemaname = 'public' and tablename = t) then
      execute format('alter publication supabase_realtime add table public.%I', t);
    end if;
  end loop;
end $$;
```

## Paso 3: Activar el inicio de sesión

**Para tus amigos (sin cuenta):**
1. Entra a **Authentication → Sign In / Providers**. En algunas versiones está en *Authentication → Settings*.
2. Activa **Allow anonymous sign-ins** y guarda.

**Para ti (tu panel):**
1. Entra a **Authentication → Users → Add user → Create new user**.
2. Escribe **tu correo** (el mismo del SQL) y una **contraseña** que recuerdes.
3. Marca **Auto Confirm User** y toca **Create user**.

**Para que puedan jugar muchos al mismo tiempo,** por ejemplo en tu fiesta con el mismo wifi:
- En **Authentication → Rate Limits**, sube *"Rate limit for anonymous users"* a **300** y guarda.

## Paso 4: Conectar la página con Supabase

1. Entra a **Project Settings** (el engranaje ⚙️) **→ API**. En algunas versiones se llama *Data API* o *API Keys*.
2. Copia:
   - **Project URL**, que se ve como `https://abcdxyz.supabase.co`.
   - La clave **anon public**, un texto largo que empieza con `eyJ...`.
3. Abre **`quiz-web/config.js`** y pega cada valor donde dice `PEGA_AQUI...`.
4. Tu correo ya está puesto. Guarda el archivo.

La clave *anon public* no es secreta y puede ir en la página. Lo que protege tu quiz son las reglas del paso 2. **Nunca uses la clave `service_role`.**

## Paso 5: Subir los cambios y publicar en Vercel

En la terminal:

```bash
cd ~/Desktop/lore
git add .
git commit -m "Conectar Supabase"
git push
```

**En Vercel (solo la primera vez):**
1. Entra a tu proyecto **lore → Settings → Build and Deployment**.
2. En **Root Directory** toca **Edit**, elige **`quiz-web`** y guarda.
3. Ve a **Deployments**, toca los tres puntos **⋯** del último deploy y elige **Redeploy**.

Tu link: **https://lore-eta-nine.vercel.app**

## Paso 6: Cargar tus preguntas

1. Abre tu link.
2. Abajo de todo toca **"Soy Lore · entrar a mi panel"**, escribe tu correo y contraseña, y toca **Entrar**.
3. Arriba te aparece **"Cargar mis preguntas y fotos"**. Tócalo y tus 10 preguntas con fotos quedan cargadas.
4. Cambia a **"Vista de amigos"** y juega una vez para probar. Después borra tu resultado con la **×** del ranking.
5. **Avísale a Claude para borrar `seed.json`** del repositorio, así nadie puede ver las respuestas en ese archivo.

## Paso 7: Compartir 🎉

Manda el link a tus amigos. Ellos solo lo abren y juegan.

---

### Bueno saber

- **Supabase pausa los proyectos gratis después de 7 días sin uso.** Si pasa, la página no carga las preguntas. Entra a supabase.com, abre tu proyecto y toca **Restore**. Tus datos no se pierden. Mientras tus amigos estén jugando, no se pausa.
- **Un intento por persona:** cada dispositivo puede jugar una vez. Si alguien borra los datos de su navegador o juega desde otro celular, podría jugar de nuevo. Si ves algo raro, borra ese resultado con la ×.
- **Tus amigos no ven** las respuestas correctas, los deseos ni tu panel.
- **Si cambias el diseño o el código,** súbelo con `git add .`, `git commit -m "..."` y `git push`, y Vercel se actualiza solo. Las preguntas que cambies desde tu panel no necesitan esto.
