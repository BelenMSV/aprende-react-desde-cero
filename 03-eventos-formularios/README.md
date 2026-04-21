# Módulo 03: Eventos y Formularios

Si hay algo que frustra a los desarrolladores cuando empiezan en React, son los formularios. En HTML nativo, los formularios "funcionan solos". En React, debemos tomar el control manual de cada tecla que el usuario pulsa, lo que a menudo resulta en aplicaciones lentas y código difícil de mantener.

En este módulo no solo aprenderemos la forma "nativa" de manejar inputs para entender cómo piensa React, sino que evolucionaremos rápidamente hacia el **estándar de la industria**: usar librerías especializadas que nos devuelven el rendimiento y la sanidad mental.

**⏱️ Tiempo estimado de estudio y práctica:** 10 - 14 horas.

## 🎯 Objetivos de Aprendizaje

- Entender el sistema de **Eventos Sintéticos** de React y la delegación de eventos.
- Dominar el patrón de **Componentes Controlados** (Uniendo inputs al estado).
- Conocer la alternativa de alto rendimiento: **Componentes No Controlados** con `useRef`.
- Manejar inputs complejos: Múltiples checkboxes, selects y carga de archivos.
- Crear experiencias de usuario fluidas (UX) deshabilitando botones y mostrando errores.
- Construir formularios de nivel empresarial usando **React Hook Form**.
- Declarar esquemas de validación estrictos y tipados utilizando **Zod**.

## 📚 Temario Detallado

1. **[Eventos Sintéticos en React](./01-eventos-sinteticos-en-react/README.md)**
   > Descubre cómo React envuelve los eventos nativos del navegador (`onClick`, `onChange`) para hacerlos predecibles en todos los sistemas operativos.

2. **[Componentes Controlados](./02-componentes-controlados/README.md)**
   > La forma clásica: Cómo sincronizar un `<input>` con un `useState` y el patrón del "Manejador Universal" para formularios grandes.

3. **[Componentes No Controlados y useRef](./03-componentes-no-controlados-useref/README.md)**
   > Optimización extrema: Cómo leer datos directamente del DOM solo cuando el usuario hace *submit*, evitando re-renderizados innecesarios.

4. **[Inputs Complejos: Selects y Checkboxes](./04-inputs-complejos-selects-y-checkboxes/README.md)**
   > Domina las etiquetas más allá del simple texto. Aprende a manejar arreglos de opciones y el objeto `FormData`.

5. **[Validaciones Manuales y UX](./05-validaciones-manuales-y-ux/README.md)**
   > Protege tus datos antes de enviarlos al servidor. Aprende a escribir expresiones regulares básicas y a manejar el estado `isSubmitting`.

6. **[El Estándar: React Hook Form](./06-react-hook-form-el-estandar/README.md)** 🚀
   > Sube a nivel Senior. Aprende a usar la librería que domina el mercado laboral combinando lo mejor del rendimiento no controlado con una API amigable.

7. **[Validación por Esquemas con Zod](./07-validacion-por-esquemas-con-zod/README.md)** 💎
   > Adiós a los `if` interminables. Construye esquemas de datos robustos y conéctalos mágicamente a React Hook Form con `zodResolver`.

---

[🔙 Volver al Índice Global](../README.md)