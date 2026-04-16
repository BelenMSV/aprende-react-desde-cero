# Módulo 02: Estado y Ciclo de Vida (Hooks)

En el módulo anterior aprendimos a maquetar interfaces hermosas y modulares, pero eran estáticas. En este módulo, aprenderemos a dotar de "memoria" a nuestros componentes y a conectarlos con el mundo exterior. 

Entender el flujo de datos y el ciclo de vida es la clave para dominar React y evitar los errores de rendimiento más comunes en aplicaciones reales.

**⏱️ Tiempo estimado de estudio y práctica:** 15 - 20 horas.

## 🎯 Objetivos de Aprendizaje

- Comprender la diferencia entre variables normales y el Estado de React.
- Dominar la **inmutabilidad** al manejar objetos y arreglos complejos.
- Aplicar el concepto de **Estado Derivado** para escribir código más limpio y eficiente.
- Sincronizar componentes con APIs externas y el DOM usando `useEffect`.
- Evitar fugas de memoria (*Memory Leaks*) mediante funciones de limpieza.
- Utilizar `useRef` para persistir valores y manipular elementos del DOM directamente.
- Abstraer lógica de negocio en **Custom Hooks** reutilizables.

## 📚 Temario Detallado

1. **[Introducción al Estado y useState](./01-intro-al-estado-y-usestate/README.md)**
   > Entiende por qué React necesita el estado para "recordar" cosas y cómo este dispara el ciclo de renderizado.

2. **[useState Profundo: Objetos y Arrays](./02-usestate-profundo-objetos-y-arrays/README.md)**
   > Aprende a trabajar con estructuras de datos complejas usando el operador *spread* y respetando la inmutabilidad.

3. **[Estado Derivado y Lifting State Up](./03-estado-derivado-y-lifting-state/README.md)**
   > Deja de crear estados innecesarios. Aprende a calcular valores al vuelo y a coordinar componentes hermanos subiendo el estado al padre.

4. **[Ciclo de Vida y useEffect](./04-ciclo-de-vida-y-useeffect/README.md)**
   > Domina los "efectos secundarios". Aprende qué es el montaje, la actualización y el desmontaje en el mundo de los componentes funcionales.

5. **[Dependencias y Cleanup en useEffect](./05-dependencias-y-cleanup-useeffect/README.md)**
   > El corazón de la sincronización. Aprende a controlar cuándo se ejecuta un efecto y cómo limpiar suscripciones o temporizadores.

6. **[Cuándo NO usar useEffect](./06-cuando-no-usar-useeffect/README.md)**
   > Una guía de supervivencia para evitar el abuso de este Hook y aprender patrones modernos que hacen tu app más rápida.

7. **[useRef: Referencias y Mutabilidad](./07-useref-referencias-y-mutabilidad/README.md)**
   > Descubre cómo guardar información que no necesita renderizar la pantalla y cómo "tocar" el HTML real cuando es estrictamente necesario.

8. **[Custom Hooks y las Reglas de Oro](./08-custom-hooks-y-reglas/README.md)**
   > Aprende a crear tus propios Hooks para reutilizar lógica entre componentes y respeta las leyes inquebrantables de React.

---

[🔙 Volver al Índice Global](../README.md)