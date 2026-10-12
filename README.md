# # SOCIAL OPLESK

## 🏴‍☠️ HACKS

---

## ⚡ Hack DevOps ⚡

---

### Requisitos previos

```
- NOTA: HACER LAS PRÁCTICAS MEDIANTE LA CONSOLA - TERMINAL
- Debes tener cuenta en GitLab (y en Vercel para el H-2)
- Si usas Windows, emplea la terminal de WSL, ideal sobre Debian
- Tener git instalado: git --version
```

| Hacks | Details |
|-------|---------|
| H-1   | pipeline en GitLab CI/CD con 3 stages, 3 jobs y artifacts |
| H-2   | lint + minify (HTML, CSS, JS) + deploy a Vercel con GitLab CI/CD |

> El **H-1** es la base del **H-2**: primero entiendes cómo fluye un pipeline y cómo se pasan archivos entre jobs, después construyes un pipeline real.

---

## 🏆 H-1

### Pipeline de 3 stages con artifacts

```
📁 hack_gl_1/
└── .gitlab-ci.yml
```

### Objetivo

Entender las piezas básicas de GitLab CI/CD: **stages**, **jobs**, **imágenes Docker**, **scripts**, **artifacts** y **dependencias entre jobs** (`needs`).

### Pasos

1. Crear un repositorio vacío en GitLab llamado `hack_gl_1` y clonarlo en tu terminal.

```bash
git clone https://gitlab.com/<tu-usuario>/hack_gl_1.git
cd hack_gl_1
```

2. Crear el archivo `.gitlab-ci.yml` en la raíz y pegar el script de abajo.
3. Subir los cambios.

```bash
git add .
git commit -m "pipeline inicial"
git push origin main
```

4. En GitLab ir a **Build → Pipelines** y abrir el pipeline recién creado.
5. Entrar a cada job y leer el log completo.
6. Descargar los **artifacts** desde la página del job y abrir `build/datos.txt`.

### Script

```yaml
stages:
  - saludar
  - adios
  - cerrar

tarea-saludar:
  image: alpine:latest
  stage: saludar
  before_script:
    - apk add --no-cache git
  script:
    - echo "git version"
    - git --version
    - echo "hola desde gitlab"
    - mkdir -p build
    - mkdir -p dist
    - echo "contenido creado en tarea-saludar" > build/datos.txt
    - echo "<body>" > dist/index.html
    - echo "<h1>hola mundo creado en tarea saludar</h1>" >> dist/index.html
    - echo "</body>" >> dist/index.html
  artifacts:
    paths:
      - build
      - dist
    expire_in: 5 min

tarea-despedida:
  image: ubuntu:latest
  stage: adios
  needs: ["tarea-saludar"]
  before_script:
    - apt-get update -y && apt-get install git -y
  script:
    - echo "git version"
    - git --version
    - echo "esto es un adios"
    - cat build/datos.txt
    - cat dist/index.html
    - echo "contenido adjunto desde tarea-despedida" >> build/datos.txt
  artifacts:
    paths:
      - build
    expire_in: 5 min

tarea-cerrar:
  image: ubuntu:latest
  stage: cerrar
  needs: ["tarea-despedida"]
  script:
    - echo "tarea-cerrar"
    - cat build/datos.txt
```

### ¿Qué hace cada pieza?

| Elemento | Para qué sirve |
|----------|----------------|
| `stages` | Define el orden de las etapas del pipeline |
| `image` | Imagen Docker donde se ejecuta el job (`alpine` o `ubuntu`) |
| `before_script` | Comandos que corren antes del `script` (aquí, instalar git) |
| `script` | Comandos principales del job |
| `artifacts` | Archivos que el job guarda para que otros jobs o tú los usen |
| `expire_in` | Tiempo que se conservan los artifacts |
| `needs` | Indica de qué job depende y de cuál descarga los artifacts |

### Preguntas de comprensión

Responde en tu `README.md` del repositorio:

