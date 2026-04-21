# 03 - Componentes No Controlados y `useRef`

En el tema anterior vimos que los **Componentes Controlados** son poderosos porque la variable de estado (`useState`) siempre tiene la última palabra. Sin embargo, su mayor debilidad es el rendimiento: cada pulsación de tecla fuerza a React a re-renderizar todo el componente.

A veces, simplemente no necesitamos saber qué está escribiendo el usuario en tiempo real. Si solo nos importa el dato final (cuando el usuario hace clic en "Enviar"), podemos usar **Componentes No Controlados**.

En este patrón, **el DOM recupera su libertad**. El `<input>` guarda su propia memoria internamente, y React solo lee esa memoria en el momento exacto en que la necesita.

---

## 🏗️ 1. La Vía Tradicional: Leyendo el DOM con `useRef`

Para leer el valor de un input sin usar `onChange`, utilizamos nuestro viejo amigo del Módulo 2: el hook `useRef`.

Creamos una referencia, la "enganchamos" al input, y cuando el formulario se envía, leemos la propiedad `current.value`.

```jsx
import { useRef } from 'react';

export const FormularioNoControlado = () => {
  // 1. Creamos referencias para cada input
  const nombreRef = useRef(null);
  const emailRef = useRef(null);

  const manejarEnvio = (e) => {
    e.preventDefault(); // 🛑 Siempre evitamos la recarga de la página

    // 3. Leemos los datos directamente del DOM SÓLO al enviar
    const datosUsuario = {
      nombre: nombreRef.current.value,
      email: emailRef.current.value,
    };

    console.log("Enviando a la API:", datosUsuario);
  };

  return (
    <form onSubmit={manejarEnvio}>
      {/* 2. Pasamos la ref. NO usamos 'value' ni 'onChange' */}
      <input type="text" ref={nombreRef} placeholder="Tu Nombre" />
      <input type="email" ref={emailRef} placeholder="Tu Email" />
      
      <button type="submit">Registrarse</button>
    </form>
  );
};
```

### 🔒 `defaultValue`: Inicializando inputs libres
Si intentas usar la prop `value="Hola"` en un input sin un `onChange`, React lo bloqueará (como vimos en el tema 2). 
En los componentes no controlados, si quieres que un input empiece con un texto por defecto, **debes usar la prop `defaultValue`**.

```jsx
// El usuario podrá borrar "Juan" y escribir otra cosa libremente
<input type="text" ref={nombreRef} defaultValue="Juan" />
```

---

## 🚀 2. Nivel Arquitecto: El Patrón `FormData`

Usar `useRef` está bien para 2 o 3 inputs. Pero, ¿qué pasa si tienes un formulario gigante con 20 campos? Crear 20 referencias con `useRef` y leerlas una por una ensucia muchísimo el código.

Los verdaderos desarrolladores Senior saben que no siempre necesitan a React para todo. **HTML5 y JavaScript nativo tienen una API espectacular llamada `FormData`.**

`FormData` lee automáticamente TODOS los inputs que estén dentro de una etiqueta `<form>`, siempre y cuando tengan el atributo `name`. ¡No necesitas ni un solo `useRef`!

✅ **El Patrón de Rendimiento Máximo:**

```jsx
export const FormularioSenior = () => {
  const manejarEnvio = (e) => {
    e.preventDefault();

    // 1. e.target es el elemento <form> entero.
    // 2. FormData extrae todos los datos de sus inputs hijos.
    const formulario = new FormData(e.target);

    // 3. Object.fromEntries convierte el FormData en un Objeto limpio
    const datos = Object.fromEntries(formulario);

    console.log("Datos listos para enviar:", datos);
    // Resultado: { nombre: "Ana", edad: "25", pais: "ES" }

    // Opcional: Limpiar el formulario visualmente tras enviarlo
    e.target.reset(); 
  };

  return (
    <form onSubmit={manejarEnvio}>
      {/* ⚠️ El atributo 'name' es OBLIGATORIO para que FormData lo encuentre */}
      <input type="text" name="nombre" placeholder="Nombre" />
      <input type="number" name="edad" placeholder="Edad" />
      
      <select name="pais">
        <option value="ES">España</option>
        <option value="MX">México</option>
        <option value="CO">Colombia</option>
      </select>

      <button type="submit">Enviar al Servidor</button>
    </form>
  );
};
```
*(Cero re-renders, cero `useStates`, cero `useRefs`. Rendimiento puro y código limpio).*

---

## ⚖️ 3. Controlado vs. No Controlado: ¿Cuál elegir?

Como Arquitecto, tu trabajo no es saber una sola herramienta, sino saber **cuándo usar cuál**.

| Característica | Controlado (`useState`) | No Controlado (`FormData` / `useRef`) |
| :--- | :--- | :--- |
| **Cuándo lee los datos** | En cada pulsación de tecla | Solo al hacer "Submit" |
| **Rendimiento** | Lento (Muchos renders) | Rapidísimo (Cero renders) |
| **Validación instantánea**| ✅ Posible (Ej: "La contraseña es muy corta" al vuelo) | ❌ Imposible (Solo valida al final) |
| **Deshabilitar botón** | ✅ Posible (Ej: Botón gris si falta el email) | ❌ Imposible |
| **Forzar formato** | ✅ Posible (Ej: Poner todo en Mayúsculas automáticamente)| ❌ Imposible |

**🏆 Regla de Oro:**
* Usa **No Controlado (FormData)** si el formulario es un simple recolector de datos (ej. Iniciar sesión o un formulario de Contacto estándar).
* Usa **Controlado** si necesitas una UI interactiva que reaccione al instante (ej. Un buscador dinámico, formatear una tarjeta de crédito con espacios, o validación compleja).
* En la vida real, usaremos librerías como **React Hook Form** que hacen magia negra para combinar lo mejor de ambos mundos (las veremos en el Tema 6).