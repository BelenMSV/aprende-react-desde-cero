# 02 - Componentes Controlados (La Vía React)

En HTML tradicional (Vanilla JS), los elementos de formulario como `<input>`, `<textarea>` y `<select>` mantienen su propio estado interno. Si el usuario escribe "Hola", el input guarda esa palabra en su propia memoria del DOM. 

El problema con esto es que perdemos el control. Si queremos validar si un email es correcto *mientras* el usuario escribe, o si queremos borrar el campo al pulsar un botón, tenemos que ir a buscar ese elemento al DOM y forzar su valor.

En React, abrazamos el patrón de **Componentes Controlados**, que tiene una regla de oro: **El estado de React (`useState`) debe ser la "Única Fuente de la Verdad" (Single Source of Truth).**

---

## 🔗 1. Anatomía de un Input Controlado

Para "controlar" un input, debemos secuestrar su memoria combinando dos atributos:
1. `value`: Obliga al input a mostrar estrictamente lo que dicte el estado de React.
2. `onChange`: Escucha al usuario e informa a React para que actualice el estado.

```jsx
import { useState } from 'react';

export const BuscadorControlado = () => {
  const [texto, setTexto] = useState("");

  const manejarCambio = (e) => {
    // 1. Convertimos todo a mayúsculas antes de guardarlo (Interceptamos)
    const textoModificado = e.target.value.toUpperCase();
    
    // 2. Actualizamos la Fuente de la Verdad
    setTexto(textoModificado);
  };

  return (
    <div>
      {/* 3. El input está atado de manos. Solo puede mostrar 'texto' */}
      <input 
        type="text" 
        value={texto} 
        onChange={manejarCambio} 
      />
      <p>Buscando: {texto}</p>
    </div>
  );
};
```

### 🔒 La Trampa de Solo Lectura (Pregunta de Entrevista)
*¿Qué pasa si pones `value={texto}` pero **olvidas** poner el `onChange`?*
React bloqueará el input. El usuario intentará escribir, pero nada cambiará. 
**¿Por qué?** Porque le dijiste al input que su valor es *exactamente* lo que diga el estado. Si no hay un `onChange` que modifique el estado, el estado es inmutable, y la pantalla queda congelada.

---

## 🛠️ 2. Formularios Grandes: El "Manejador Universal"

Crear 5 estados distintos para 5 inputs (`nombre`, `email`, `password`...) genera código repetitivo. Agrupamos todo en un solo Objeto y usamos el atributo `name` del HTML para crear un **Manejador Universal**.

```jsx
export const FormularioRegistro = () => {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
  });

  const handleChange = (e) => {
    const { name, value } = e.target; 

    setForm(prevForm => ({
      ...prevForm,    // 1. Copiamos los datos anteriores (Inmutabilidad)
      [name]: value   // 2. Sobreescribimos SOLO el campo que coincide con el 'name'
    }));
  };

  return (
    <form>
      {/* El 'name' DEBE coincidir con la clave del estado */}
      <input type="text" name="nombre" value={form.nombre} onChange={handleChange} />
      <input type="email" name="email" value={form.email} onChange={handleChange} />
    </form>
  );
};
```
*(Nota: Este manejador universal falla con los Checkboxes y Selects. Aprenderemos a manejarlos en el Tema 4).*

---

## ⚠️ 3. El Costo Oculto: Re-render por Tecla

El patrón controlado es la forma oficial de React, pero **tiene un costo de rendimiento** que un Senior debe conocer.

Analicemos el flujo de vida:
1. Tecla "A" pulsada -> Se dispara `onChange` -> Se actualiza estado -> **React re-renderiza el componente completo** -> Se dibuja la "A".

Escribir "Hola Mundo" (10 teclas) obliga al componente a recalcularse 10 veces en un segundo. Si tu formulario está en la raíz de una aplicación gigante, congelarás la pantalla.

---

## 🚀 4. Nivel Arquitecto: Mitigando el impacto con "Debounce"

Imagina que tu input es un buscador que hace una petición a una API (ej. buscar películas). Si haces un *fetch* por cada tecla pulsada, saturarás el servidor en 2 segundos.

Para mantener el input controlado pero evitar el colapso, usamos un patrón llamado **Debounce (Rebote)**. Consiste en esperar a que el usuario *deje de escribir* durante un tiempo determinado (ej. 500ms) antes de ejecutar la acción pesada.

Para esto, creamos un Custom Hook espectacular:

```jsx
// src/hooks/useDebounce.js
import { useState, useEffect } from 'react';

export const useDebounce = (valor, retraso) => {
  const [valorDebounced, setValorDebounced] = useState(valor);

  useEffect(() => {
    // Configuramos un temporizador para actualizar el valor debounced
    const timer = setTimeout(() => {
      setValorDebounced(valor);
    }, retraso);

    // CLEANUP: Si el 'valor' cambia ANTES de que termine el retraso,
    // cancelamos el temporizador anterior. ¡Esto es la magia del Debounce!
    return () => {
      clearTimeout(timer);
    };
  }, [valor, retraso]);

  return valorDebounced;
};
```

**Cómo usarlo en la vida real:**

```jsx
import { useState, useEffect } from 'react';
import { useDebounce } from '../hooks/useDebounce';

export const BuscadorPeliculas = () => {
  const [busqueda, setBusqueda] = useState("");
  // Obtenemos un valor que "va con retraso"
  const busquedaDebounced = useDebounce(busqueda, 500); 

  useEffect(() => {
    if (busquedaDebounced) {
      console.log("Haciendo FETCH a la API con:", busquedaDebounced);
      // Aquí harías tu llamada a la base de datos
    }
  }, [busquedaDebounced]); // Solo reacciona al valor con retraso

  return (
    <div>
      <input 
        type="text" 
        value={busqueda} 
        onChange={(e) => setBusqueda(e.target.value)} 
        placeholder="Buscar..."
      />
    </div>
  );
};
```
Con este patrón, el input se siente instantáneo para el usuario (la variable `busqueda` se actualiza por tecla), pero el trabajo pesado (la variable `busquedaDebounced`) solo ocurre cuando el usuario hace una pausa de medio segundo. ¡Magia Senior!