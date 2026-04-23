# 01 - Arquitectura Escalable: Feature-Sliced Design (FSD)

El primer día en un trabajo como Junior, abres el repositorio de la empresa y ves un monstruo de 500 archivos. Tu primera tarea es: *"Arregla un bug en el formulario de inicio de sesión"*. 

Si el proyecto tiene una mala arquitectura, tardarás 3 horas solo en encontrar los archivos. Si tiene una buena arquitectura, lo resolverás en 15 minutos.

En este tema aprenderemos a estructurar un proyecto de React para que soporte años de desarrollo sin colapsar.

---

## ❌ 1. El Anti-patrón Clásico: Agrupación por Tipo

Cuando aprendemos React, casi todos los tutoriales nos enseñan a crear carpetas basadas en el **tipo de archivo** (lo que hace la pieza técnicamente).

```text
src/
 ┣ 📂 components/   <-- 150 archivos mezclados (Botones, Formularios, Modales)
 ┣ 📂 hooks/        <-- 40 archivos
 ┣ 📂 pages/        <-- 30 archivos
 ┣ 📂 services/     <-- Llamadas a APIs
 ┗ 📂 store/        <-- Estado global
```

**El problema (Alta Acoplamiento, Baja Cohesión):**
Para cambiar la lógica del "Login", tienes que abrir `components/LoginForm.jsx`, modificar `hooks/useAuth.js`, tocar `services/authApi.js` y editar `store/authStore.js`. Tienes que saltar por todo el árbol de carpetas constantemente. Si quieres borrar la funcionalidad de Login, tienes que ir a "cazar" archivos sueltos en 4 carpetas distintas.

---

## ✅ 2. La Solución Arquitectónica: Agrupación por Funcionalidad (Feature-Based)

En el desarrollo de software moderno, la industria ha adoptado metodologías inspiradas en el **Feature-Sliced Design (FSD)** o Diseño Orientado al Dominio.

La regla de oro es: **Agrupa los archivos por lo que SIGNIFICA para el negocio, no por lo que son técnicamente.**

La estructura moderna estándar se divide en estas capas principales:

```text
src/
 ┣ 📂 app/          <-- Configuración global que arranca la app (Router, Providers, Estilos base).
 ┣ 📂 pages/        <-- Composición de funcionalidades para armar una pantalla.
 ┣ 📂 features/     <-- EL CORAZÓN: Cada carpeta aquí es un "mini-proyecto" aislado.
 ┗ 📂 shared/       <-- Lo verdaderamente global: Componentes genéricos (Botones), utilidades puras.
```

---

## 🏗️ 3. Anatomía de una *Feature* (Funcionalidad)

Veamos cómo se ve la carpeta `/features`. Imagina que nuestra app tiene Usuarios, Tareas y Autenticación. Cada una tiene su propio ecosistema aislado.

```text
src/features/auth/
 ┣ 📂 api/          <-- Servicios de Axios y TanStack Query solo de Auth
 ┣ 📂 components/   <-- Formularios de Login/Registro (No son genéricos)
 ┣ 📂 hooks/        <-- Lógica específica (ej. useLoginMutation)
 ┣ 📂 store/        <-- Zustand store (useAuthStore)
 ┗ 📜 index.js      <-- 🛡️ Public API (Punto de entrada)
```

### 🛡️ El concepto de "Public API" (Barrera de Aislamiento)
Fíjate en el archivo `index.js` (o `index.ts`) dentro de cada *feature*. 
La regla de oro de un Arquitecto es que **ningún componente externo debe acceder a los archivos internos de una feature directamente**. Todo debe importarse a través del `index.js`.

```javascript
// src/features/auth/index.js
// Solo exportamos lo que el resto de la app necesita saber.
// Todo lo demás se queda como "privado" de esta feature.
export { LoginForm } from './components/LoginForm';
export { useAuthStore } from './store/useAuthStore';
export { ProtectedRoute } from './components/ProtectedRoute';
```

❌ **Importación Ilegal:**
`import { LoginForm } from '@/features/auth/components/LoginForm';`

✅ **Importación Arquitectónica (a través del index):**
`import { LoginForm } from '@/features/auth';`

---

## ⚖️ 4. Reglas de Dependencia (Cómo evitar el código espagueti)

Para que esta arquitectura funcione, debes respetar estrictamente hacia dónde fluyen las importaciones:

1. **`shared/` no puede importar nada de nadie.** Son piezas de Lego puras (ej. un `Button` o un `formatDate`).
2. **`features/` pueden importar de `shared/`, pero NO de otras features.** (Si `auth` necesita de `tareas`, hay un problema de diseño. Deberían conectarse más arriba, en la página).
3. **`pages/` importan de `features/` y de `shared/`.** Una página solo es un cascarón vacío que junta piezas de Lego (Shared) y Bloques de Lógica (Features) para mostrarlos en pantalla.
4. **`app/` importa a las `pages/`** para meterlas en el enrutador.

### 🏆 Beneficios Inmediatos:
* **Contexto Cognitivo:** Si tienes que trabajar en las Tareas, entras a `/features/tareas/` y tienes todo lo que necesitas a la mano sin distraerte.
* **Refactorización Segura:** Si el jefe dice "Quita el sistema de comentarios", simplemente borras la carpeta `/features/comentarios` y sabes que no has roto nada más en la aplicación.
* **Escalabilidad Infinita:** Da igual si tu app tiene 10 o 1,000 funcionalidades, el directorio raíz siempre tendrá las mismas 4 carpetas limpias.