1. ¿Por qué `tarea-despedida` puede leer `build/datos.txt`?
2. ¿Qué contenido tiene `build/datos.txt` cuando lo imprime `tarea-cerrar`?
3. ¿Por qué `tarea-cerrar` **no** puede leer `dist/index.html`?
4. ¿Qué diferencia hay entre usar `alpine` y `ubuntu` en cuanto a instalación de paquetes (`apk` vs `apt-get`)?

### Retos

Haz un commit por cada reto y verifica el pipeline después de cada push.

| Reto | Qué hacer |
|------|-----------|
| 1 | Cambiar el texto del `<h1>` y comprobar el cambio en el log de `tarea-despedida` |
| 2 | Agregar un job `tarea-extra` en el stage `adios` que corra **en paralelo** con `tarea-despedida`. Observa cómo se ve en el grafo del pipeline |
| 3 | Hacer que `tarea-cerrar` también muestre `dist/index.html` (pista: revisa los `artifacts` de `tarea-despedida` o el `needs` de `tarea-cerrar`) |
| 4 | Provocar un fallo a propósito (un comando que no exista en `tarea-despedida`) y observar qué pasa con `tarea-cerrar` |
| 5 | Agregar una variable en `variables:` y usarla en un `echo` |
| 6 | Limitar `tarea-cerrar` para que solo corra en la rama `main` usando `rules` |

### Entregables

- URL del repositorio en GitLab
- Captura del pipeline en verde con los 3 jobs
- Captura del pipeline fallando (reto 4)
- Respuestas a las preguntas de comprensión

---

## 🏆 H-2

### Lint + Minify + Deploy a Vercel

```
📁 hack_gl_2/
├── .gitlab-ci.yml
├── .htmlhintrc
├── index.html
├── main.js
├── style.css
└── README.md
```

### Contexto

Tienes un sitio web estático mínimo (HTML, CSS y JS). Cada cambio debe pasar por una validación automática, los archivos deben optimizarse antes de publicarse y el despliegue a producción debe hacerse sin intervención manual.

### Objetivo

Construir un pipeline en **GitLab CI/CD** con tres etapas (`lint`, `build` y `deploy`) que valide el HTML, minifique los tres tipos de archivo y publique el resultado en **Vercel**.

### Herramientas

| Tarea | Herramienta | Documentación |
|-------|-------------|---------------|
| Lint de HTML | HTMLHint | https://htmlhint.com/getting-started/ |
| Minificar HTML | html-minifier-terser | https://github.com/terser/html-minifier-terser |
| Minificar CSS | clean-css-cli | https://github.com/clean-css/clean-css-cli |
| Minificar JS | Terser | https://github.com/terser/terser |
| Despliegue | Vercel CLI | https://vercel.com/docs/cli |

### Requisitos

#### 1. Preparación del proyecto

- Enlazar `style.css` y `main.js` desde `index.html`.
- Añadir contenido real a `style.css` y `main.js` (estilos básicos y alguna interacción simple), con comentarios y espacios para que la minificación tenga efecto.
- Configurar `.htmlhintrc` con al menos estas reglas: `doctype-first`, `tag-pair`, `tagname-lowercase`, `attr-lowercase`, `attr-value-double-quotes`, `title-require`, `id-unique`, `src-not-empty`.

#### 2. Etapa `lint`

- Ejecutar HTMLHint sobre `index.html`.
- Si hay errores, el pipeline **debe fallar** y no continuar.
- Debe ejecutarse en todas las ramas y en los merge requests.

#### 3. Etapa `build`

- Generar la carpeta `dist/` con los tres archivos minificados:
  - `dist/index.html` con html-minifier-terser (colapsar espacios y eliminar comentarios).
  - `dist/style.css` con clean-css-cli.
  - `dist/main.js` con Terser (`--compress --mangle`).
- Publicar `dist/` como **artifact** con caducidad de 1 hora.
- Mostrar en el log el tamaño de cada archivo antes y después (por ejemplo con `wc -c`).

#### 4. Etapa `deploy`

