# legal/

Páginas legales públicas de Alma para `https://alma.mercurium.ai/...`.

## Estructura final del sitio

Cuando subas estos archivos, el sitio servirá:

| Archivo local | URL pública | Descripción |
|---|---|---|
| `index.html` | `/` | Landing con enlaces |
| `privacy/index.html` | `/privacy` | Política de privacidad |
| `terms/index.html` | `/terms` | Términos y condiciones |
| `eula/index.html` | `/eula` | EULA |
| `_assets/styles.css` | `/_assets/styles.css` | Estilos compartidos |

> **Importante:** las URLs ya **NO** llevan `/legal/` por delante. El subdominio `alma.mercurium.ai` ya está dedicado a esto, así que añadir `/legal/` era redundante. Las rutas finales son: `/privacy`, `/terms`, `/eula`. Esto está sincronizado con `app.json → expo.extra.legal`.

## Cómo desplegar (Cloudflare Workers Static Assets — lo que estás usando)

Si en Cloudflare elegiste **"Upload assets" → Worker** (URL `*.workers.dev`), funciona perfectamente para sitios estáticos. Los pasos correctos:

### 1. Sube SOLO el contenido de `legal/`, no la carpeta

Cuando arrastras la carpeta `legal/` completa al uploader, Cloudflare descarta el nivel superior. Los archivos quedan en la raíz del Worker:

```
/                   → index.html
/_assets/styles.css
/privacy/index.html
/terms/index.html
/eula/index.html
```

Esto es **exactamente** lo que queremos con la nueva estructura. No hace falta cambiar nada — solo asegúrate de que en la lista de archivos veas `index.html`, `_assets/`, `privacy/`, `terms/`, `eula/` directamente al primer nivel (sin `legal/` por encima).

### 2. Verifica el deploy temporal

Cloudflare te da una URL tipo `<algo>.polbuisan.workers.dev`. Comprueba:

- `https://<algo>.polbuisan.workers.dev/` → landing estilizado.
- `https://<algo>.polbuisan.workers.dev/privacy` → política de privacidad estilizada.
- `https://<algo>.polbuisan.workers.dev/terms` → términos.
- `https://<algo>.polbuisan.workers.dev/eula` → EULA.

Las 4 deben cargar con CSS y los enlaces internos del header/footer deben funcionar.

### 3. Configurar dominio custom `alma.mercurium.ai`

En el panel del Worker:

1. Pestaña **Settings** → sección **Domains & Routes** → **Add** → **Custom Domain**.
2. Escribe `alma.mercurium.ai` → **Add Custom Domain**.
3. Cloudflare comprueba si tu dominio `mercurium.ai` está gestionado por Cloudflare DNS:
   - **Si SÍ** (lo más probable si lo compraste o lo migraste a Cloudflare): te crea el CNAME automáticamente. Activación en 1-2 min.
   - **Si NO** (lo tienes en GoDaddy/IONOS/Namecheap): te muestra el destino CNAME (`<algo>.polbuisan.workers.dev`). Ve al DNS de tu registrador:
     - Tipo: `CNAME`
     - Nombre: `alma`
     - Valor: el que te indique Cloudflare
     - TTL: Auto / 3600
4. Espera 1-5 min. Cloudflare provisiona SSL automáticamente.
5. Verifica los URLs finales:
   - <https://alma.mercurium.ai/privacy>
   - <https://alma.mercurium.ai/terms>
   - <https://alma.mercurium.ai/eula>

¡Listo!

## Alternativas (si Cloudflare se complica)

### Netlify Drop (lo más rápido — 30 segundos)

1. Ve a <https://app.netlify.com/drop>.
2. Arrastra **el contenido** de `legal/` (selecciona todo dentro de `legal/` y arrástralo, no la carpeta).
3. Te da la URL al instante.
4. Settings → Domain → Add custom domain → `alma.mercurium.ai`.
5. Te indica los DNS records que añadir.

### Vercel

```bash
cd alma/legal
npx vercel --prod
```

Luego asigna `alma.mercurium.ai` desde el dashboard.

## Verificación final (CRÍTICO antes de submit a Apple)

Apple va a comprobar que estos 3 URLs están vivos y funcionan. Confírmalo manualmente:

- ✅ <https://alma.mercurium.ai/privacy>
- ✅ <https://alma.mercurium.ai/terms>
- ✅ <https://alma.mercurium.ai/eula>

Estos URLs son los que están en:
- `app.json → expo.extra.legal.{privacyUrl,termsUrl,eulaUrl}`
- Visibles en `app/(auth)/welcome.tsx` y `app/(auth)/paywall.tsx` (vía `getLegalConfig()`).
- Que se ponen en App Store Connect como Privacy Policy URL y EULA URL en la review.

## Cosas que MUY recomendablemente revises antes de publicar

- [ ] **Razón social exacta** de Mercurium si está constituida como SL/SAS/etc. (ahora pone solo "Mercurium").
- [ ] **NIF/CIF y domicilio fiscal:** la AEPD recomienda incluirlos si la empresa está constituida.
- [ ] **Jurisdicción:** los Términos asumen "Juzgados de Barcelona". Cambia si tu domicilio fiscal está en otra ciudad.
- [ ] **Edad mínima:** está fijada en 16 años (límite GDPR-ES). No bajar.

Si me dices la razón social y la ciudad, te lo sustituyo en los 3 documentos de golpe.
