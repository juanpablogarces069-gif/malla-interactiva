# 🎓 Malla Interactiva — Ing. Mecánica + Industrial (PUC)

Planificador visual e interactivo de la malla curricular de la **Licenciatura en Ciencias
Naturales y Matemáticas con Major en Ingeniería Mecánica + Minor en Ingeniería Industrial**, y
su **Plan de Continuidad** hacia el título de *Ingeniero Civil de Industrias con Diploma en
Ingeniería Mecánica* (Pontificia Universidad Católica de Chile).

Permite arrastrar ramos entre semestres, marcar estados (por poner / planificado / en curso /
aprobado / reprobado), validar prerrequisitos, ver una cuenta regresiva para egresar y un
checklist de avance. Con cuenta opcional, guarda tu malla en la nube.

## ✨ Características

- Drag & drop de ramos entre semestres (máx. 60 cr por semestre).
- Validación de prerrequisitos (por posición en la malla).
- Estados por ramo con colores; reintento automático al reprobar.
- Renombrar ramos genéricos (AFG / OPI / Optativo Exploratorio).
- Semestres dinámicos (agregar/quitar).
- Panel de egreso: cuenta regresiva, gráficos de créditos y checklist.
- **Login con Google (Supabase Auth + OAuth)** y **guardado por usuario en la nube**, con
  `localStorage` como caché/respaldo offline.

## 🧱 Stack

HTML + CSS + JavaScript **vanilla**, en un **único archivo `index.html`**. Sin framework, sin
bundler, sin build. La única dependencia es **Supabase JS v2**, cargada por CDN.

## ▶️ Correr en local

El login necesita un origen `http://` (no funciona con `file://`). Sirve la carpeta:

```bash
python -m http.server 5500
```

y abre **http://localhost:5500**.
(Alternativas: `npx serve -l 5500`, o la extensión *Live Server* de VS Code.)

## ☁️ Configurar Supabase

1. Crea un proyecto en [supabase.com](https://supabase.com).
2. En **SQL Editor**, crea la tabla y las políticas RLS:

   ```sql
   create table public.malla_state (
     user_id    uuid primary key references auth.users(id) on delete cascade,
     state      jsonb not null default '{}'::jsonb,
     updated_at timestamptz not null default now()
   );

   alter table public.malla_state enable row level security;

   create policy "leer lo propio"       on public.malla_state
     for select using (auth.uid() = user_id);
   create policy "insertar lo propio"   on public.malla_state
     for insert with check (auth.uid() = user_id);
   create policy "actualizar lo propio" on public.malla_state
     for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
   create policy "borrar lo propio"     on public.malla_state
     for delete using (auth.uid() = user_id);
   ```

3. En **Project Settings → API**, copia la **Project URL** y la **anon / publishable key**, y
   pégalas en el bloque `CONFIG SUPABASE` al inicio del segundo `<script>` de `index.html`:

   ```js
   const SUPABASE_URL      = "https://TU-PROYECTO.supabase.co"; // solo el dominio, sin /rest/v1/
   const SUPABASE_ANON_KEY = "TU_ANON_KEY_PUBLICA";
   ```

4. En **Authentication → URL Configuration**, agrega tu(s) origen(es) en **Site URL** y
   **Redirect URLs**: `http://localhost:5500` y, al publicar, la URL de producción
   (ej. `https://tuusuario.github.io/malla-interactiva/`).

> **Sobre la anon/publishable key:** es segura de exponer en el frontend. Identifica al
> proyecto, no a un usuario, y el acceso a los datos lo controla **Row Level Security**: cada
> usuario solo puede leer/escribir su propia fila. La clave que **nunca** debe publicarse es la
> `service_role` (no se usa aquí).

## 🚀 Publicar (GitHub Pages)

1. Sube este repositorio a GitHub.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**, rama `main`,
   carpeta `/ (root)`.
3. La URL queda como `https://TU-USUARIO.github.io/malla-interactiva/`.
4. Agrega esa URL en Supabase → Authentication → URL Configuration (paso 4 de arriba).

## 📄 Estructura

```
malla-interactiva/
├── index.html     ← toda la app (HTML + CSS + JS)
├── README.md
└── .gitignore
```