- Desplegar `dist/` a producción con Vercel CLI.
- Ejecutarse solo en la **rama por defecto**.
- Usar `resource_group` para evitar despliegues simultáneos.
- El token **no puede estar escrito en el repositorio**: usar variables de CI/CD de GitLab.

#### 5. Configuración general

- Imagen base de Node.
- `timeout` del job de 10 minutos.
- `retry` de 1 intento solo ante `runner_system_failure`.
- Reutilizar la instalación de dependencias con plantillas (`.nombre` + `extends`) o `before_script`.

### Variables de CI/CD

En **Settings → CI/CD → Variables** del proyecto:

| Variable | Descripción | Opciones recomendadas |
|----------|-------------|-----------------------|
| `VERCEL_TOKEN` | Token personal creado en Vercel | Masked, Protected |
| `VERCEL_ORG_ID` | ID de tu cuenta u organización | Opcional según cómo enlaces el proyecto |
| `VERCEL_PROJECT_ID` | ID del proyecto en Vercel | Opcional según cómo enlaces el proyecto |

### Pruebas obligatorias

1. **Caso correcto:** un commit que cambia `<h1>VERSION: 1.0.1</h1>` a `1.0.2` y se despliega en Vercel (verifica la URL pública).
2. **Fallo del lint:** introduce un error a propósito (por ejemplo, una etiqueta sin cerrar). El pipeline debe detenerse en `lint` y no ejecutar `build` ni `deploy`.
3. **Rama de feature:** crea una rama, haz un cambio y comprueba que se ejecutan `lint` y `build`, pero **no** `deploy`.

### Pistas

```bash
npx htmlhint index.html
terser main.js --compress --mangle -o dist/main.js
cleancss -o dist/style.css style.css
html-minifier-terser --collapse-whitespace --remove-comments -o dist/index.html index.html
vercel deploy dist --prod --token="$VERCEL_TOKEN" --yes
```

> El paquete de clean-css se instala como `clean-css-cli` y el comando es `cleancss`.

### Entregables

- URL del repositorio en GitLab
- `.gitlab-ci.yml`, `.htmlhintrc` y el resto de archivos del proyecto
- URL del sitio desplegado en Vercel
- Capturas de los tres pipelines de las pruebas obligatorias
- `README.md` con una breve explicación de cada etapa y las decisiones tomadas

### Checklist de verificación

Marca cada ítem cuando lo hayas comprobado en tu pipeline:

**Lint**
- [ ] El job `lint` ejecuta HTMLHint sobre `index.html`
- [ ] `.htmlhintrc` incluye las 8 reglas indicadas
- [ ] Un error en el HTML detiene el pipeline y no se ejecutan `build` ni `deploy`

**Build**
- [ ] Se genera `dist/index.html`, `dist/style.css` y `dist/main.js` minificados
- [ ] `dist/` se publica como artifact con caducidad de 1 hora
- [ ] El log muestra el tamaño de los archivos antes y después

**Deploy**
- [ ] El deploy solo se ejecuta en la rama por defecto
- [ ] El job usa `resource_group` para evitar despliegues simultáneos
- [ ] El sitio es accesible desde la URL pública de Vercel
- [ ] El cambio de versión (`1.0.1` → `1.0.2`) se ve reflejado en producción

**Seguridad y configuración**
- [ ] `VERCEL_TOKEN` está en variables de CI/CD (Masked y Protected) y no en el repositorio
- [ ] Se define imagen de Node, `timeout` de 10 minutos y `retry` solo ante `runner_system_failure`
- [ ] La instalación de dependencias se reutiliza con plantillas (`extends`) o `before_script`

**Evidencias**
- [ ] Captura del pipeline correcto
- [ ] Captura del pipeline detenido en `lint`
- [ ] Captura del pipeline de rama de feature (solo `lint` y `build`)
- [ ] `README.md` del repositorio con la explicación de cada etapa

---
<h3 align="center">SOCIAL OPLESK</h3>
