# 01 - Eventos Sintéticos en React

En HTML tradicional, para escuchar un clic usarías el atributo `onclick`. En React, usamos `onClick` (con *camelCase*). Aunque se ven casi iguales, bajo el capó **no tienen nada que ver**.

React no le entrega tus eventos directamente al navegador. En su lugar, utiliza un sistema avanzado llamado **Eventos Sintéticos (SyntheticEvents)** y una técnica de rendimiento brutal llamada **Delegación de Eventos**.

---

## 🎭 1. ¿Qué es un Evento Sintético?

Cada navegador (Chrome, Safari, Firefox) tiene sus propias rarezas al manejar eventos. Para evitar que tengas que escribir código distinto para cada navegador, React intercepta el evento real y lo envuelve en un "Evento Sintético".

Este evento de React es un clon perfecto que:
1. **Funciona exactamente igual en todos los navegadores.**
2. Cumple con los estándares modernos de la W3C.

### La magia de la Delegación de Eventos (Pregunta de Entrevista Senior)
Si tienes una lista con 1,000 botones y a cada uno le pones un `onClick`, un desarrollador Junior pensaría que React crea 1,000 "escuchadores" (*event listeners*) en el navegador. 

**Falso.** React crea **un solo escuchador gigante** en la raíz de tu aplicación (`<div id="root">`). Cuando haces clic en un botón, el evento viaja hasta la raíz, React deduce qué botón tocaste y ejecuta tu función. Esto hace que React sea extremadamente rápido y consuma muy poca memoria.

---

## ❌ 2. El Anti-patrón de la Ejecución Inmediata

Este es el error número uno de los principiantes.

❌ **El instinto del Junior (Ejecución inmediata):**
```jsx
export const BotonPeligroso = () => {
  const borrarDatos = () => console.log("Datos borrados");

  // 🚩 ERROR: Los paréntesis () hacen que la función se ejecute 
  // EN EL MOMENTO en que React dibuja la pantalla, ¡sin esperar al clic!
  return <button onClick={borrarDatos()}>Borrar todo</button>;
};
```

✅ **La forma correcta (Pasar la referencia):**
Debes pasar el *nombre* de la función, sin paréntesis. Le estás diciendo a React: *"Aquí tienes las instrucciones, ejecútalas **solo** cuando el usuario haga clic"*.
```jsx
// ✅ CORRECTO: Pasamos la referencia a la función
<button onClick={borrarDatos}>Borrar todo</button>
```

### ¿Y si necesito pasar un parámetro?
Si necesitas pasar un ID, envuélvelo en una función de flecha anónima:
```jsx
// ✅ CORRECTO: React ejecutará la función anónima al hacer clic
<button onClick={() => borrarDatos(7)}>Borrar usuario 7</button>
```

---

## 🧰 3. El Objeto `e` (Event)

Cuando React ejecuta tu manejador de eventos, automáticamente le inyecta un objeto con toda la información de lo que acaba de ocurrir. Por convención, lo llamamos `e` o `event`.

```jsx
export const InputEspia = () => {
  const manejarCambio = (e) => {
    // e.target es el <input> exacto que disparó el evento
    console.log("El usuario escribió:", e.target.value); 
  };

  return <input type="text" onChange={manejarCambio} />;
};
```

---

## 🛡️ 4. `e.preventDefault()` (El salvavidas de los Formularios)

Por defecto, cuando presionas "Enter" o haces clic en un botón dentro de un `<form>`, el navegador **recarga toda la página** intentando enviar los datos a un servidor.

En React, recargar la página destruye todo tu estado y tu memoria de JavaScript. Para evitarlo, usamos `preventDefault()`.

```jsx
export const FormularioSeguro = () => {
  const manejarEnvio = (e) => {
    e.preventDefault(); // 🛑 ¡Detiene la recarga de la página!
    console.log("Procesando datos con React...");
  };

  return (
    <form onSubmit={manejarEnvio}>
      <input type="text" />
      <button type="submit">Enviar</button>
    </form>
  );
};
```

---

## 🌀 5. `e.stopPropagation()` (La Burbuja de Eventos)

En HTML, si haces clic en un elemento que está dentro de otro, el clic "burbujea" hacia arriba. Imagina una Tarjeta que tiene un botón de "Borrar" dentro. Si haces clic en el botón, también estás haciendo clic en la Tarjeta.

❌ **El Problema (Doble disparo):**
```jsx
<div onClick={abrirTarjeta} className="tarjeta">
  {/* 🚩 Al hacer clic aquí, se ejecuta borrar() Y TAMBIÉN abrirTarjeta() */}
  <button onClick={borrar}>Borrar</button> 
</div>
```

✅ **La Solución (Detener la burbuja):**
Usamos `e.stopPropagation()` para decirle a React: *"Atiende este evento y no dejes que suba a mis padres"*.

```jsx
const borrar = (e) => {
  e.stopPropagation(); // 🛑 La burbuja explota aquí. El div padre no se entera.
  console.log("Elemento borrado");
};
```

---

## 🚀 6. Nivel Arquitecto: La Fase de Captura (`onClickCapture`)

Hemos visto que los eventos burbujean de abajo hacia arriba (Hijo -> Padre -> Abuelo). 

Pero la realidad técnica es que el navegador primero lanza el evento **de arriba hacia abajo** (Fase de Captura) antes de que empiece a burbujear. React te permite escuchar esta fase añadiendo la palabra `Capture` al final del evento (ej. `onClickCapture`).

Esto es útil en analíticas o modales cuando quieres que el Padre intercepte un clic *antes* de que el Hijo se entere:

```jsx
<div onClickCapture={() => console.log("1. El padre se entera primero")}>
  <button onClick={() => console.log("2. El hijo se entera después")}>
    Clic aquí
  </button>
</div>
```

---

## 🦖 7. Pro Tip: El fin del "Event Pooling" (React 17+)

Si buscas respuestas en *StackOverflow* sobre eventos asíncronos en React, verás mucho código antiguo que usa `e.persist()`. 

En React 16 y anteriores, React "reciclaba" el objeto `e` inmediatamente después de ejecutar tu función para ahorrar memoria (Event Pooling). Si intentabas leer `e.target.value` dentro de un `setTimeout` o de una Promesa, te daba error porque el objeto `e` ya había sido borrado. Para evitarlo, se usaba `e.persist()`.

**A partir de React 17, el Event Pooling fue eliminado** porque las computadoras modernas tienen memoria de sobra y causaba mucha confusión. Hoy en día, **puedes leer el objeto `e` dentro de funciones asíncronas sin problemas** y `e.persist()` ya no hace nada. ¡No lo uses!