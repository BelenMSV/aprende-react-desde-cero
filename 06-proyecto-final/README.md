# Módulo 06: Proyecto Final y Buenas Prácticas

Bienvenido al último escalón de tu transformación de Junior a Senior. Has aprendido a dominar React, gestionar el estado global, conectar con APIs y enrutar tu aplicación. Ahora, debes aprender a pensar como un **Arquitecto de Software**.

En este módulo no aprenderemos librerías nuevas (con excepción del Testing). Nos enfocaremos en **cómo escribir código que pueda ser mantenido por un equipo de 10 personas durante los próximos 5 años sin romperse**. Culminaremos este viaje construyendo un proyecto masivo desde cero y desplegándolo a producción.

**⏱️ Tiempo estimado de estudio y práctica:** 25 - 30 horas.

## 🎯 Objetivos de Aprendizaje

- Dominar la arquitectura de carpetas moderna (Feature-Sliced Design) abandonando la vieja estructura agrupada por "tipos de archivo".
- Utilizar el **React Profiler** para medir el rendimiento real de tu aplicación.
- Entender exactamente cuándo usar (y cuándo NO usar) `useMemo`, `useCallback` y `React.memo` para evitar re-renderizados costosos.
- Implementar **Patrones de Diseño Avanzados**, como los *Compound Components* (Componentes Compuestos) para crear UIs altamente flexibles.
- Configurar un entorno de **Testing** moderno con Vitest y React Testing Library para asegurar que tu código no se rompa con futuras actualizaciones.
- Construir **ReactBoard** (un clon simplificado de Trello/Jira), integrando Zustand, TanStack Query, React Hook Form y React Router en un solo lugar.
- Automatizar y desplegar la aplicación a producción utilizando plataformas en la nube.

## 📚 Temario Detallado

1. **[Arquitectura Escalable: Feature-Sliced Design](./01-arquitectura-escalable/README.md)**
   > Si tienes una carpeta llamada `components` con 50 archivos adentro, tienes un problema. Aprende a estructurar tu proyecto agrupando por "Funcionalidad" (Features) en lugar de por "Tipo".

2. **[Rendimiento Avanzado: Optimizando el Motor](./02-rendimiento-avanzado/README.md)**
   > No optimices prematuramente. Aprende a usar la pestaña Profiler de las DevTools para encontrar cuellos de botella y aplicar memorización (`useMemo`, `useCallback`) solo donde realmente se necesita.

3. **[Patrones de Diseño de Componentes](./03-patrones-de-diseno/README.md)** 💎
   > ¿Cómo construyes un `<Select>` o un `<Modal>` que sea 100% reutilizable y fácil de leer? Descubre el patrón de los Componentes Compuestos (Compound Components) que utilizan librerías como Radix o Material UI.

4. **[La Malla de Seguridad: Testing en React](./04-testing-en-react/README.md)**
   > El código sin pruebas está roto por defecto. Aprende a configurar **Vitest** y **React Testing Library** para escribir pruebas unitarias y de integración que simulen cómo un usuario interactúa con tu interfaz.

5. **[PROYECTO FINAL: ReactBoard (Clon de Trello)](./05-proyecto-react-board/README.md)** 🚀
   > El examen final. Construiremos un gestor de tareas tipo Kanban. Tendrás columnas, tarjetas arrastrables, gestión de usuarios, caché optimizado y UIs optimistas. Todo el curso concentrado en un solo repositorio.

6. **[Despliegue a Producción y CI/CD](./06-despliegue-y-ci-cd/README.md)**
   > Tu código no sirve de nada si solo funciona en `localhost`. Aprenderemos a preparar el proyecto para producción y a desplegarlo en plataformas como Vercel o Netlify de forma automatizada.

---

[🔙 Volver al Índice Global](../README.md)