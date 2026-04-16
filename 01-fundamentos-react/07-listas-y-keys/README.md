# 07 - Renderizado de Listas y la prop `key`

En la vida real, rara vez escribirás componentes uno por uno de forma manual. Lo más normal es que recibas una lista de datos (un *Array* de objetos) desde una base de datos o una API, y necesites transformar cada uno de esos datos en un componente visual.

Como en JSX no podemos usar bucles tradicionales como `for` o `while` directamente dentro del marcado, utilizamos los métodos nativos de los Arrays de JavaScript. El rey indiscutible aquí es **`.map()`**.

---

## 🔄 Transformando Datos en Interfaz con `.map()`

El método `.map()` toma un Array original, le aplica una función a cada elemento, y devuelve un **nuevo Array**. 

En React, usamos `.map()` para tomar un Array de datos (ej. usuarios) y devolver un Array de JSX (ej. componentes `<TarjetaUsuario />`). ¡Y a React le encanta renderizar Arrays de JSX!

```jsx
export const ListaUsuarios = () => {
  const usuarios = ["Ana", "Carlos", "Beatriz", "David"];

  return (
    <ul>
      {usuarios.map((nombre) => {
        return <li>{nombre}</li>;
      })}
    </ul>
  );
};
```

*(Pro Tip: Podemos usar el retorno implícito de las Arrow Functions para que el código quede mucho más limpio sustituyendo las llaves `{}` por paréntesis `()` y omitiendo la palabra `return`).*

```jsx
// Versión limpia con retorno implícito
{usuarios.map((nombre) => (
  <li>{nombre}</li>
))}
```

---

## 🔑 El Misterio de la prop `key`

Si ejecutas el código anterior y abres la consola de tu navegador, verás un error rojo gigante que dice:
> 💥 *Warning: Each child in a list should have a unique "key" prop.*

**¿Por qué React se queja?**
Recuerda el Tema 2 (El Virtual DOM). Cuando cambia el estado de tu aplicación, React compara el Virtual DOM antiguo con el nuevo para saber qué debe actualizar en la pantalla.

Si tienes una lista de 1000 elementos y decides borrar el elemento que está en la posición 2, ¿cómo sabe React exactamente qué elemento de la pantalla tiene que destruir sin tener que volver a dibujar los 999 restantes? **Gracias a la `key`.**

La prop `key` es un identificador único que le damos a React para que pueda rastrear cada elemento de la lista individualmente a través del tiempo.

✅ **La forma correcta:**
Debes usar un identificador único real que venga de tu base de datos (como un `id`, un `uuid`, o un `slug`).

```jsx
export const ListaTareas = () => {
  const tareas = [
    { id: 't-01', texto: 'Comprar leche' },
    { id: 't-02', texto: 'Aprender React' },
    { id: 't-03', texto: 'Pasear al perro' }
  ];

  return (
    <ul>
      {tareas.map((tarea) => (
        // La key va SIEMPRE en el elemento raíz que devuelve el map
        <li key={tarea.id}>
          {tarea.texto}
        </li>
      ))}
    </ul>
  );
};
```

---

## ⚠️ El peligro de usar el `index` (Pregunta de Entrevista)

Muchos desarrolladores Junior, para silenciar el error rojo de la consola, utilizan el índice del array (la posición 0, 1, 2...) como `key`.

❌ **Mala práctica:**
```jsx
{tareas.map((tarea, index) => (
  <li key={index}>{tarea.texto}</li> // 🚩 ¡Peligroso!
))}
```

**¿Por qué es una mala idea?**
El índice no está atado al *dato*, está atado a la *posición*. 

Imagina que tienes tres tareas ordenadas:
1. (Index 0) - Tarea A
2. (Index 1) - Tarea B
3. (Index 2) - Tarea C

Si el usuario elimina la "Tarea A", la "Tarea B" sube a la primera posición y pasa a tener el Index 0. React verá que el elemento con `key={0}` sigue existiendo, así que en lugar de destruir la Tarea A, reciclará el componente de forma incorrecta. Esto causa **bugs visuales gravísimos**: inputs que mantienen el texto del elemento borrado, animaciones que se rompen o estados que se mezclan.

**Regla de oro:** Solo usa el `index` como `key` si la lista es **100% estática**, es decir, si los elementos nunca se van a reordenar, añadir, ni eliminar.

---

