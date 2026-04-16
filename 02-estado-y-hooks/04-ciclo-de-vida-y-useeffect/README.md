# 04 - Ciclo de Vida y Efectos Secundarios (`useEffect`)

La regla principal de React es que **los componentes deben ser funciones puras**. Esto significa que su único trabajo es recibir datos y devolver JSX. No deben "tocar" nada fuera de sí mismos durante el proceso de renderizado.

Sin embargo, las aplicaciones reales necesitan interactuar con el mundo exterior:
- Hacer peticiones a una API (Fetch).
- Modificar el DOM directamente (ej. cambiar el `<title>`).
- Crear temporizadores (`setTimeout`).
- Escuchar eventos globales del navegador (ej. el scroll de la ventana).

A todas estas acciones que ocurren "fuera de la burbuja" de React se les llama **Efectos Secundarios (Side Effects)**.

---

## ⏳ El Ciclo de Vida de un Componente

Para entender cuándo ejecutar un efecto secundario, primero debes entender la vida de un componente:

1. **Montaje (Mount):** El componente nace. Es la primera vez que se renderiza y se inserta en la pantalla.
2. **Actualización (Update):** El componente crece. Su estado o sus props cambian, lo que provoca que se vuelva a renderizar.
3. **Desmontaje (Unmount):** El componente muere. Es destruido y eliminado de la pantalla.

El Hook `useEffect` unifica todo el ciclo de vida en una sola herramienta, diciéndole a React: *"Después de pintar la pantalla, ejecuta este código"*.

---

## ⚡ Introducción a `useEffect`

```jsx
import { useEffect, useState } from 'react';

export const SincronizadorDeTitulo = () => {
  const [contador, setContador] = useState(0);

  // useEffect se ejecutará SIEMPRE DESPUÉS de que el componente se renderice.
  useEffect(() => {
    document.title = `Hiciste clic ${contador} veces`;
  });

  return (
    <button onClick={() => setContador(prev => prev + 1)}>
      Clics: {contador}
    </button>
  );
};
```

---

## ☠️ El peligro de los Bucles Infinitos

Un error clásico es actualizar el estado **dentro** de un `useEffect` sin controlarlo.

❌ **El Código de la Muerte:**
```jsx
export const ComponentePeligroso = () => {
  const [datos, setDatos] = useState([]);

  useEffect(() => {
    // 1. El efecto se ejecuta y actualiza el estado
    setDatos(["Dato 1", "Dato 2"]); 
  }); 

  return <div>Renderizando...</div>;
};
```
**¿Por qué este código destruye tu navegador?**
React renderiza ➔ Ejecuta el efecto ➔ El efecto cambia el estado ➔ El cambio de estado provoca un nuevo renderizado ➔ Se vuelve a ejecutar el efecto ➔ **Bucle Infinito**.

Para evitar esto, usamos el **Arreglo de Dependencias** (que dominaremos en el próximo tema).

---

## 📐 Arquitectura Limpia: Un Efecto = Una Responsabilidad

Cuando los Juniors descubren `useEffect`, tienden a crear un solo bloque gigante donde meten todas las acciones de la aplicación.

❌ **El "Mega-Efecto" (Difícil de mantener):**
```jsx
useEffect(() => {
  fetchDatosDelUsuario();
  document.title = "Perfil";
  window.addEventListener('resize', manejarResize);
});
```

✅ **El enfoque Profesional (Separación de Responsabilidades):**
Puedes (y debes) usar múltiples `useEffect` en un mismo componente. Si una parte de la lógica cambia, no quieres que afecte al resto.

```jsx
// Efecto 1: Solo se encarga de la API
useEffect(() => {
  fetchDatosDelUsuario();
});

// Efecto 2: Solo se encarga del DOM
useEffect(() => {
  document.title = "Perfil";
});

// Efecto 3: Solo se encarga de los Eventos
useEffect(() => {
  window.addEventListener('resize', manejarResize);
});
```

---

## 📦 El dilema de las Funciones Externas

Si vas a hacer un *fetch* de datos, es común crear una función para ello. ¿Dónde la pones?

❌ **Fuera del efecto (Causa advertencias del Linter):**
```jsx
const cargarUsuarios = async () => { /* ... */ };

useEffect(() => {
  cargarUsuarios(); 
  // React se quejará: "cargarUsuarios es una dependencia faltante"
});
```

✅ **Dentro del efecto (La práctica recomendada):**
Si una función solo se usa dentro de un `useEffect`, declárala dentro. Esto garantiza que la función tenga acceso a las variables locales actualizadas y evita bugs de dependencias.

```jsx
useEffect(() => {
  // Declaramos la función aquí dentro
  const cargarUsuarios = async () => {
    const respuesta = await fetch('/api/usuarios');
    // ...
  };

  // Y la ejecutamos
  cargarUsuarios();
});
```

---

## 🏃‍♂️ El Mundo Real: Condiciones de Carrera (Race Conditions)

Este es el bug más avanzado y común al usar `useEffect` para pedir datos a una API.

Imagina que tienes un perfil donde puedes ver los datos del Usuario 1 o del Usuario 2 haciendo clic en un botón.
1. Haces clic para ver el **Usuario 1**. React inicia el `fetch`. Pero la red está muy lenta y tarda **3 segundos**.
2. Te desesperas y rápidamente haces clic para ver el **Usuario 2**. La red responde rápido y tarda solo **1 segundo**.
3. La pantalla muestra los datos del Usuario 2 correctamente.
4. Dos segundos después, la petición del Usuario 1 finalmente termina y... **¡Sobreescribe los datos en la pantalla!** Terminas viendo la URL del "Usuario 2", pero con los datos del "Usuario 1". A esto se le llama **Condición de Carrera**.

En el Tema 5 aprenderemos a resolver esto y evitar fugas de memoria usando el superpoder final de los efectos: **La Función de Limpieza (Cleanup Function).**

---

## 🐛 "¿Por qué mi useEffect se ejecuta dos veces al inicio?"

Si pones un `console.log` dentro de tu efecto, verás que se imprime dos veces cuando recargas la página en desarrollo.

React envuelve tu aplicación en `<React.StrictMode>`. Este modo monta tu componente, lo desmonta inmediatamente y lo vuelve a montar a propósito. Lo hace para ayudarte a descubrir si tus efectos secundarios tienen fallos de limpieza (Memory Leaks). **En producción, solo se ejecutará una vez.**