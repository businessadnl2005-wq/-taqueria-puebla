# Taquería Puebla — despliegue en Netlify

## Estructura
```
taqueria-puebla/
├── index.html                  ← el sitio (frontend)
├── netlify/functions/chat.js   ← backend del chatbot (Netlify Function)
├── netlify.toml                 ← config: dónde están las funciones + redirect
└── .env.example                 ← plantilla de la variable de entorno
```

## Pasos para desplegar

1. **Sube esta carpeta completa a tu repo de GitHub** (ya lo tienes creado).
   Asegúrate de subir también `netlify.toml` y la carpeta `netlify/` — sin
   esos dos, el chat no va a funcionar aunque el sitio se vea bien.

2. **Entra a netlify.com y crea cuenta** (puedes usar tu mismo GitHub para
   registrarte, un clic).

3. **"Add new site" → "Import an existing project" → conecta GitHub → elige
   el repo `taqueria-puebla`.**
   - Netlify va a detectar `netlify.toml` solo. No necesitas tocar el build
     command ni el publish directory — ya están definidos en ese archivo.
   - Dale "Deploy site".

4. **Mete la API key (el paso que la gente olvida):**
   - En tu sitio dentro de Netlify: **Site configuration → Environment
     variables → Add a variable**.
   - Key: `ANTHROPIC_API_KEY`
   - Value: tu key real (la sacas de console.anthropic.com → API Keys)
   - Scopes: deja el default (todos).
   - Guarda, y ve a **Deploys → Trigger deploy → Deploy site** para que la
     variable nueva se aplique. (No se aplica sola con solo guardarla.)

5. **Prueba en el dominio real:**
   - Abre la URL que te dio Netlify (algo como `taqueria-puebla.netlify.app`).
   - Abre el chat, pregunta "¿a qué hora abren?". Si responde, ya está vivo.
   - Si dice "no está configurado todavía", revisa que el nombre de la
     variable sea exacto: `ANTHROPIC_API_KEY`, sin espacios ni typos.
   - Si el chat ni siquiera intenta responder (error de red inmediato),
     revisa en el navegador → pestaña Network → busca la llamada a
     `/api/chat` y mira qué código regresa. Un 404 casi siempre significa
     que `netlify.toml` no se subió o el redirect no cargó.

6. **Conecta tu dominio propio (opcional, cuando lo tengas):**
   - Site configuration → Domain management → Add a domain personalizado,
     sigue las instrucciones de DNS que te da Netlify.

## Checklist antes de entregarle esto a un cliente real

- [ ] Revisar y corregir menú, precios, horario, dirección y teléfono reales
      en **ambos** archivos: `index.html` (lo que ve el cliente) y
      `netlify/functions/chat.js` (lo que sabe el chatbot) — ahora mismo
      todo es de ejemplo
- [ ] Configurar `ANTHROPIC_API_KEY` como variable de entorno en Netlify —
      nunca en el código
- [ ] Confirmar que `.env.local` (si lo usas para pruebas locales) está en
      `.gitignore` y nunca se subió a GitHub
- [ ] Poner una alerta de gasto en console.anthropic.com → Billing
- [ ] Si vas a replicar este chatbot en más sitios de clientes, considera una
      API key separada por cliente — así un problema en un sitio no apaga el
      chatbot de todos los demás
- [ ] Probar el chat ya en el dominio real, no solo en el preview de Netlify

## Límite de uso incluido

`netlify/functions/chat.js` bloquea a una IP después de 15 mensajes por hora.
Es protección básica en memoria (no perfecta entre instancias, pero frena
abuso casual). Si el sitio recibe tráfico serio, la siguiente mejora es mover
ese límite a Upstash Redis (capa gratuita, exacto entre instancias):
https://upstash.com/docs/redis/sdks/ratelimit-ts/overview
