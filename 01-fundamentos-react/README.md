# Módulo 01: Fundamentos y Filosofía de React

Bienvenido al primer módulo. Antes de construir aplicaciones complejas, necesitamos entender cómo "piensa" React. En este módulo no solo aprenderemos sintaxis, sino los patrones de diseño y la arquitectura mental necesarios para dominar la librería.

**⏱️ Tiempo estimado de estudio y práctica:** 12 - 15 horas.

## 🎯 Objetivos de Aprendizaje

- Configurar un entorno de desarrollo moderno usando Vite.
- Comprender el funcionamiento del Virtual DOM y React Fiber.
- Escribir JSX limpio respetando sus reglas estrictas y evitando ataques XSS.
- Construir interfaces modulares usando Componentes Funcionales.
- Comunicar componentes y controlar qué se renderiza en pantalla bajo distintas condiciones.
- Dominar el ecosistema de estilos moderno (CSS Modules, Tailwind CSS, daisyUI).

## 📚 Temario Detallado

1. **[Introducción y Configuración del Entorno](./01-introduccion-y-entorno/README.md)**
   > Configura tu primer proyecto con Vite, entiende la estructura de carpetas (`src` vs `public`) y el papel del `package.json`.

2. **[¿Cómo piensa React? (Virtual DOM)](./02-como-piensa-react/README.md)**
   > Descubre la diferencia entre programación imperativa y declarativa, y por qué el Virtual DOM hace que React sea tan rápido.

3. **[Profundizando en JSX](./03-profundizando-en-jsx/README.md)**
   > Domina la sintaxis que mezcla HTML y JavaScript. Aprende sus 3 reglas de oro y cómo inyectar variables de forma segura.

4. **[Componentes Funcionales](./04-componentes-funcionales/README.md)**
   > Aprende a crear bloques de construcción reutilizables y descubre la mejor arquitectura de carpetas para proyectos reales (el patrón `index.js`).

5. **[Props y Comunicación entre componentes](./05-props-y-comunicacion/README.md)**
   > Haz que tus componentes cobren vida pasando datos de padres a hijos. Aprende a desestructurar y a usar el operador *spread*.

6. **[Renderizado Condicional](./06-renderizado-condicional/README.md)**
   > Controla qué se muestra en pantalla (spinners, errores, interfaces) aplicando técnicas de *Clean Code* con `if`, ternarios y `&&`.

7. **[Listas y la importancia de las Keys](./07-listas-y-keys/README.md)**
   > Transforma arrays de datos en listas visuales usando `.map()`, y entiende por qué usar el `index` como `key` puede romper tu aplicación.

8. **[Manejo de Estilos en React](./08-estilos-en-react/README.md)**
   > Explora desde el CSS tradicional hasta las herramientas que dominan la industria hoy: Tailwind CSS, daisyUI y la función utilitaria `cn`.

9. **[Composición y la prop 'children'](./09-composicion-y-children/README.md)**
   > Eleva tu nivel de arquitectura. Aprende a crear *Layouts*, el patrón de *Slots* y cómo evitar el *Prop Drilling* delegando responsabilidades.

---

[🔙 Volver al Índice Global](../README.md)