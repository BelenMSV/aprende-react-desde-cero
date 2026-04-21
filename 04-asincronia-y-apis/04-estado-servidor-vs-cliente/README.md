# 04 - El Cambio de Paradigma: Estado Servidor vs Estado Cliente

Antes de escribir una sola línea de código con la herramienta que veremos en el siguiente tema, necesitas entender un concepto que cambió para siempre la forma en que construimos aplicaciones React: **La separación del Estado**.

Hace unos años, era común usar librerías de "Estado Global" (como Redux o el Context API) para guardar absolutamente todo: si un modal estaba abierto, el tema oscuro de la app, y también la lista de 1,000 usuarios que venía de la base de datos. 

Hoy, sabemos que mezclar esto es un error de arquitectura. Existen dos tipos de estado que se comportan de manera radicalmente opuesta.

---

## 💻 1. Estado del Cliente (Client State / UI State)

El Estado del Cliente es información que **le pertenece exclusivamente a la interfaz gráfica** y al dispositivo del usuario.

**Características:**
* **Síncrono e Instantáneo:** Cambia en el momento exacto en que el usuario interactúa (ej. hacer clic).
* **Efímero:** Si recargas la página, este estado suele desaparecer (a menos que lo guardes en el `localStorage`).
* **Privado:** Nadie más en el mundo puede ver ni modificar este estado. Es solo para el usuario que está frente a la pantalla.
* **Control Total:** Tú (el desarrollador frontend) tienes el 100% del control sobre cuándo y cómo cambia.

**Ejemplos:**
* ¿El menú lateral está abierto o cerrado? `isOpen: true`
* ¿Qué pestaña de navegación está activa? `activeTab: 'perfil'`
* ¿Qué ha escrito el usuario en un input controlado antes de enviarlo? `text: 'Hola'`

**Herramientas para manejarlo:**
* `useState` y `useReducer` (Para estado local en componentes).
* `Zustand`, `Jotai` o `Context API` (Para estado global de la UI).

---

## ☁️ 2. Estado del Servidor (Server State / Server Cache)

El Estado del Servidor es información que **le pertenece a una base de datos lejana**, y nosotros solo tenemos una "fotografía" temporal de ella en nuestra pantalla.

**Características:**
* **Asíncrono e Impredecible:** Tarda en llegar, puede fallar por problemas de red, y requiere manejar estados de carga (`isLoading`).
* **Persistente:** Sobrevive a recargas de página porque vive en la base de datos.
* **Compartido:** Muchos usuarios pueden estar leyendo o modificando esta misma información al mismo tiempo.
* **Obsoleto (Stale):** En el mismo instante en que la información llega a tu pantalla, ya podría ser vieja. *(Imagina que cargas un post de Twitter con 10 Likes. En los 5 minutos que pasas leyendo, el post ya tiene 5,000 Likes en la base de datos real, pero tú sigues viendo 10)*.

**Ejemplos:**
* El perfil del usuario desde la base de datos.
* La lista de productos de un e-commerce.
* El feed de noticias de una red social.

---

## 💥 3. El Problema de usar Herramientas Equivocadas

Si intentas manejar el Estado del Servidor utilizando herramientas de Estado del Cliente (como `useState` o `Redux`), te enfrentarás a un infierno de problemas de sincronización.

Tendrás que escribir código manual para:
1. **Deduplicación:** Evitar hacer la misma petición a la API 3 veces si el usuario hace clics rápidos.
2. **Caché:** Guardar temporalmente los datos para no volver a pedirlos si el usuario navega a otra pantalla y vuelve.
3. **Background Fetching:** Actualizar los datos en segundo plano sin que el usuario se dé cuenta para que no vean información obsoleta (Stale).
4. **Race Conditions:** Lo que resolvimos en el Tema 3 con el `AbortController`.

❌ **El Anti-patrón:**
```jsx
// 🚩 Intentar gestionar el Estado del Servidor como si fuera Estado del Cliente
const [usuarios, setUsuarios] = useState([]);
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  // Lógica pesada, abort controllers, try/catch, etc...
}, []);
```

---

## 🚀 4. La Arquitectura Moderna

Los Arquitectos de Software modernos han decidido sacar el Estado del Servidor fuera de las herramientas tradicionales de React, delegándolo a librerías especializadas en sincronización, caché y promesas.

**Regla de Oro:**
* **Zustand / useState:** Úsalos para saber si el "Modo Oscuro" está activado.
* **TanStack Query (React Query) / SWR:** Úsalos para traer, guardar en caché y mantener actualizados los datos de tu API.

En el próximo tema, aprenderemos a usar la herramienta que ha estandarizado la industria para el Estado del Servidor: **TanStack Query**. 
Te prometo que, una vez que la aprendas, no querrás volver a usar `useEffect` para hacer un `fetch` nunca más.