# 04 - Inputs Complejos: Selects, Checkboxes y Archivos

Trabajar con inputs de texto es la parte fácil de los formularios. Las verdaderas complicaciones surgen cuando necesitas capturar arreglos de datos (múltiples checkboxes), valores booleanos (términos y condiciones) o archivos binarios (imágenes).

En este tema aprenderemos cómo domar los elementos más rebeldes de HTML usando el patrón de Componentes Controlados y las excepciones donde React nos obliga a usar No Controlados.

---

## 🔽 1. El elemento `<select>`

En HTML tradicional, para marcar una opción por defecto en un `<select>`, tendrías que poner el atributo `selected` dentro de la etiqueta `<option>`. 

En React, esto cambia para hacerte la vida más fácil: **El estado se controla directamente en la etiqueta padre `<select>`** usando el atributo `value`.

### Select Simple
```jsx
import { useState } from 'react';

export const SelectorPais = () => {
  const [pais, setPais] = useState('mx'); // Selecciona 'mx' por defecto

  return (
    <select value={pais} onChange={(e) => setPais(e.target.value)}>
      <option value="es">España</option>
      <option value="mx">México</option>
      <option value="ar">Argentina</option>
    </select>
  );
};
```

### Select Múltiple (Reto Senior)
Si añades el atributo `multiple`, el usuario puede elegir varias opciones a la vez usando `Ctrl` o `Cmd`. Aquí `e.target.value` falla, porque solo devuelve el primer elemento. Debemos iterar sobre las opciones seleccionadas y guardarlas en un Array.

```jsx
export const SelectorMultiple = () => {
  const [etiquetas, setEtiquetas] = useState(['react']); // Estado inicial: Array

  const manejarCambio = (e) => {
    // e.target.selectedOptions es una colección HTML, la convertimos a Array
    const opcionesSeleccionadas = Array.from(e.target.selectedOptions);
    const valores = opcionesSeleccionadas.map(opcion => opcion.value);
    
    setEtiquetas(valores);
  };

  return (
    <select multiple value={etiquetas} onChange={manejarCambio}>
      <option value="react">React</option>
      <option value="node">Node.js</option>
      <option value="css">CSS</option>
    </select>
  );
};
```

---

## ✅ 2. Checkboxes y la trampa del `value`

Los `<input type="checkbox">` no se controlan con el atributo `value`, sino con el atributo `checked` (que recibe un booleano `true`/`false`). Al escuchar el evento, en lugar de leer `e.target.value`, leemos **`e.target.checked`**.

### Checkbox Simple (Booleano)
Ideal para "Aceptar términos y condiciones" o "Suscribirse al boletín".

```jsx
const [terminosAceptados, setTerminosAceptados] = useState(false);

<input 
  type="checkbox" 
  checked={terminosAceptados} 
  onChange={(e) => setTerminosAceptados(e.target.checked)} 
/>
```

### Grupo de Checkboxes (Array de Valores) - Patrón Profesional
Este es un requerimiento clásico: tienes una lista de categorías ("Deportes", "Música", "Cine") y el usuario puede marcar varias. El estado debe ser un Array de *strings*.

```jsx
export const Preferencias = () => {
  const [intereses, setIntereses] = useState([]); // Array vacío

  const manejarCheckbox = (e) => {
    const { value, checked } = e.target;

    if (checked) {
      // Si el usuario lo marcó, lo AÑADIMOS al array
      setIntereses(prev => [...prev, value]);
    } else {
      // Si el usuario lo desmarcó, lo QUITAMOS del array
      setIntereses(prev => prev.filter(interes => interes !== value));
    }
  };

  return (
    <div>
      <label>
        <input 
          type="checkbox" 
          value="deportes" 
          // React marcará la casilla si el valor está en el array
          checked={intereses.includes("deportes")} 
          onChange={manejarCheckbox} 
        /> Deportes
      </label>
      
      <label>
        <input 
          type="checkbox" 
          value="musica" 
          checked={intereses.includes("musica")} 
          onChange={manejarCheckbox} 
        /> Música
      </label>
    </div>
  );
};
```

---

## 🔘 3. Botones de Radio (Radio Buttons)

Los botones de radio funcionan agrupándolos por el atributo `name`. A diferencia de los checkboxes múltiples, un grupo de radios solo puede tener **un valor activo a la vez**, por lo que su estado es un simple *String*.

Para controlarlos en React, comparamos el estado actual con el `value` de cada input.

```jsx
export const GeneroForm = () => {
  const [genero, setGenero] = useState("femenino");

  return (
    <div>
      <label>
        <input 
          type="radio" 
          value="femenino" 
          // Devuelve true si el estado es exactamente "femenino"
          checked={genero === "femenino"} 
          onChange={(e) => setGenero(e.target.value)} 
        /> Femenino
      </label>

      <label>
        <input 
          type="radio" 
          value="masculino" 
          checked={genero === "masculino"} 
          onChange={(e) => setGenero(e.target.value)} 
        /> Masculino
      </label>
    </div>
  );
};
```

---

## 📂 4. Subida de Archivos (La Excepción a la Regla)

**Los `<input type="file">` NO pueden ser Componentes Controlados.**

Por razones de seguridad del navegador, JavaScript no tiene permiso para decirle a un input de archivos qué archivo debe tener seleccionado. Su valor es de "Solo Lectura" para React.

Por lo tanto, **siempre debemos usar la técnica de Componentes No Controlados** (vistos en el Tema 3) para leer los archivos:

```jsx
import { useRef } from 'react';

export const SubirAvatar = () => {
  const archivoRef = useRef(null);

  const enviarFormulario = (e) => {
    e.preventDefault();
    
    // .files es una colección (tipo Array) de archivos seleccionados
    const archivoSeleccionado = archivoRef.current.files[0];

    if (!archivoSeleccionado) {
      alert("Por favor selecciona un archivo.");
      return;
    }

    console.log("Nombre del archivo:", archivoSeleccionado.name);
    console.log("Tamaño:", archivoSeleccionado.size, "bytes");

    // Para enviarlo al servidor, usualmente lo metemos en un FormData
    const formData = new FormData();
    formData.append("avatar", archivoSeleccionado);
    // fetch('/api/upload', { method: 'POST', body: formData });
  };

  return (
    <form onSubmit={enviarFormulario}>
      {/* ⚠️ Nota: No tiene 'value' ni 'onChange' */}
      <input type="file" accept="image/png, image/jpeg" ref={archivoRef} />
      <button type="submit">Subir Imagen</button>
    </form>
  );
};
```

---

## 🧠 5. El "Manejador Universal" Definitivo

En el Tema 2 creamos un "Manejador Universal" elegante para inputs de texto. Ahora que sabemos que los checkboxes usan `checked` en lugar de `value`, debemos actualizar nuestro manejador para que soporte CUALQUIER tipo de input nativo.

```jsx
const handleChange = (e) => {
  // Obtenemos type, name, value y checked del input que disparó el evento
  const { type, name, value, checked } = e.target;

  setForm(prevForm => ({
    ...prevForm,
    // Si es un checkbox booleano, guardamos 'checked'. 
    // Para todos los demás (texto, select, radio), guardamos 'value'.
    [name]: type === 'checkbox' ? checked : value
  }));
};
```
*(Este es el estándar que verás en bases de código limpias antes de la llegada de librerías avanzadas).*