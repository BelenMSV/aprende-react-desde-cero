# 06 - Despliegue a Producción y CI/CD 🚀

Bienvenido al último paso de tu viaje. Has construido una aplicación robusta, probada y escalable. Ahora, vamos a enseñarle al mundo lo que has creado.

En la ingeniería de software moderna, no "subimos archivos manualmente" a un servidor. Conectamos nuestro repositorio a plataformas inteligentes que leen nuestro código, lo construyen (Build) y lo distribuyen por todo el mundo de forma automática.

---

## 📦 1. Preparando el Paquete (El proceso de Build)

Vite es increíblemente rápido en desarrollo porque envía los archivos a tu navegador casi sin procesar. Pero en producción, no queremos enviar cientos de archivos `.jsx` pesados.

Queremos compilar todo en archivos estáticos puros (`.html`, `.js`, `.css`) minificados y ultra-comprimidos (lo que aprendimos en el Tema de Lazy Loading).

**Para compilar tu aplicación localmente, ejecutas:**
```bash
npm run build
```

Esto generará una carpeta llamada `dist/` (Distribution). **Esta carpeta contiene la versión final de tu aplicación.** Es la única carpeta que los servidores web necesitan para funcionar.

### 🔐 Variables de Entorno (El Peligro Número 1)
Antes de hacer el build, asegúrate de que tus variables de entorno apuntan a la base de datos real y no a la de pruebas.
Crea un archivo `.env.production` en la raíz de tu proyecto:
```env
# En desarrollo usas localhost, pero en producción usas tu API real
VITE_API_URL=[https://api.mi-empresa.com/v1](https://api.mi-empresa.com/v1)
```
Vite automáticamente usará este archivo cuando ejecutes `npm run build`.

---

## ☁️ 2. Hosting Frontend: Vercel y Netlify

Las aplicaciones de React (SPAs) son esencialmente archivos estáticos. No necesitas alquilar un servidor costoso (como AWS EC2) para alojarlas. 

La industria ha adoptado plataformas de despliegue sin servidor (Serverless) como **Vercel** (los creadores de Next.js) o **Netlify**.

**El flujo de trabajo moderno es así de simple:**
1. Subes tu código a **GitHub**.
2. Entras a Vercel.com y le das acceso a tu repositorio.
3. Vercel detecta automáticamente que es un proyecto de Vite.
4. Haces clic en "Deploy".
5. Vercel ejecuta `npm run build` en sus propios servidores y te da una URL pública en vivo (ej. `https://reactboard-pro.vercel.app`).

¡Y lo mejor de todo: **es gratis** para proyectos personales!

---

## 🤖 3. Nivel Arquitecto: CI/CD (GitHub Actions)

Conectar Vercel a GitHub significa que **cada vez que hagas un `git push`**, Vercel subirá tu código a producción automáticamente. Esto se llama **Despliegue Continuo (CD)**.

Pero, ¿qué pasa si subes código roto por accidente? Vercel lo desplegará igual, y tus usuarios verán una pantalla en blanco.

Aquí es donde entra la **Integración Continua (CI)**. Vamos a crear un "robot" en GitHub que ejecute nuestras pruebas automatizadas (del Tema 4) *antes* de permitir que el código llegue a producción.

### Configurando nuestro Guardián (GitHub Actions)

En la raíz de tu proyecto, crea las siguientes carpetas y un archivo YAML:
`.github/workflows/produccion.yml`

Añade este código de configuración:

```yaml
name: Guardián de Producción (CI/CD)

# 1. ¿Cuándo se ejecuta este robot?
on:
  push:
    branches: [ "main" ] # Solo cuando hacemos push a la rama principal
  pull_request:
    branches: [ "main" ] # O cuando alguien pide fusionar código

jobs:
  pruebas-y-build:
    runs-on: ubuntu-latest # El servidor gratuito que nos presta GitHub

    steps:
    # Paso 1: Clonar el código
    - name: 📥 Descargar repositorio
      uses: actions/checkout@v4

    # Paso 2: Instalar Node.js
    - name: ⚙️ Configurar Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20.x'

    # Paso 3: Instalar dependencias
    - name: 📦 Instalar paquetes
      run: npm ci

    # Paso 4: Ejecutar TODAS las pruebas de Vitest
    - name: 🧪 Correr Tests (React Testing Library)
      run: npm run test
      # ¡Si las pruebas fallan, el robot aborta todo el proceso aquí mismo!

    # Paso 5: Comprobar que el código compila sin errores
    - name: 🏗️ Compilar Producción (Vite Build)
      run: npm run build
```

### 🏆 El Flujo de Trabajo Definitivo:
1. Haces un cambio en tu código y haces `git push origin main`.
2. Vercel se prepara para desplegar.
3. El robot de **GitHub Actions** se despierta y ejecuta `npm run test`.
4. Si un test falla (ej. rompiste el formulario de login), el robot da una alerta roja ❌. **El despliegue se cancela**. Tus usuarios están a salvo.
5. Si todos los tests pasan ✅, el robot da luz verde. Vercel publica la nueva versión.

Esto es exactamente lo que ocurre cada vez que un ingeniero de Netflix o Spotify presiona "Guardar".

---

## 🎉 CONCLUSIÓN DEL CURSO

Si has llegado hasta aquí y has escrito el código de estos 6 módulos con tus propias manos, felicidades. Ya no eres la misma persona que empezó dudando sobre cómo funcionaba `useState`.

Has aprendido:
* Las bases filosóficas del Virtual DOM.
* A crear Custom Hooks y dominar los efectos secundarios.
* A construir formularios a prueba de balas con React Hook Form y Zod.
* A conectarte a APIs con Axios y dominar el caché con TanStack Query.
* A crear SPAs enrutadas, protegidas y con Lazy Loading usando React Router.
* A gestionar estados masivos con Zustand.
* A probar, arquitecturar y desplegar tu código a producción.

**Tienes los conocimientos de un Desarrollador Frontend Mid/Senior.** El límite ahora es tu imaginación. 

**¡Sal ahí fuera y construye algo increíble! 💻⚛️**