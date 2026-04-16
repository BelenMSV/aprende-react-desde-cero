# 06 - Cuándo NO usar `useEffect`

Existe una broma en la comunidad de React: *Si le das a un Junior un martillo (useEffect), tratará todo como un clavo.*

Como `useEffect` permite sincronizar datos y ejecutar código automáticamente, es muy tentador usarlo para controlar la lógica interna de tu aplicación. Sin embargo, **los efectos son una vía de escape** del paradigma de React. Deben usarse solo para salir de React y sincronizarse con sistemas externos.

Si usas efectos para la lógica que podría resolverse durante el renderizado o en un evento, harás que tu aplicación sea lenta (por re-renderizados dobles) y difícil de mantener.

Aquí tienes los 4 Anti-Patrones más comunes y cómo solucionarlos.

---

## ❌ 1. Transformar datos (Anti-Patrón de Estado Derivado)

Esto lo vimos en el Tema 3, pero es vital recordarlo en el contexto de los efectos. Si tienes un estado y quieres crear otro a partir de él, **nunca uses un efecto**.

❌ **El Anti-patrón:**
```jsx
const [usuarios, setUsuarios] = useState([]);
const [activos, setActivos] = useState([]);

useEffect(() => {
  // 🚩 MAL: Esto provoca una cascada. React pinta todos los usuarios, 
  // luego ejecuta el efecto, y vuelve a pintar la pantalla con los activos.
  const filtrados = usuarios.filter(u => u.isActive);
  setActivos(filtrados);
}, [usuarios]);
```

✅ **La Solución (Cálculo durante el renderizado):**
```jsx
const [usuarios, setUsuarios] = useState([]);

// ✅ BIEN: Se calcula al instante sin re-renderizados extra.
const activos = usuarios.filter(u => u.isActive);
```

---

## ❌ 2. Manejar Eventos del Usuario

Este es un error de lógica de negocio muy común. Imagina un botón de "Comprar". Cuando el usuario compra, quieres mostrar una notificación.

❌ **El Anti-patrón (Usar el efecto como observador):**
```jsx
const [compraExitosa, setCompraExitosa] = useState(false);

useEffect(() => {
  if (compraExitosa) {
    // 🚩 MAL: La notificación es resultado de un CLIC, no del ciclo de vida.
    mostrarNotificacion("¡Gracias por tu compra!");
  }
}, [compraExitosa]);

const manejarCompra = async () => {
  await procesarPago();
  setCompraExitosa(true);
};
```

✅ **La Solución (Lógica dentro del Evento):**
Si una acción es causada directamente por el usuario (un clic, un envío de formulario), la lógica debe ir **dentro de la función manejadora del evento**, no en un efecto.

```jsx
const manejarCompra = async () => {
  await procesarPago();
  setCompraExitosa(true);
  // ✅ BIEN: Sabemos exactamente qué causó la notificación.
  mostrarNotificacion("¡Gracias por tu compra!");
};
```

---

## ❌ 3. Efectos en Cadena (El Efecto Dominó)

Cuando tienes variables que dependen unas de otras, es tentador encadenarlas.

❌ **El Anti-patrón:**
```jsx
useEffect(() => { setPasoB(pasoA + 1); }, [pasoA]);
useEffect(() => { setPasoC(pasoB * 2); }, [pasoB]);
useEffect(() => { enviarAlServidor(pasoC); }, [pasoC]);
```
*Por qué es terrible:* Cada eslabón de esta cadena fuerza a React a hacer un renderizado completo de la pantalla. Es ineficiente y si hay un bug, es casi imposible saber qué efecto lo disparó.

✅ **La Solución:**
Calcula todo lo que puedas en el evento inicial que disparó el cambio, o calcula los valores directamente durante el renderizado (estado derivado).

---

## ❌ 4. Inicializar toda la App (La trampa del Fetch)

Hasta ahora hemos enseñado que el "Escenario B" (Arreglo Vacío `[]`) se usa para pedir datos a una API al cargar la página. 

```jsx
// 🚩 Lo que te enseñan en el 90% de los tutoriales (y que hoy se considera obsoleto)
useEffect(() => {
  fetch('/api/datos').then(res => res.json()).then(setDatos);
}, []);
```

Aunque funciona, **la documentación oficial de React hoy en día desaconseja hacer fetch manual dentro de useEffect en aplicaciones grandes**. 

**¿Por qué?**
1. **No soporta SSR (Server-Side Rendering):** Tu app primero carga en blanco, luego carga el código JS, y recién ahí empieza a pedir datos.
2. **Cascadas de Red:** Si un componente Padre hace *fetch* y luego pinta al componente Hijo, el componente Hijo no puede empezar su propio *fetch* hasta que el Padre termine.
3. **No hay caché:** Si cambias de pestaña y vuelves, vuelve a pedir los datos a lo tonto.

✅ **El Estándar Moderno de la Industria:**
En el mundo real (y lo que las empresas exigen), no usamos `useEffect` para pedir datos críticos de la interfaz. Usamos herramientas especializadas que manejan caché, re-intentos y carga paralela.

Las dos formas aceptadas hoy en día son:
1. **Librerías de Fetching (Para apps tradicionales):** Como *React Query (TanStack Query)* o *SWR*. 
2. **Frameworks de React:** Como *Next.js* o *Remix*, que piden los datos en el servidor antes de enviarle la página al usuario.

*(Nota: Aprenderemos a usar React Query a fondo en el Módulo de Asincronía y APIs, pero debes saber desde hoy que el fetch manual con useEffect es cosa del pasado).*

---

## 🚦 Regla de Oro (Resumen)

Cuando vayas a escribir un `useEffect`, pregúntate: **¿Estoy intentando sincronizarme con un sistema externo (API, DOM, Navegador, Animación 3D)?**
- Si la respuesta es **SÍ**: ¡Adelante! Ese es el uso correcto de `useEffect`.
- Si la respuesta es **NO** (solo estoy calculando datos, reaccionando a un botón, o conectando componentes): **Borra el efecto.**