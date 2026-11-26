# Monorepo NestJS + React

Este proyecto reúne una API de NestJS y un cliente React en un mismo repositorio. Turborepo coordina los comandos de las aplicaciones desde la raíz y permite reutilizar la caché de las tareas que pueden repetirse.

## Estructura

```text
apps/
├── api/       # API NestJS
└── client/    # Cliente React con Vite
```

Cada aplicación mantiene su propio `package.json` y sus scripts. La configuración compartida del monorepo está en la raíz.

## Requisitos

- Node.js instalado.
- npm instalado. La configuración del proyecto declara npm `11.17.0` como gestor de paquetes.

## Instalación

Desde la raíz del repositorio:

```bash
npm install
```

El `package.json` raíz usa npm workspaces:

```json
"workspaces": ["apps/*"]
```

Por eso una sola instalación gestiona las dependencias de `apps/api` y `apps/client`.

## Configuración raíz

El `package.json` de la raíz contiene los comandos principales:

```json
"scripts": {
  "dev": "turbo dev",
  "build": "turbo build"
}
```

Turbo está declarado como dependencia de desarrollo:

```json
"devDependencies": {
  "turbo": "^2.11.13"
}
```

### `turbo.json`

La configuración usa el esquema de Turborepo y define dos tareas:

```json
{
  "$schema": "https://turborepo.dev/schema.json",
  "tasks": {
    "dev": {
      "cache": false
    },
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    }
  }
}
```

- `dev` no usa caché porque es un proceso de desarrollo que permanece activo y observa cambios.
- `build` espera primero las tareas `build` de las dependencias (`^build`) y considera como salida los archivos dentro de `dist/**`.

## Scripts de las aplicaciones

### API (`apps/api`)

- `npm run dev`: inicia NestJS en modo desarrollo, con recarga (`nest start --watch --preserveWatchOutput`).
- `npm run build`: compila la API con `nest build`.
- `npm run start`: inicia la API normalmente.
- `npm run start:prod`: inicia la versión compilada con `node dist/main`.
- `npm run lint`: ejecuta Oxlint.
- `npm test`: ejecuta las pruebas con Vitest.

### Cliente (`apps/client`)

- `npm run dev`: inicia el servidor de desarrollo de Vite.
- `npm run build`: comprueba TypeScript y genera la compilación de Vite (`tsc -b && vite build`).
- `npm run lint`: ejecuta ESLint.
- `npm run preview`: sirve la compilación para previsualizarla.

## Desarrollo

Para iniciar el modo desarrollo de las aplicaciones desde la raíz:

```bash
npm run dev
```

Este comando ejecuta `turbo dev` y Turbo busca el script `dev` de cada workspace (`apps/api` y `apps/client`).

También se puede iniciar una aplicación de forma individual:

```bash
npm run dev --workspace=api
npm run dev --workspace=client
```

## Build

Para compilar todas las aplicaciones desde la raíz:

```bash
npm run build
```

Turbo ejecuta los scripts `build` de los workspaces respetando sus dependencias. La compilación de cada aplicación produce sus salidas en `dist/**`, que es el patrón configurado para la caché.

También se puede compilar una aplicación individual:

```bash
npm run build --workspace=api
npm run build --workspace=client
```

## ¿Cómo coordina y cachea Turbo?

Turbo detecta los workspaces y ejecuta en paralelo las tareas que no dependen entre sí. Para `build`, `dependsOn: ["^build"]` establece el orden de compilación entre paquetes dependientes y `outputs: ["dist/**"]` indica qué resultados debe conservar.

Cuando una tarea cacheable se ejecuta otra vez sin cambios relevantes, Turbo puede reutilizar sus resultados en lugar de repetir el trabajo. La tarea `dev` es la excepción: tiene la caché desactivada porque necesita ejecutarse continuamente y reaccionar a los cambios del código.
