# Módulo 05: Enrutamiento y Estado Global

Nuestra aplicación ya sabe comunicarse con el servidor, manejar formularios complejos y gestionar el caché. Pero hasta ahora, todo ocurre en una sola vista. Las verdaderas aplicaciones web tienen decenas de URLs distintas (`/inicio`, `/perfil`, `/dashboard/reportes`). 

En este módulo daremos el salto definitivo creando una verdadera **Single Page Application (SPA)** utilizando el estándar indiscutible: **React Router v6**. Aprenderemos a diseñar la arquitectura de navegación, proteger rutas privadas y optimizar la carga.

Además, resolveremos el infame problema del *Prop Drilling* (pasar propiedades de padres a hijos a través de 10 niveles) dominando las dos herramientas esenciales de la industria moderna para el Estado Global del Cliente: **Context API** y **Zustand**.

**⏱️ Tiempo estimado de estudio y práctica:** 15 - 20 horas.

## 🎯 Objetivos de Aprendizaje

- Dominar la API moderna de **React Router v6** (`createBrowserRouter`).
- Diseñar Layouts reutilizables mediante **Rutas Anidadas** y el componente `<Outlet>`.
- Capturar parámetros dinámicos de la URL (ej. `/usuario/123`) para mostrar contenido específico.
- Implementar **Auth Guards** (Rutas Protegidas) para bloquear usuarios no autenticados.
- Aprovechar las funciones de nivel Arquitecto de React Router v6.4+ (`loaders` y `actions`) para evitar cascadas de renderizado.
- Identificar cuándo usar **Context API** (Inyección de dependencias) y cuándo NO usarlo.
- Implementar **Zustand** para gestionar el estado global de la UI de forma predecible y sin re-renderizados innecesarios.
- Optimizar el tamaño de descarga de la aplicación utilizando **Code Splitting** y **Lazy Loading**.

## 📚 Temario Detallado

1. **[Fundamentos de React Router v6](./01-react-router-fundamentos/README.md)**
   > Abandona las etiquetas `<a>` tradicionales que recargan la página. Aprende a configurar el enrutador moderno y a navegar entre pantallas usando `<Link>` y `useNavigate` manteniendo el estado intacto.

2. **[Arquitectura de Layouts: Rutas Dinámicas y Anidadas](./02-rutas-dinamicas-y-anidadas/README.md)**
   > ¿Cómo mantienes un menú lateral fijo mientras solo cambia el contenido central? Aprende el poder del componente `<Outlet>` y cómo leer parámetros como IDs directamente desde la URL.

3. **[Seguridad en el Frontend: Auth Guards](./03-proteccion-de-rutas-auth-guards/README.md)**
   > No dejes que usuarios anónimos entren a tu Panel de Control. Construye componentes "Guardianes" que intercepten la navegación y redirijan al Login si falta un token.

4. **[El Futuro del Enrutamiento: Loaders y Actions](./04-react-router-avanzado-loaders-actions/README.md)** 🚀
   > Nivel Arquitecto. Aprende cómo React Router v6.4+ permite cargar los datos de una ruta *antes* de que el componente siquiera exista, eliminando los spinners intermedios por completo.

5. **[El Infierno del Prop Drilling y Context API](./05-prop-drilling-y-context-api/README.md)**
   > Entiende el problema de pasar información por 10 niveles de componentes y cómo usar Context API como un "teletransportador" de datos (y por qué usarlo para todo destruirá el rendimiento de tu app).

6. **[Zustand: El Estándar Moderno de Estado Global](./06-zustand-el-estandar-moderno/README.md)** 🐻
   > Redux quedó en el pasado por ser demasiado complejo. Descubre por qué la industria migró a Zustand: un gestor de estado diminuto, increíblemente rápido y que requiere cero configuración inicial.

7. **[Rendimiento Extremo: Lazy Loading y Code Splitting](./07-performance-lazy-loading/README.md)** 💎
   > Si tu app tiene 50 pantallas, el usuario no debería descargar el código de las 50 pantallas al entrar al Login. Aprende a "cortar" tu código en pedazos usando `React.lazy` y cargarlo solo cuando el usuario lo necesita.

---

[🔙 Volver al Índice Global](../README.md)