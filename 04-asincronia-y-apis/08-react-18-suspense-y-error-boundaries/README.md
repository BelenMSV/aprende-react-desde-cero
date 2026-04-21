# 08 - React 18+: Suspense y Error Boundaries

Hasta ahora, nuestro manejo de asincronía ha sido "imperativo". Obligamos a cada componente a ser consciente de su propio estado de carga y de sus propios errores.

❌ **El problema del código imperativo:**
```jsx
export const Perfil = () => {
  const { data, isLoading, isError } = useQuery(...);

  // 🚩 El componente está contaminado con lógica de control
  if (isLoading) return <Spinner />;
  if (isError) return <PantallaRoja />;

  return <div>{data.nombre}</div>;
};
```

Si tienes una página con 5 componentes distintos haciendo esto, la pantalla parecerá un árbol de Navidad: spinners apareciendo y desapareciendo por todas partes, y si uno falla, puede romper la experiencia entera.

La solución de React 18 es sacar esta lógica **fuera** del componente usando envoltorios declarativos: `<Suspense>` y `<ErrorBoundary>`.

---

## ⏳ 1. Suspense (Delegando el estado de carga)

**Suspense** le permite a un componente decirle a React: *"Oye, todavía no estoy listo para pintarme. Ponme en pausa y, mientras esperas, dibuja este Spinner"*.

Para usar esto con TanStack Query (versión 5), en lugar de usar `useQuery`, usamos el hook especializado **`useSuspenseQuery`**.

✅ **1. El componente se vuelve puramente visual (Cero ifs):**
```jsx
import { useSuspenseQuery } from '@tanstack/react-query';

// El componente asume que los datos YA EXISTEN. 
// No hay 'isLoading', no hay comprobaciones de nulos.
export const TarjetaUsuario = () => {
  const { data } = useSuspenseQuery({
    queryKey: ['usuario'],
    queryFn: fetchUsuario
  });

  return <div>Bienvenido, {data.nombre}</div>;
};
```

✅ **2. El Padre controla la Experiencia de Usuario:**
```jsx
import { Suspense } from 'react';
import { TarjetaUsuario } from './TarjetaUsuario';

export const PanelPrincipal = () => {
  return (
    <div className="dashboard">
      <h1>Mi Aplicación</h1>
      
      {/* Suspense atrapa a cualquier hijo que "pida tiempo" para cargar */}
      <Suspense fallback={<p>Cargando el perfil de forma elegante... ⏳</p>}>
        <TarjetaUsuario />
      </Suspense>
    </div>
  );
};
```
**La Magia:** Si mañana decides cambiar el Spinner por un Esqueleto Animado (Skeleton), solo tienes que cambiar el `fallback` del Padre. No tienes que tocar el código del Hijo.

---

## 🛡️ 2. Error Boundaries (El Escudo Protector)

¿Qué pasa si el servidor devuelve un Error 500? En React estándar, un error no capturado durante el renderizado **destruye toda la aplicación**, dejando al usuario con una pantalla en blanco (la Pantalla Blanca de la Muerte).

Un **Error Boundary** (Límite de Error) es un componente especial que actúa como un escudo. Si cualquier componente hijo explota, el escudo absorbe el impacto, evita que la app muera, y muestra una UI de repuesto.

Dado que React aún requiere usar Componentes de Clase (Class Components) para crear Error Boundaries desde cero, la industria ha estandarizado el uso de la librería `react-error-boundary`.

```bash
npm install react-error-boundary
```

### Implementando el Escudo

```jsx
import { ErrorBoundary } from 'react-error-boundary';

const FallbackDeError = ({ error, resetErrorBoundary }) => {
  return (
    <div className="alerta-roja">
      <h2>¡Ups! Algo salió mal.</h2>
      <p>{error.message}</p>
      {/* Botón para reintentar la petición */}
      <button onClick={resetErrorBoundary}>Intentar de nuevo</button>
    </div>
  );
};

export const PanelSeguro = () => {
  return (
    <ErrorBoundary 
      FallbackComponent={FallbackDeError}
      onReset={() => console.log("Limpiando caché y reintentando...")}
    >
      <ComponenteQuePodriaExplotar />
    </ErrorBoundary>
  );
};
```

---

## 🏛️ 3. Nivel Arquitecto: La Hamburguesa Declarativa

El patrón definitivo de la industria moderna consiste en combinar todo lo que hemos aprendido: `<ErrorBoundary>` por fuera, `<Suspense>` en medio, y el Componente que hace el *fetch* en el centro.

```jsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';
import { useQueryErrorResetBoundary } from '@tanstack/react-query';

export const DashboardTopTier = () => {
  // Este hook permite que el botón "Reintentar" del ErrorBoundary 
  // le diga a TanStack Query que vuelva a hacer el fetch automáticamente.
  const { reset } = useQueryErrorResetBoundary();

  return (
    <ErrorBoundary FallbackComponent={FallbackDeError} onReset={reset}>
      <Suspense fallback={<SpinnerPantallaCompleta />}>
        
        {/* Estos componentes usan useSuspenseQuery internamente */}
        <NavegacionPerfil />
        <FeedDeNoticias />
        <Estadisticas />

      </Suspense>
    </ErrorBoundary>
  );
};
```

### ¿Por qué esto te convierte en Arquitecto?
1. **Separación de Responsabilidades:** Tus componentes solo se encargan de pintar datos (UI). No saben qué es un Spinner ni qué es un Error 500.
2. **Coordinación Visual:** `<Suspense>` espera a que *Navegacion*, *Feed* y *Estadisticas* terminen de cargar para mostrar la pantalla de golpe. Evitas el efecto desagradable de componentes apareciendo uno por uno dando saltos en la pantalla.
3. **Resiliencia:** Si la API de *Estadisticas* se cae, el `ErrorBoundary` atrapará el error de esa sección específica, dejando que el resto de la aplicación siga funcionando.

---

### 🎉 ¡Felicidades! Has completado el Módulo 4.
Has pasado de hacer peticiones básicas con `fetch` a dominar la concurrencia, el caché, la sincronización de estados y la arquitectura declarativa. El "Estado del Servidor" ya no tiene secretos para ti.