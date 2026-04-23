# 07 - Rendimiento Extremo: Lazy Loading y Code Splitting 💎

El **Code Splitting** (División de Código) es la técnica de romper ese archivo gigante (`bundle.js`) en pedazos más pequeños (Chunks). 
El **Lazy Loading** (Carga Perezosa) es la acción de pedir esos pedazos *solo* cuando el usuario los necesita.

Si el usuario nunca entra a la pantalla `/reportes`, su navegador **nunca** descargará el código de esa pantalla. 

---

## 🐢 1. El Problema del "Eager Loading" (Carga Ansiosa)

Mira tu archivo de rutas (`main.jsx`) actual. Fíjate en los `imports` de arriba:

❌ **El Anti-patrón:**
```jsx
// 🚩 Al hacer esto, obligas al navegador a descargar el código de todas estas 
// pantallas ANTES de que la aplicación pueda arrancar.
import { PaginaInicio } from './pages/PaginaInicio';
import { PanelAdmin } from './pages/PanelAdmin'; // Código pesado
import { GraficasGigantes } from './pages/GraficasGigantes'; // Código masivo

const router = createBrowserRouter([
  { path: '/', element: <PaginaInicio /> },
  { path: '/admin', element: <PanelAdmin /> },
  { path: '/graficas', element: <GraficasGigantes /> },
]);
```

---

## ⚡ 2. La Solución Clásica: `React.lazy` y `<Suspense>`

Para evitar que las importaciones ocurran de inmediato, usamos la función `React.lazy()`. Esta función toma una promesa (el import dinámico) y la convierte en un componente de React.

Como el componente tardará unos milisegundos en descargarse a través de la red cuando el usuario navegue hacia él, React nos obliga a usar `<Suspense>` (del Tema 8, Módulo 4) para mostrar un fallback (Spinner).

✅ **Optimizando con React puro:**
```jsx
import { Suspense, lazy } from 'react';
import { createBrowserRouter } from 'react-router-dom';

// 1. Eager Loading (Carga inmediata):
// La página de inicio SIEMPRE debe cargar rápido, no le hacemos lazy loading.
import { PaginaInicio } from './pages/PaginaInicio';

// 2. Lazy Loading (Carga perezosa):
// Estas páginas solo se descargarán si el usuario navega hacia ellas.
const PanelAdmin = lazy(() => import('./pages/PanelAdmin'));
const GraficasGigantes = lazy(() => import('./pages/GraficasGigantes'));

const router = createBrowserRouter([
  { 
    path: '/', 
    element: <PaginaInicio /> 
  },
  { 
    path: '/admin', 
    element: (
      // 3. Envolvemos con Suspense para manejar el tiempo de descarga del archivo
      <Suspense fallback={<p>Descargando módulo Admin...</p>}>
        <PanelAdmin />
      </Suspense>
    ) 
  },
]);
```

---

## 🚀 3. Nivel Arquitecto: Lazy Routes en React Router v6.4+

Si usas el nuevo paradigma de React Router (el de `createBrowserRouter`), poner `<Suspense>` por todas partes se vuelve tedioso. 

Los creadores de React Router añadieron una propiedad mágica llamada `lazy` directamente a la configuración de rutas. Hace exactamente lo mismo, pero con una sintaxis mucho más limpia, ¡y además puede importar los `loaders` y `actions` al mismo tiempo!

✅ **El Estándar Moderno:**

```jsx
// src/main.jsx
import { createBrowserRouter } from 'react-router-dom';
import { PaginaInicio } from './pages/PaginaInicio';

const router = createBrowserRouter([
  { 
    path: '/', 
    element: <PaginaInicio /> // La inicial siempre se importa normal
  },
  {
    path: '/graficas',
    // Usamos la propiedad 'lazy' en lugar de 'element' y 'loader'.
    // React Router hará el code-splitting y mostrará el Spinner global 
    // automáticamente usando tu configuración de useNavigation()
    lazy: async () => {
      // Importamos el archivo dinámicamente
      const modulo = await import('./pages/GraficasGigantes');
      
      // Devolvemos el Componente y (opcionalmente) su Loader
      return { 
        Component: modulo.GraficasGigantes,
        loader: modulo.graficasLoader 
      };
    }
  }
]);
```

---

## 🏆 4. Reglas de Oro del Lazy Loading

Saber *cómo* usar Lazy Loading es fácil. Saber *cuándo* usarlo separa a los Juniors de los Seniors.

1. **NUNCA hagas Lazy Load de la primera pantalla visible:** Si haces lazy load de la `/`, el usuario primero descargará el archivo principal, luego verá un spinner, y *luego* descargará el archivo de inicio. Es una pérdida de tiempo.
2. **Separa por "Paredes de Autenticación":** Todo lo que está detrás del Login (ej. `/dashboard`, `/perfil`) debe ser un *chunk* separado. Si un visitante no logueado entra, no tiene sentido que descargue código privado.
3. **Aísla librerías gigantes:** Si una página usa una librería pesada (ej. `Three.js` para 3D, o `Chart.js` para gráficas), asegúrate de que esa página use Lazy Load para que esas librerías no inflen el peso del resto de la aplicación.

---

### 🎉 ¡Felicidades! Has superado el Módulo 5.
Has dejado atrás la creación de componentes aislados y ahora sabes diseñar la Arquitectura completa de una SPA: navegación protegida, cargas optimizadas y un estado global impecable.