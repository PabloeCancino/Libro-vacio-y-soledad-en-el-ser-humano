# Guía: publicar un libro en GitHub + GitHub Pages (mdBook)

Documenta el proceso completo que se siguió para publicar *Vacío y Soledad en el Ser Humano*, para reutilizarlo tal cual en los próximos títulos. Son dos partes independientes: **(1)** subir el manuscrito a un repositorio de GitHub y **(2)** desplegarlo como sitio navegable con mdBook. La parte 1 se puede hacer sola (repo con los `.md` en bruto); la parte 2 requiere que la parte 1 ya exista.

---

## Parte 1 — Repositorio en GitHub

### 1.1 Estructura mínima antes del primer commit

```
vacio-y-soledad-en-el-ser-humano/
├── capitulos/          (o src/: manuscrito en Markdown)
├── LICENSE.md
├── README.md
└── .gitignore          ← créalo ANTES del primer `git add`
```

### 1.2 Seguridad primero: `.gitignore`

Si el proyecto usa algún script, contenedor Docker o pipeline de publicación automática, casi siempre habrá un `.env` con credenciales (tokens, claves). **Ese archivo nunca debe llegar al repo**, y GitHub no perdona: una vez pusheado un secreto a un repo público, se considera comprometido aunque lo borres después (queda en el historial). Crea `.gitignore` desde el primer commit:

```
# Secretos y configuración local — nunca deben subirse al repo
.env
.env.*

# Salida de compilación de mdBook
/book/

# Archivos de sistema
.DS_Store
Thumbs.db
```

Verifica que no haya quedado nada suelto antes de cada `git add`:

```bash
git status
```

Si algo que no debería estar ahí aparece como "Untracked" o "Changes to be committed", revísalo antes de seguir. Si tienes duda de si un archivo con datos sensibles ya quedó trackeado en un commit anterior, dilo y se revisa antes de tocar nada — no se resuelve a ciegas.

### 1.3 Flujo básico (crear o actualizar)

```bash
cd "ruta/al/repo"
git status                     # qué cambió
git add archivo1 archivo2      # evita `git add .` a ciegas; añade lo que sí revisaste
git commit -m "Mensaje descriptivo"
git push origin main
```

### 1.4 Si el push es rechazado ("non-fast-forward")

Pasa cuando el repo remoto (GitHub) tiene un commit que tu copia local no tiene — por ejemplo, si editaste algo directamente en la web de GitHub. La solución es traer ese cambio antes de subir el tuyo:

```bash
git pull origin main
```

Esto puede abrir un editor de texto pidiendo un mensaje de merge — no hay que escribir nada, solo guardar y cerrar. Luego:

```bash
git push origin main
```

---

## Parte 2 — Despliegue como libro (mdBook + GitHub Pages)

Convierte el repo en un sitio web navegable y con buscador, publicado gratis en `https://<usuario>.github.io/<repo>/`.

### 2.1 Estructura que espera mdBook

```
vacio-y-soledad-en-el-ser-humano/
├── book.toml
├── src/
│   ├── SUMMARY.md
│   ├── 00_modulo_00_el_vacio_antes_del_orden.md
│   ├── 01_modulo_01_anatomia_del_vacio.md
│   └── ...
└── .github/
    └── workflows/
        └── deploy.yml
```

### 2.2 `book.toml` (plantilla)

```toml
[book]
title = "Vacío y Soledad en el Ser Humano"
authors = ["Dr. Pablo Eduardo Cancino Marentes"]
description = "Distinción ontológica, anatomía de la vacuidad y caminos de restauración en la experiencia bíblica y contemporánea"
language = "es"
src = "src"

[output.html]
default-theme = "light"
preferred-dark-theme = "navy"
git-repository-url = "https://github.com/PabloeCancino/vacio-y-soledad-en-el-ser-humano"
edit-url-template = "https://github.com/PabloeCancino/vacio-y-soledad-en-el-ser-humano/edit/main/{path}"

[output.html.search]
enable = true
limit-results = 20
```

### 2.3 `src/SUMMARY.md`

Índice de navegación del sitio — controla qué aparece en el menú lateral y en qué orden:

```markdown
# Índice

- [Módulo 00 — El vacío antes del orden](./00_modulo_00_el_vacio_antes_del_orden.md)
- [Módulo 01 — Anatomía del vacío](./01_modulo_01_anatomia_del_vacio.md)
- [Módulo 02 — Cuando la soledad hiere](./02_modulo_02_cuando_la_soledad_hiere.md)
- [Módulo 03 — El vacío del sentido perdido](./03_modulo_03_el_vacio_del_sentido_perdido.md)
- [Módulo 04 — La soledad elegida](./04_modulo_04_la_soledad_elegida.md)
- [Módulo 05 — De la soledad a la comunión](./05_modulo_05_de_la_soledad_a_la_comunion.md)
- [Módulo 06 — Caminos de restauración](./06_modulo_06_caminos_de_restauracion.md)
- [Módulo 07 — Acompañar al que está solo](./07_modulo_07_acompanar_al_que_esta_solo.md)
- [Apéndice A — Glosario](./08_apendice_a_glosario.md)
- [Apéndice B — Guía de sesiones](./09_apendice_b_guia_de_sesiones.md)
- [Apéndice C — Recursos](./10_apendice_c_recursos.md)
- [Apéndice D — Nota final](./11_apendice_d_nota_final.md)
```

### 2.4 Workflow de GitHub Actions (`.github/workflows/deploy.yml`)

```yaml
name: Publicar libro en GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el repositorio
        uses: actions/checkout@v4

      - name: Instalar mdBook
        uses: peaceiris/actions-mdbook@v2
        with:
          mdbook-version: "latest"

      - name: Compilar el libro
        run: mdbook build

      - name: Configurar GitHub Pages
        uses: actions/configure-pages@v5

      - name: Subir el sitio compilado
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./book

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Publicar en GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 2.5 Activar GitHub Pages

1. En el repo: pestaña **Settings**.
2. Menú lateral → **Pages**.
3. En "Build and deployment → Source", selecciona **GitHub Actions**.

### 2.6 Sitio final:

`https://pabloecancino.github.io/vacio-y-soledad-en-el-ser-humano/`
