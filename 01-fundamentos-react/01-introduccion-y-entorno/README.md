# 01 - Introducción y Configuración del Entorno

React es una librería (no un framework) de JavaScript de código abierto diseñada para crear interfaces de usuario, desarrollada por Facebook (ahora Meta). Su principal ventaja es la creación de **Single Page Applications (SPAs)** rápidas y reactivas.

## 🛠️ Herramientas modernas: Adiós CRA, Hola Vite

Históricamente, la forma estándar de crear un proyecto en React era usando `create-react-app` (CRA). Sin embargo, hoy en día la comunidad y la propia documentación de React recomiendan usar herramientas más modernas y rápidas. Nosotros usaremos **Vite**.

Vite es una herramienta de construcción que proporciona una experiencia de desarrollo increíblemente rápida gracias a cómo utiliza los módulos ES nativos del navegador.

### Creando nuestro primer proyecto

A la hora de inicializar un proyecto con Vite, tienes dos caminos dependiendo de lo que prefieras en ese momento. Abre tu terminal y elige uno de los dos:

#### 1. El Modo "Asistente" (Interactivo)
```bash
npm create vite@latest mi-primer-proyecto
```
- **Cómo funciona:** Le dices a Vite "crea un proyecto en esta carpeta y ayúdame con el resto". La terminal entra en modo interactivo y te va haciendo preguntas (¿Quieres React, Vue o Svelte? ¿Quieres JavaScript o TypeScript?).
- **Cuándo usarlo:** Es ideal cuando estás empezando, cuando quieres explorar qué opciones nuevas ha metido Vite, o cuando simplemente no recuerdas el nombre exacto de la plantilla.

#### 2. El Modo "Directo" o "Atajo"
```bash
npm create vite@latest mi-primer-proyecto -- --template react
```
- **Cómo funciona:** Con ese `-- --template react`, le estás diciendo a la terminal: "Crea el proyecto y no me hagas ni una sola pregunta. Ponme directamente la plantilla de React con JavaScript". *(Nota técnica: el primer `--` sirve para avisarle a npm que los siguientes argumentos son para Vite).*
- **Cuándo usarlo:** Es el favorito de los desarrolladores cuando tienen prisa, o cuando se usan "scripts" automáticos en servidores o pipelines (CI/CD) donde no hay un humano para responder a las preguntas con las flechitas del teclado.

Una vez creado (con cualquiera de los dos métodos), entra en la carpeta, instala las dependencias y arranca el servidor de desarrollo:

```bash
cd mi-primer-proyecto
npm install
npm run dev
```

Tu aplicación estará corriendo habitualmente en `http://localhost:5173`.

## 📦 El corazón del proyecto: `package.json`

Al abrir tu proyecto, uno de los archivos más importantes que verás es el `package.json`. Piensa en él como el "carnet de identidad" y el "manual de instrucciones" de tu proyecto. Sus partes clave son:

- **`scripts`**: Son los comandos que puedes ejecutar en la terminal.
  - `"dev"`: Arranca el servidor de desarrollo local (el que usamos para programar).
  - `"build"`: Empaqueta tu aplicación y la prepara y optimiza para subirla a producción.
  - `"preview"`: Te permite probar localmente cómo se verá la versión de producción (`build`) antes de subirla.
- **`dependencies`**: Son los paquetes estrictamente necesarios para que tu aplicación *funcione* cuando el usuario la visite (ej. `react`, `react-dom`).
- **`devDependencies`**: Son herramientas que solo necesitas tú *durante el desarrollo* (ej. `vite`, `@eslint/js`). Estas no se enviarán al cliente final, haciendo tu app más ligera.

## 📂 ¿Dónde pongo mis archivos? `src` vs `public`

Vite genera dos carpetas principales que suelen generar confusión al principio: `public/` y `src/`. La regla de oro para diferenciarlas es saber **si Vite debe procesar o no esos archivos**.

### La carpeta `src/` (Source)
Aquí vivirá el 99% de tu código.
- **Qué contiene:** Componentes de React (`.jsx`), estilos (`.css`, `.scss`), imágenes que importes dentro de tus componentes, utilidades de JavaScript, etc.
- **Qué hace Vite con ella:** Cuando haces un `build`, Vite toma todo lo que hay aquí, lo compila, junta los archivos pequeños en unos pocos archivos más grandes, minifica el código (le quita espacios y comentarios) y les añade un "hash" (letras raras al final del nombre) para la caché del navegador.

### La carpeta `public/`
Es una carpeta de "escape" para cosas estáticas.
- **Qué contiene:** Archivos que nunca cambian y no necesitan ser procesados, como el `favicon.ico`, el `robots.txt` para Google, o archivos 3D/videos muy pesados.
- **Qué hace Vite con ella:** Nada. Literalmente copia y pega su contenido tal cual en la carpeta final de producción. Se accede a ellos desde la raíz de tu dominio (ej. `tusitio.com/favicon.ico`).

## 📁 Entendiendo el punto de entrada (`main.jsx`)

Al abrir la carpeta `src/`, verás el archivo `main.jsx`. Es el puente entre el HTML y tu código de React.

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

// 1. Seleccionamos el div con id "root" del index.html
const rootElement = document.getElementById('root');

// 2. Creamos la "raíz" de React
const root = ReactDOM.createRoot(rootElement);

// 3. Renderizamos nuestra aplicación dentro de la raíz
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### 🕵️‍♂️ ¿Qué es `<React.StrictMode>`?

Habrás notado que `<App />` está envuelto en `<React.StrictMode>`. Esto es una herramienta de desarrollo que:

1. **No renderiza nada en la interfaz de usuario.**
2. **Ejecuta los componentes dos veces en desarrollo** (esto es normal y sirve para detectar efectos secundarios impuros, como fetchings duplicados o mutaciones accidentales).
3. **Advierte sobre el uso de APIs obsoletas.**

Es una excelente práctica dejarlo activado mientras desarrollamos.

## 📝 Ejercicio

1. Inicializa un proyecto con Vite llamado `react-sandbox` usando tu método preferido de la terminal.
2. Limpia los archivos predeterminados: borra todo el contenido de `App.jsx` y haz que solo retorne un `<h1>Hola Mundo desde React</h1>`.
3. Mueve una imagen cualquiera a la carpeta `public/` e intenta mostrarla en tu componente `App` usando la ruta `/nombre-de-tu-imagen.jpg`.
4. Borra el contenido de `App.css` y `index.css` para empezar con un lienzo en blanco.