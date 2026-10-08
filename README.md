# DevTasks

## Descripción

DevTasks es una aplicación web para gestionar tareas, hecha con HTML, CSS y JavaScript. Con ella puedes:

- crear tareas;
- marcarlas como completadas;
- eliminarlas;
- filtrarlas (todas, pendientes o completadas);
- ver cuántas tareas hay en total, pendientes y completadas;
- guardar las tareas en el navegador con `localStorage`, así no se pierden al recargar.

En esta práctica he convertido el proyecto en un repositorio con un flujo DevOps usando GitHub: trabajo con ramas y Pull Requests, tests automáticos, protección de la rama `main`, publicación automática en GitHub Pages y Dependabot para las dependencias.

## Instalación

Primero hay que tener instalado Node.js, que ya trae npm.

Después se descarga el proyecto y se instalan las dependencias:

```bash
git clone https://github.com/angelo808080-lgtm/Proyecto1_Entrega.git
cd Proyecto1_Entrega
npm install
```

Para ver la app en local uso Live Server en VS Code. No funciona abriendo el `index.html` con doble clic porque el JavaScript usa `import`/`export` (módulos) y eso necesita un servidor.

## Tests

Los tests están hechos con Vitest y están en `tests/taskManager.test.js`. Prueban las cuatro funciones de `taskManager.js`: `isValidTask`, `createTask`, `filterTasks` y `getTaskStats`.

Para ejecutarlos:

```bash
npm test
```

Si todo está bien, todos los tests salen en verde.

## GitHub Actions

Tengo dos workflows dentro de `.github/workflows/`:

- **`ci.yml` (Continuous Integration):** se ejecuta cada vez que se abre o se actualiza una Pull Request hacia `main`. Descarga el código, prepara Node.js, instala las dependencias y ejecuta los tests. Si algún test falla, la PR sale en rojo.
- **`deploy.yml` (Deploy a GitHub Pages):** se ejecuta cuando entra un cambio en `main` (por ejemplo, al hacer merge de una PR). Sube la web a GitHub Pages automáticamente, así no tengo que publicarla a mano.

## Pull Requests

A partir del paso 5 de la práctica ya no trabajo directamente en `main`. Lo que hago es:

1. Crear una rama nueva desde `main`, por ejemplo `feature/dependabot`.
2. Hacer los cambios, commit y push de esa rama.
3. Abrir una Pull Request hacia `main`.
4. GitHub Actions ejecuta los tests solo.
5. Si los tests pasan, hago merge. Si fallan, lo arreglo y vuelvo a hacer push.

Además, he protegido la rama `main` para que:

- no se pueda subir nada sin Pull Request;
- no se pueda hacer merge si el check **Install Dependencies and Run Tests** no ha pasado.

Para comprobarlo, provoqué un error a propósito en `getTaskStats`: el CI falló y el merge quedó bloqueado. Al corregirlo, los tests volvieron a pasar.

```
rama feature  →  Pull Request  →  Tests  →  PASS  →  Merge  →  main  →  Deploy  →  GitHub Pages
                                          →  FAIL  →  Corregir
```

## Deploy

La web está publicada con GitHub Pages y se actualiza sola cada vez que se hace merge a `main`:

https://angelo808080-lgtm.github.io/Proyecto1_Entrega/

## Dependencias

El proyecto usa npm. La única dependencia es Vitest, que sirve para los tests (está en `package.json`).

Para no tener que revisar a mano si hay versiones nuevas, he configurado Dependabot en `.github/dependabot.yml`. Una vez por semana revisa:

- las dependencias de npm;
- las versiones de las acciones que uso en los workflows (por ejemplo `actions/checkout`).

Si encuentra algo para actualizar, abre una Pull Request él solo, y esa PR también tiene que pasar los tests antes de hacer merge.

## Arquitectura

Al principio todo el código estaba en `app.js`. Lo he separado en dos archivos:

- **`js/taskManager.js`**: aquí está la lógica de las tareas (crear una tarea, comprobar si es válida, filtrar y calcular estadísticas). Estas funciones no tocan el HTML ni el `localStorage`, solo reciben datos y devuelven un resultado. Por eso se pueden testear sin abrir el navegador.
- **`js/app.js`**: aquí está todo lo que tiene que ver con la página: leer el formulario, pintar la lista, los botones, los filtros y guardar en `localStorage`. Para hacer los cálculos usa las funciones de `taskManager.js`.
- **`tests/`**: aquí están los tests. Como importan solo `taskManager.js`, se pueden ejecutar en mi ordenador con `npm test` y también en GitHub Actions.