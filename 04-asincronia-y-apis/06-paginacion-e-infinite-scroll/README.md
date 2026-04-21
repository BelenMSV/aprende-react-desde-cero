# 06 - Patrones de Datos: Paginación e Infinite Scroll

Cuando nos enfrentamos a bases de datos masivas, debemos traer la información en "trozos" (Chunks). TanStack Query brilla con luz propia en estos escenarios, proporcionando una Experiencia de Usuario (UX) fluida y sin parpadeos.

En este tema abordaremos los dos estándares de la industria para listas largas.

---

## 📄 1. Paginación Clásica (El problema del Parpadeo)

Imagina una tabla de usuarios. El estado que controla en qué página estamos es un simple `useState(1)`. Ese número de página lo ponemos en el `queryKey` para que TanStack Query sepa que debe hacer una nueva petición cada vez que el usuario cambie de página.

❌ **El Problema de UX (Hard Loading):**
```jsx
const [pagina, setPagina] = useState(1);

const { data, isLoading } = useQuery({
  queryKey: ['usuarios', pagina], // El caché aísla cada página
  queryFn: () => fetchUsuarios(pagina),
});

if (isLoading) return <Spinner />; 
```
Si usas el código de arriba, cada vez que el usuario haga clic en "Siguiente", la variable `pagina` cambia. Como el caché para la nueva página está vacío, `isLoading` se vuelve `true`. **La tabla desaparece bruscamente, se muestra un Spinner, y luego aparece la nueva página.** Esto se siente muy torpe.

✅ **La Solución Senior: `keepPreviousData`**
En React Query v5, usamos una función especial llamada `keepPreviousData` como `placeholderData`. Esto le dice a React: *"Mientras buscas la página 2, NO borres la página 1 de la pantalla. Déjala ahí para que la transición sea suave"*.

```jsx
import { useState } from 'react';
import { useQuery, keepPreviousData } from '@tanstack/react-query';

export const TablaPaginada = () => {
  const [pagina, setPagina] = useState(1);

  const { data, isLoading, isPlaceholderData } = useQuery({
    queryKey: ['proyectos', pagina],
    queryFn: () => fetchProyectos(pagina),
    // 🪄 MAGIA: Mantiene los datos anteriores en pantalla mientras carga los nuevos
    placeholderData: keepPreviousData, 
  });

  if (isLoading) return <p>Cargando primera vez...</p>;

  return (
    <div>
      {/* 1. Mostramos los datos */}
      <ul>
        {data.resultados.map(p => <li key={p.id}>{p.nombre}</li>)}
      </ul>

      {/* 2. Controles de Paginación */}
      <div className="controles">
        <button 
          onClick={() => setPagina(p => p - 1)} 
          disabled={pagina === 1}
        >
          Anterior
        </button>

        <span>Página {pagina}</span>

        <button 
          onClick={() => setPagina(p => p + 1)} 
          // isPlaceholderData es true mientras carga la siguiente página.
          // Lo usamos para deshabilitar el botón y evitar clics dobles.
          disabled={isPlaceholderData || !data.hayMasPaginas}
        >
          Siguiente
        </button>
      </div>
      
      {/* 3. Indicador sutil de carga en segundo plano */}
      {isPlaceholderData && <span> Cargando nueva página...</span>}
    </div>
  );
};
```

---

## ♾️ 2. Scroll Infinito (`useInfiniteQuery`)

La paginación clásica reemplaza los datos antiguos por los nuevos. El **Scroll Infinito**, por el contrario, *añade* los datos nuevos al final de la lista existente.

Para esto, TanStack Query nos da un Hook completamente distinto: `useInfiniteQuery`. Este Hook es más complejo porque el caché ya no es un simple objeto, sino que es un **Array de Páginas**.

### La Anatomía de `useInfiniteQuery`

Necesitamos enseñarle a la librería cómo descubrir cuál es la "siguiente página" leyendo la respuesta de nuestra API.

```jsx
import { useInfiniteQuery } from '@tanstack/react-query';

export const FeedRedSocial = () => {
  const {
    data,
    fetchNextPage, // Función para pedir más datos
    hasNextPage,   // Booleano: ¿Hay más datos disponibles?
    isFetchingNextPage, // Booleano: ¿Está cargando el trozo extra?
    status,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    // pageParam es inyectado automáticamente por la librería
    queryFn: ({ pageParam }) => fetchPostsDesdeAPI(pageParam), 
    initialPageParam: 1, // Empezamos en la página 1
    
    // Aquí le enseñamos a calcular el número de la siguiente página.
    // 'lastPage' es la respuesta cruda de tu API (ej. { posts: [...], nextPage: 2 })
    getNextPageParam: (lastPage) => {
      // Si la API devuelve un null o undefined, hasNextPage se volverá 'false'
      return lastPage.nextPage; 
    },
  });

  if (status === 'pending') return <p>Cargando feed inicial...</p>;
  if (status === 'error') return <p>Error al cargar el feed.</p>;

  return (
    <div>
      {/* 1. data.pages es un Array que contiene Arrays. Usamos un doble map() */}
      {data.pages.map((paginaAPI, indice) => (
        <div key={indice}> {/* Usar el índice aquí es seguro porque las páginas no se reordenan */}
          {paginaAPI.posts.map(post => (
            <article key={post.id}>
              <h4>{post.titulo}</h4>
              <p>{post.contenido}</p>
            </article>
          ))}
        </div>
      ))}

      {/* 2. Botón manual para cargar más */}
      <button 
        onClick={() => fetchNextPage()} 
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Cargando más...'
          : hasNextPage
          ? 'Cargar Más Posts'
          : 'No hay más publicaciones'}
      </button>
    </div>
  );
};
```

---

## 🚀 3. Nivel Arquitecto: Automatizando el Scroll Infinito

Tener un botón de "Cargar Más" arruina la ilusión del scroll infinito. Queremos que la función `fetchNextPage()` se dispare automáticamente cuando el usuario llegue al final de la pantalla.

En lugar de pelear con el evento `onScroll` del navegador (que causa problemas de rendimiento horribles), los Seniors usan una API nativa llamada **Intersection Observer**. 

La forma más fácil de implementarlo en React es instalando la micro-librería `react-intersection-observer`:
```bash
npm install react-intersection-observer
```

### El Componente Invisible (El Gatillo)

La estrategia consiste en colocar un `<div>` invisible justo al final de nuestra lista de posts. Cuando ese `<div>` entra en la pantalla, disparamos el fetch.

```jsx
import { useEffect } from 'react';
import { useInView } from 'react-intersection-observer';
import { useInfiniteQuery } from '@tanstack/react-query';

export const FeedAutomatico = () => {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({ /* ... configuración anterior ... */ });
  
  // 1. Configuramos el observador
  const { ref, inView } = useInView({
    threshold: 0.5, // Se activa cuando el elemento es 50% visible
  });

  // 2. Si el gatillo es visible y hay más páginas, pedimos más
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div className="feed-contenedor">
      {/* ... renderizamos los posts ... */}

      {/* 3. EL GATILLO INVISIBLE */}
      <div ref={ref} className="gatillo-scroll" style={{ height: '20px' }}>
        {isFetchingNextPage ? 'Cargando más contenido...' : ''}
      </div>
    </div>
  );
};
```
¡Boom! Tienes un feed de TikTok/Instagram construido con calidad empresarial. Sin bugs, con caché automático, y extremadamente rápido.