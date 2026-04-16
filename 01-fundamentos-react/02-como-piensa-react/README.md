# 02 - ¿Cómo piensa React? (Declarativo y Virtual DOM)

Antes de empezar a escribir código sin parar, necesitamos entender la filosofía detrás de React. Si vienes de usar JavaScript puro (*Vanilla JS*) o librerías como jQuery, tu cerebro está acostumbrado a pensar de forma **imperativa**. React te exige pensar de forma **declarativa**.

## 🤔 Imperativo vs Declarativo

Piensa en la diferencia entre darle a alguien instrucciones paso a paso para llegar a tu casa (Imperativo) o simplemente darle la dirección de tu casa y que use un GPS (Declarativo).

### El enfoque Imperativo (Vanilla JS)
Tú tienes que decirle al navegador **CÓMO** hacer cada pequeña cosa: busca este elemento, crea un botón, ponle esta clase, cámbiale el texto y mételo dentro de este otro elemento.

```javascript
// Ejemplo Imperativo: Crear un botón rojo
const container = document.getElementById('app');
const button = document.createElement('button');
button.className = 'btn-red';
button.textContent = 'Haz clic';
container.appendChild(button);
```

### El enfoque Declarativo (React)
Tú le dices a React **QUÉ** quieres ver en pantalla según el estado actual de tu aplicación, y React se encarga de averiguar el "cómo" actualizar el DOM para que coincida.

```jsx
// Ejemplo Declarativo en React
export const MyComponent = () => {
  return (
    <div id="app">
      <button className="btn-red">Haz clic</button>
    </div>
  );
};
```

## 🧠 El Virtual DOM: El secreto de la velocidad

Manipular el DOM real del navegador (añadir, quitar o modificar nodos HTML) es una de las operaciones más lentas que puede hacer una aplicación web. React soluciona esto usando el **Virtual DOM (VDOM)**.

El Virtual DOM no es más que una copia ligera del DOM real, guardada en la memoria de JavaScript en forma de un objeto gigantesco. Modificar este objeto en memoria es rapidísimo porque no tiene que pintar nada en la pantalla.

### El proceso de actualización
Cuando cambia un dato en tu aplicación, React hace lo siguiente:
1. **Render (Creación):** Crea un nuevo Virtual DOM con los datos actualizados.
2. **Diffing (Comparación):** Compara este *nuevo* VDOM con la versión *anterior*. React utiliza un algoritmo heurístico inteligente para encontrar exactamente qué ha cambiado.
3. **Reconciliation (Reconciliación):** Una vez que sabe qué piezas son diferentes, React va al DOM real y **solo** actualiza esas partes específicas.

> **💡 Nota de rendimiento:** Si tienes una lista de 1000 elementos y solo cambia el texto de uno, React no vuelve a dibujar los 1000 elementos. Solo actualiza el nodo de texto que ha cambiado.

## 🏎️ Bajo el capó: React Fiber

A partir de React 16, el equipo de Meta reescribió por completo el motor principal del Virtual DOM. A esta nueva arquitectura se le llamó **React Fiber**.

¿Por qué era necesario? Antes, cuando React empezaba a actualizar el árbol de componentes, no podía detenerse. Si la actualización era muy pesada, bloqueaba el hilo principal del navegador (y la pantalla se congelaba brevemente).

Fiber introdujo el **renderizado asíncrono e interrumpible**. Ahora, React puede:
- **Pausar** el trabajo de renderizado si hay algo más urgente (como escribir en un input o una animación).
- **Priorizar** ciertas actualizaciones sobre otras.
- **Descartar** trabajo que ya no es necesario porque el usuario navegó a otra vista.

Esta es la tecnología que permite que las aplicaciones de React modernas se sientan extremadamente fluidas.

## 🌊 Flujo de Datos Unidireccional (One-Way Data Flow)

Una regla fundamental es cómo viaja la información: **los datos siempre fluyen hacia abajo.**
- Un componente Padre le pasa datos a su Hijo.
- Un componente Hijo