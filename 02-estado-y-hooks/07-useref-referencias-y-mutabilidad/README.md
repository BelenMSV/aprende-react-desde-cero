# 07 - `useRef`: Referencias, Mutabilidad y el DOM

En React, el estado (`useState`) es la memoria oficial de un componente. Cada vez que el estado cambia, React vuelve a pintar la pantalla para reflejar ese cambio. 

Sin embargo, a veces necesitamos guardar información que **NO** tiene impacto visual. Si guardamos esa información en el estado, estaríamos forzando a React a hacer trabajo inútil (re-renderizados innecesarios).

Para solucionar esto, usamos `useRef`. Este Hook tiene **dos superpoderes principales**:
1. Guardar valores mutables que no provocan re-renderizados (La Caja Fuerte).
2. Acceder directamente a los elementos reales del DOM.

---

## 🔒 1. Primer Superpoder: La Caja Fuerte (Valores Mutables)

Cuando llamas a `useRef(valorInicial)`, React te devuelve un objeto simple con una sola propiedad: `{ current: valorInicial }`.

Puedes leer y modificar esta propiedad en cualquier momento. A diferencia del estado, **modificar `ref.current` es síncrono, inmediato y NO avisa a React** (la pantalla no se actualiza).

### Caso de uso clásico: Guardar el ID de un Temporizador

Imagina que quieres crear un cronómetro. Necesitas guardar el ID del `setInterval` para detenerlo después.

❌ **El Anti-patrón (Usar estado):**
```jsx
// 🚩 Guardar el timerId en el estado provoca un renderizado inútil
const [timerId, setTimerId] = useState(null); 
const [segundos, setSegundos] = useState(0);

const iniciar = () => {
  const id = setInterval(() => setSegundos(s => s + 1), 1000);
  setTimerId(id); // ¡Obliga a React a repintar la pantalla sin motivo visual!
};
```

✅ **El Patrón Profesional (Usar useRef):**
```jsx
import { useState, useRef } from 'react';

export const Cronometro = () => {
  const [segundos, setSegundos] = useState(0);
  // Creamos la referencia en secreto
  const timerRef = useRef(null); 

  const iniciar = () => {
    if (timerRef.current !== null) return; 
    
    // Mutamos la propiedad .current directamente
    timerRef.current = setInterval(() => setSegundos(s => s + 1), 1000);
  };

  const detener = () => {
    clearInterval(timerRef.current);
    timerRef.current = null; // Lo limpiamos sin causar renderizados
  };

  return (
    <div>
      <p>Tiempo: {segundos}s</p>
      <button onClick={iniciar}>Iniciar</button>
      <button onClick={detener}>Detener</button>
    </div>
  );
};
```

---

## 🎯 2. Segundo Superpoder: Acceso Directo al DOM

Hay ciertas cosas que React no puede hacer automáticamente con el estado:
- Poner el foco (`.focus()`) en un input.
- Medir el tamaño de un `<div>`.
- Reproducir un `<video>`.

Para esto, usamos `useRef` para capturar el elemento HTML real.

```jsx
export const Buscador = () => {
  // 1. Creamos la referencia
  const inputRef = useRef(null);

  const enfocarInput = () => {
    // 3. inputRef.current ahora es el <input> real del DOM
    inputRef.current.focus();
    inputRef.current.style.border = "2px solid blue"; 
  };

  return (
    <div>
      {/* 2. Le pasamos la referencia al atributo especial 'ref' */}
      <input ref={inputRef} type="text" placeholder="Busca aquí..." />
      <button onClick={enfocarInput}>Escribir</button>
    </div>
  );
};
```

---

## 🚀 3. Nivel Senior: `forwardRef`

¿Qué pasa si creaste un componente `<InputPersonalizado />` y quieres pasarle una referencia desde el padre?