## 🧼 Pro Tips: Escribiendo Listas Limpias

Cuando la lógica dentro de tu `.map()` empieza a crecer, el archivo se vuelve difícil de leer. Aplica estos patrones de Clean Code:

### 1. Desestructuración dentro de los parámetros del map
Si estás iterando sobre un Array de objetos, no repitas `tarea.id`, `tarea.titulo`, `tarea.fecha`. Desestructura directamente en la función:

```jsx
// ❌ Repetitivo
{tareas.map((tarea) => (
  <li key={tarea.id}>
    <h3>{tarea.titulo}</h3>
    <p>{tarea.descripcion}</p>
  </li>
))}

// ✅ Profesional y Limpio
{tareas.map(({ id, titulo, descripcion }) => (
  <li key={id}>
    <h3>{titulo}</h3>
    <p>{descripcion}</p>
  </li>
))}
```

### 2. Extraer el elemento de la lista a su propio Componente
Si el elemento que vas a renderizar tiene más de 4 o 5 líneas de JSX, o si tiene su propia lógica (como un botón para expandir detalles), sácalo a un componente independiente.

```jsx
// TareaItem.jsx (Componente independiente)
export const TareaItem = ({ titulo, descripcion }) => (
  <article className="card-tarea">
    <h3>{titulo}</h3>
    <p>{descripcion}</p>
    <button>Completar</button>
  </article>
);

// ListaTareas.jsx
import { TareaItem } from './TareaItem';

export const ListaTareas = ({ tareas }) => {
  return (
    <section>
      {tareas.map((tarea) => (
        // La key sigue yendo aquí, en la llamada al componente
        <TareaItem 
          key={tarea.id} 
          titulo={tarea.titulo} 
          descripcion={tarea.descripcion} 
        />
        
        // Alternativa usando el operador spread si las props coinciden exactamente con el objeto:
        // <TareaItem key={tarea.id} {...tarea} />
      ))}
    </section>
  );
};
```

---

## 🌍 Escenarios del Mundo Real

En las aplicaciones profesionales, no basta con renderizar la lista. Tienes que pensar en la Experiencia de Usuario (UX) y en la manipulación de los datos antes de mostrarlos.

### 1. Estados Vacíos (Empty States)

¿Qué pasa si la base de datos nos devuelve un Array vacío (`[]`)? El `.map()` simplemente no se ejecutará y el usuario verá un hueco en blanco en la pantalla. Esto genera confusión ("¿Está cargando? ¿Se rompió?").

Siempre debemos combinar el renderizado de listas con el renderizado condicional para crear "Empty States".

```jsx
export const ListaUsuarios = ({ usuarios }) => {
  // Patrón de Early Return para el estado vacío
  if (usuarios.length === 0) {
    return (
      <div className="empty-state">
        <img src="gato-triste.png" alt="Sin resultados" />
        <h3>No se encontraron usuarios</h3>
        <p>Intenta ajustar tu búsqueda o crea un usuario nuevo.</p>
      </div>
    );
  }

  // Si hay usuarios, renderizamos la lista normal
  return (
    <ul>
      {usuarios.map((usuario) => (
        <li key={usuario.id}>{usuario.nombre}</li>
      ))}
    </ul>
  );
};
```

### 2. El Combo Ganador: `.filter()` y `.map()`

Muchas veces no queremos mostrar *toda* la lista, sino solo una parte de ella (por ejemplo, un buscador, o pestañas de "Pendientes" y "Completadas").

Como `.map()` y `.filter()` devuelven nuevos Arrays, **podemos encadenarlos** directamente en nuestro JSX.

```jsx
export const PanelTareas = ({ tareas }) => {
  return (
    <section>
      <h2>Tareas Pendientes (Urgentes)</h2>
      <ul>
        {tareas
          // 1. Primero filtramos: nos quedamos solo con las que NO están completadas
          .filter((tarea) => tarea.completada === false)
          // 2. Luego mapeamos ese nuevo array filtrado
          .map((tarea) => (
            <li key={tarea.id}>{tarea.texto}</li>
          ))
        }
      </ul>
    </section>
  );
};
```
*💡 Nota: Si la lista es gigantesca (miles de elementos), encadenar métodos dentro del `return` puede afectar al rendimiento en cada re-renderizado. En esos casos, aprenderemos a memorizar cálculos complejos con `useMemo` en el Módulo 7.*