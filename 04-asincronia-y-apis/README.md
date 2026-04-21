# Módulo 04: Asincronía y APIs

Hasta ahora, nuestra aplicación ha funcionado en un entorno perfecto e instantáneo: la memoria del navegador. Pero las aplicaciones reales necesitan hablar con el mundo exterior. 

En el momento en que nos conectamos a una API, introducimos latencia, errores de red, fallos de servidor y problemas de sincronización. Manejar estos problemas manualmente con `useState` y `useEffect` resulta en un código frágil y difícil de mantener.

En este módulo aprenderemos la evolución del *fetching* de datos en React. Empezaremos con las bases nativas para entender el dolor, y rápidamente escalaremos a las herramientas de grado empresarial (**Axios y TanStack Query**) que manejan el caché y la sincronización como por arte de magia.

**⏱️ Tiempo estimado de estudio y práctica:** 15 - 20 horas.

## 🎯 Objetivos de Aprendizaje

- Dominar el flujo asíncrono (`async/await`) dentro del ecosistema de React.
- Crear una **Arquitectura de Servicios** limpia utilizando **Axios** e Interceptores.
- Entender y prevenir las **Condiciones de Carrera (Race Conditions)** cancelando peticiones en vuelo con `AbortController`.
- Comprender el cambio de paradigma moderno: **Client State vs Server State**.
- Implementar **TanStack Query (React Query)** para eliminar el código repetitivo de *fetching*, manejar caché y re-intentos automáticos.
- Construir interfaces complejas de datos: Paginación y Scroll Infinito.
- Lograr una UX impecable mediante **Actualizaciones Optimistas (Optimistic Updates)**.
- Utilizar las APIs modernas de React 18: `<Suspense>` y Error Boundaries.

## 📚 Temario Detallado

1. **[Asincronía en React: Promesas y useEffect](./01-asincronia-en-react/README.md)**
   > Repaso de cómo interactúa el Event Loop de JavaScript con el ciclo de renderizado de React. Cómo y dónde usar `async/await` correctamente sin romper los Hooks.

2. **[Arquitectura de Servicios con Axios](./02-arquitectura-de-servicios-axios/README.md)**
   > Abandona el código espagueti. Aprende a crear instancias de Axios, separar las llamadas a la API en su propia capa lógica y usar Interceptores para inyectar *tokens* de seguridad.

3. **[El Peligro de la Red: Race Conditions](./03-race-conditions-y-abort-controller/README.md)**
   > ¿Qué pasa si el usuario hace clic rápido en 3 perfiles distintos? Aprende por qué los datos se mezclan y cómo usar `AbortController` para cancelar peticiones obsoletas.

4. **[El Cambio de Paradigma: Servidor vs Cliente](./04-estado-servidor-vs-cliente/README.md)**
   > La teoría que cambió React. Por qué guardar los datos de una API en un `useState` o en Redux se considera hoy un anti-patrón de arquitectura.

5. **[TanStack Query: El Estándar de la Industria](./05-tanstack-query-el-estandar/README.md)** 🚀
   > Tu vida como desarrollador se divide en "Antes de TanStack Query" y "Después de TanStack Query". Aprende a manejar caché, *stale time*, *loading states* y errores con una sola línea de código.

6. **[Patrones de Datos: Paginación e Infinite Scroll](./06-paginacion-e-infinite-scroll/README.md)**
   > Pasa de cargar 10 elementos a cargar miles. Cómo usar los Hooks avanzados de TanStack Query (`useQuery` paginado y `useInfiniteQuery`) para interfaces pesadas.

7. **[UX Mágica: Mutaciones y Optimistic Updates](./07-mutaciones-y-optimistic-updates/README.md)** 💎
   > ¿Has notado que al dar 'Like' en Twitter el corazón se pinta al instante sin esperar al servidor? Aprende a engañar al usuario para crear una experiencia de velocidad extrema (y cómo revertirla si el servidor falla).

8. **[React 18+: Suspense y Error Boundaries](./08-react-18-suspense-y-error-boundaries/README.md)**
   > El futuro de React. Aprende a eliminar los `if (isLoading)` de tus componentes, delegando los estados de carga y los errores a componentes envolventes declarativos.

---

[🔙 Volver al Índice Global](../README.md)