React bloquea por defecto las referencias hacia componentes personalizados para evitar que el Padre rompa el código del Hijo. Para permitirlas, debes usar `forwardRef`.

```jsx
import { useRef, forwardRef } from 'react';

// EL HIJO: Usamos forwardRef. Recibe (props, ref)
const InputPersonalizado = forwardRef((props, ref) => {
  return <input ref={ref} className="mi-input" {...props} />;
});

// EL PADRE: Lo usa con normalidad
export const Formulario = () => {
  const inputRef = useRef(null);
  return (
    <form>
      <InputPersonalizado ref={inputRef} />
      <button type="button" onClick={() => inputRef.current.focus()}>Enfocar</button>
    </form>
  );
};
```

---

## 🛡️ 4. Nivel Arquitecto: Proteger al Hijo con `useImperativeHandle`

El problema de `forwardRef` es que le das al Padre el control **absoluto** sobre el HTML del Hijo. El Padre podría hacer `inputRef.current.remove()` y borrar tu componente de la pantalla.

Para evitar esto, usamos el Hook `useImperativeHandle`. Este hook te permite **elegir exactamente qué funciones quieres exponerle al padre**, ocultando el HTML real.

```jsx
import { useRef, forwardRef, useImperativeHandle } from 'react';

// EL HIJO:
const ReproductorVideo = forwardRef((props, ref) => {
  const videoRef = useRef(null);

  // Definimos QUÉ le entregamos al Padre en su "ref.current"
  useImperativeHandle(ref, () => {
    return {
      reproducir: () => videoRef.current.play(),
      pausar: () => videoRef.current.pause(),
      // El padre NO puede cambiar el volumen ni borrar el video.
    };
  }, []);

  return <video ref={videoRef} src="gatos.mp4" />;
});

// EL PADRE:
export const App = () => {
  const miVideo = useRef(null);

  const manejarClic = () => {
    // El padre solo puede usar los métodos que el hijo expuso
    miVideo.current.reproducir(); 
  };

  return (
    <div>
      <ReproductorVideo ref={miVideo} />
      <button onClick={manejarClic}>Play</button>
    </div>
  );
};
```

---

## 📏 5. Pro Tip: Midiendo el DOM con "Callback Refs"

Una limitación de `useRef` es que **no te avisa cuando su contenido cambia**. Si renderizas un elemento condicionalmente (`if (mostrar) return <div ref={miRef}>`) y quieres medir su altura justo cuando aparece, `useRef` normal fallará porque no dispara ningún efecto.

La solución oficial es pasar una **función** al atributo `ref` en lugar de un objeto `useRef`. A esto se le llama **Callback Ref**.

```jsx
export const Medidor = () => {
  const [altura, setAltura] = useState(0);

  // Esta función se ejecuta exactamente en el momento en que 
  // React inserta el <h1> en el DOM, o cuando lo destruye (pasando null).
  const medirElemento = (nodoDOM) => {
    if (nodoDOM !== null) {
      setAltura(nodoDOM.getBoundingClientRect().height);
    }
  };

  return (
    <>
      <h1 ref={medirElemento}>Mídeme por favor</h1>
      <p>El título mide {altura} píxeles de alto.</p>
    </>
  );
};
```

---

## ⚠️ La Regla de Oro de `useRef`

**NUNCA leas ni escribas `ref.current` durante el renderizado.**

React asume que la fase de renderizado es "pura" (solo calcula la UI). Como las referencias mutan fuera del control de React, leerlas mientras se construye la interfaz causa bugs impredecibles.

❌ **El Anti-patrón:**
```jsx
const miRef = useRef(0);
miRef.current = miRef.current + 1; // 🚩 ERROR: Mutando durante el render
return <div>{miRef.current}</div>; // 🚩 ERROR: Leyendo durante el render
```

✅ **La forma correcta:**
Solo debes leer/escribir referencias dentro de Manejadores de Eventos (clics) o Efectos (`useEffect`).