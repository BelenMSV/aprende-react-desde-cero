# 02 - Rendimiento Avanzado: Optimizando el Motor 🏎️

React es increíblemente rápido por defecto. En el 95% de los casos, no necesitas preocuparte por el rendimiento. Pero cuando construyes dashboards complejos, tablas con miles de filas o gráficos interactivos, la interfaz puede empezar a sentirse "pesada" o con retraso (lag).

En este tema aprenderemos a identificar el problema real usando herramientas científicas y a aplicar las optimizaciones exactas.

---

## 🔬 1. No adivines, mide: El React Profiler

El error número uno en rendimiento es adivinar qué parte del código es lenta. 
*"Creo que el Navbar está causando lag"*. **¡No! ¡Mídelo!**

React nos proporciona una herramienta oficial en las DevTools del navegador: **El Profiler**.
1. Abre las herramientas de desarrollador (F12).
2. Ve a la pestaña "Profiler" (Asegúrate de tener instalada la extensión React Developer Tools).
3. Dale al botón de grabar (Círculo azul).
4. Interactúa con tu app (ej. haz clic en un botón que se sienta lento).
5. Detén la grabación.

El Profiler te mostrará un gráfico de barras (Flamegraph). Te dirá exactamente **qué componentes se renderizaron, cuántos milisegundos tardaron, y por qué se renderizaron**. Si un componente tardó `0.1ms`, déjalo en paz. Si tardó `15.0ms`, ¡ahí tienes a tu culpable!

---

## 🔄 2. La Anatomía de un Re-render

Para optimizar, primero debes entender por qué React dibuja de nuevo un componente. Ocurre por 3 razones:
1. Su estado interno (`useState`) cambió.
2. Sus `props` (lo que recibe del padre) cambiaron.
3. **El componente Padre se re-renderizó.** (Esta es la causa del 90% de los problemas).

Si el componente `<Padre />` cambia su estado, **TODOS** sus hijos se vuelven a dibujar por defecto, incluso si sus props no cambiaron.

---

## 🛡️ 3. El Escudo Protector: `React.memo`

Si tienes un componente pesado (como un `<GraficoComplejo />`) que no debería volver a dibujarse a menos que sus datos cambien, puedes envolverlo en `React.memo`.

```jsx
import { memo } from 'react';

// 1. Envolvemos el componente con memo()
export const GraficoComplejo = memo(({ datos, titulo }) => {
  console.log("Renderizando gráfico pesado...");
  
  return (
    <div className="grafico">
      <h2>{titulo}</h2>
      {/* ... lógica de dibujado intensiva ... */}
    </div>
  );
});
```

**¿Qué hace `memo`?** Le dice a React: *"Si mi componente padre se vuelve a dibujar, detente. Revisa mis props (`datos` y `titulo`). Si son exactamente iguales a la última vez, NO me vuelvas a dibujar. Usa la versión en caché."*

---

## 🧠 4. El problema de las Referencias: `useMemo` y `useCallback`

`React.memo` parece magia, pero tiene una debilidad fatal: **La igualdad referencial en JavaScript**.

En JavaScript, si comparas dos objetos o funciones idénticas, el resultado es falso:
`{ a: 1 } === { a: 1 } // false`
`(() => {}) === (() => {}) // false`

Mira este código problemático:

❌ **El Anti-patrón que rompe `React.memo`:**
```jsx
export const Dashboard = () => {
  const [contador, setContador] = useState(0);

  // 🚩 CADA VEZ que el contador cambia, React crea una NUEVA función en memoria
  const manejarClic = () => console.log("Clic");
  
  // 🚩 CADA VEZ que el contador cambia, React crea un NUEVO array en memoria
  const datosGrafico = [10, 20, 30];

  return (
    <div>
      <button onClick={() => setContador(c => c + 1)}>
        Contador: {contador}
      </button>

      {/* Aunque GraficoComplejo use memo(), se re-renderizará CADA VEZ que toques 
          el contador, porque 'manejarClic' y 'datosGrafico' son referencias nuevas 
          en cada render. ¡Memo se rompe! */}
      <GraficoComplejo 
        datos={datosGrafico} 
        onClick={manejarClic} 
      />
    </div>
  );
};
```

### ✅ La Solución: Memorizar Valores y Funciones

Para que `React.memo` funcione, debemos asegurar que el array y la función sean **exactamente la misma referencia en memoria** entre renders.

1. **`useMemo`:** Memoriza **valores calculados** (Arrays, Objetos, resultados de cálculos pesados).
2. **`useCallback`:** Memoriza **funciones**.

```jsx
import { useState, useMemo, useCallback } from 'react';

export const DashboardOptimizado = () => {
  const [contador, setContador] = useState(0);

  // ✅ Memorizamos la función. React solo creará una nueva si cambian 
  // las dependencias en el array [].
  const manejarClic = useCallback(() => {
    console.log("Clic");
  }, []); // Array vacío = la función es la misma para siempre

  // ✅ Memorizamos el array.
  const datosGrafico = useMemo(() => {
    return [10, 20, 30];
  }, []); 

  return (
    <div>
      <button onClick={() => setContador(c => c + 1)}>
        Contador: {contador}
      </button>

      {/* ¡Ahora sí! Cuando toques el contador, GraficoComplejo NO se dibujará, 
          porque sus props mantienen la misma referencia en memoria. */}
      <GraficoComplejo datos={datosGrafico} onClick={manejarClic} />
    </div>
  );
};
```

---

## ⚖️ 5. La Regla de Oro (Cuándo NO usarlos)

No uses `useMemo`, `useCallback` o `React.memo` a menos que tengas un problema de rendimiento real.

**¿Por qué?**
* Hacer que React compare referencias antiguas con nuevas (lo que hace `useMemo`) tiene un costo de CPU.
* Mantener variables viejas en memoria tiene un costo de RAM.
* Si memorizas un componente simple como un `<Button>`, el costo de calcular si debe memorizarse o no, es **más caro** que simplemente dejar que React lo dibuje de nuevo. React renderiza botones en microsegundos.

**Usa estas herramientas SÓLO si:**
1. Estás pasando objetos/funciones como *props* a un componente envuelto en `React.memo`.
2. Estás pasando objetos/funciones como dependencia a un `useEffect` (para evitar bucles infinitos).
3. Tienes un cálculo matemático masivo (ej. procesar 10,000 elementos de una tabla).