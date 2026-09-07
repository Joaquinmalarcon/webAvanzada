# webAvanzada

Laboratorio evaluado — Git, Angular, Integración Continua y CD básico con Terraform
Ingeniería Web Avanzada (OII436-1) · Segundo semestre 2026

## Datos del equipo

- Integrante 1: Vicente Cisternas
- Integrante 2: Joaquin Muñoz
- Sección: 1
- Fecha: 07 de septiembre de 2026
- Rama de trabajo: `devops/ci-cd`

## Respuestas

### Pregunta 1 (2 pts)
¿Por qué no se recomienda desarrollar directamente sobre `main` en este laboratorio?

`main` representa el código estable de referencia del repositorio. Si se trabaja directamente ahí, cualquier error o cambio incompleto la afecta de inmediato, sin posibilidad de revisión previa. Usar una rama de trabajo (`devops/ci-cd`) permite aislar los cambios, ejecutar la validación de CI y revisar el código mediante un Pull Request antes de integrarlo, evitando romper la rama principal.

### Pregunta 2 (2 pts)
¿Qué problema se evita al utilizar `--skip-git` al crear el proyecto Angular?

Se evita que Angular CLI inicialice un segundo repositorio Git dentro de `frontend/`. Como el repositorio principal (`webAvanzada`) ya tiene su propio `.git`, crear otro anidado generaría un repositorio dentro de otro (similar a un submódulo no intencional), lo que impediría versionar correctamente los archivos del frontend junto con el resto del proyecto.

### Pregunta 3 (2 pts)
¿Qué verifica `npm run build` en esta etapa del laboratorio?

Verifica que el proyecto compile correctamente en modo producción: que no existan errores de TypeScript, plantillas rotas o dependencias faltantes, y que se generen los artefactos estáticos finales (`dist/`). Es la confirmación de que el código es "compilable" antes de automatizar esa misma tarea en el pipeline de CI.

### Pregunta 4 (2 pts)
¿Qué utilidad tiene revisar `git status` o `git diff --cached` antes de realizar un commit?

Permiten confirmar exactamente qué archivos y qué cambios quedarán incluidos en el commit, evitando subir por error archivos no deseados (credenciales, `node_modules`, archivos temporales) y asegurando que el mensaje del commit refleje realmente el contenido que se está versionando.

### Pregunta 5 (2 pts)
¿Qué evento activa el workflow `ci.yml`?

Se activa con el evento `pull_request` cuando la rama base del Pull Request es `main` (`branches: [main]`).

### Pregunta 6 (2 pts)
En `runs-on: ubuntu-latest`, ¿qué representa `ubuntu-latest`?

Es el runner (máquina virtual) que GitHub Actions provee para ejecutar el job, basado en la versión estable más reciente de Ubuntu Linux que GitHub mantiene disponible para sus imágenes hospedadas.

### Pregunta 7 (2 pts)
Ordene las etapas de validación del job `frontend` y explique por qué `npm ci` se ejecuta antes que las pruebas.

Orden: 1) Obtener código (`checkout`) → 2) Configurar Node.js → 3) Instalar dependencias (`npm ci`) → 4) Ejecutar pruebas (`npm test`) → 5) Construir Angular (`npm run build`).

`npm ci` debe ejecutarse antes que las pruebas porque instala exactamente las dependencias definidas en `package-lock.json` (incluyendo Karma/Jasmine y el propio Angular). Sin ese paso, las pruebas fallarían por falta de módulos instalados, no por errores reales en el código de la aplicación.

### Pregunta 8 (3 pts)
Después del push con la prueba alterada, ¿qué etapa del pipeline falla y qué ocurre con las etapas siguientes?

Falla la etapa "Ejecutar pruebas" (`npm test`), porque la aserción espera el texto `Título incorrecto`, que no existe en el `<h1>` real. Al fallar ese paso, GitHub Actions detiene la ejecución del job: la etapa siguiente ("Construir Angular") no llega a ejecutarse, y el workflow se marca como fallido.

### Pregunta 9 (3 pts)
¿Debería integrarse este Pull Request a `main` mientras el pipeline está fallando? Justifique.

No. Un pipeline en rojo indica que el código no cumple las validaciones automáticas mínimas (en este caso, las pruebas unitarias). Integrarlo pondría en riesgo la estabilidad de `main` y, en este flujo, además dispararía el workflow de CD hacia staging con código roto. La buena práctica de CI/CD es que `main` solo reciba cambios que hayan pasado exitosamente todas las validaciones del pipeline.

### Pregunta 10 (4 pts)
Clasifique cada elemento como "versionable", "variable/configuración" o "secreto/no versionable":

| Elemento | Clasificación |
|---|---|
| `package.json` | Versionable |
| `API_URL` pública | Variable/configuración |
| `AWS_REGION` | Variable/configuración |
| `DB_PASSWORD` | Secreto/no versionable |
| `API_TOKEN` | Secreto/no versionable |
| `terraform.tfstate` | Secreto/no versionable |

### Pregunta 11 (2 pts)
¿Por qué una contraseña o token no debe escribirse directamente dentro de `ci.yml`, `cd.yml` o un archivo TypeScript del frontend?

Porque esos archivos quedan versionados en el historial de Git, visible para cualquier persona con acceso al repositorio (colaboradores, forks, o el público si el repo es abierto), y la credencial permanecería expuesta en el historial aunque se borre después. Los *GitHub Secrets*, en cambio, se inyectan cifrados y solo en tiempo de ejecución, sin quedar expuestos en el código fuente ni en los logs.

### Pregunta 12 (2 pts)
Si un secreto real fue incluido en un commit y luego se agrega su archivo a `.gitignore`, ¿queda solucionado el problema? Explique qué acción adicional debe realizarse.

No queda solucionado. `.gitignore` solo evita que **futuros** cambios de ese archivo se vuelvan a versionar; el secreto ya expuesto sigue presente en los commits anteriores del historial (accesible vía `git log`, clones previos o forks). Es necesario, como mínimo: (1) revocar y rotar la credencial expuesta de inmediato, y (2) reescribir el historial de Git (por ejemplo con `git filter-repo` o BFG Repo-Cleaner) para eliminarla de todos los commits, forzando el push posterior y avisando a los demás colaboradores.

### Pregunta 13 (2 pts)
¿Qué diferencia existe entre `terraform validate`, `terraform plan` y `terraform apply`?

- `terraform validate`: revisa que la sintaxis y estructura del código sean correctas, sin comparar contra el estado real de infraestructura.
- `terraform plan`: calcula y muestra qué cambios se aplicarían (crear, modificar, eliminar) comparando el código con el estado actual, sin ejecutar nada.
- `terraform apply`: ejecuta realmente esos cambios sobre la infraestructura (en este laboratorio, genera la carpeta `staging/` a partir del build de Angular).

### Pregunta 14 (2 pts)
¿Por qué `ci.yml` se activa con `pull_request` y `cd.yml` se activa con `push` sobre `main`?

`ci.yml` valida el código **antes** de integrarlo, por lo que debe correr en cada Pull Request hacia `main`, detectando errores previo a la fusión. `cd.yml` despliega código que **ya fue validado e integrado**, por lo que se dispara cuando el código efectivamente llega a `main` (push), garantizando que solo se despliegue lo que ya pasó por la revisión y las pruebas del PR.

### Pregunta 15 (2 pts)
¿Qué función cumple Terraform dentro de este flujo de CD?

Actúa como herramienta de infraestructura como código (IaC): automatiza de forma declarativa y reproducible la preparación del entorno de staging, en este caso copiando los artefactos compilados de Angular (`dist/frontend/browser`) hacia la carpeta `staging/`, dejando ese "despliegue" simulado documentado y repetible sin intervención manual.

### Pregunta 16 (2 pts)
¿Por qué el workflow usa `${{ secrets.DEMO_TOKEN }}` en lugar de escribir el valor directamente?

Porque `secrets.DEMO_TOKEN` permite que GitHub inyecte el valor de forma cifrada solo en tiempo de ejecución, sin exponerlo en el código fuente ni en los logs (GitHub además enmascara automáticamente su valor si aparece impreso). Escribir el valor directamente lo dejaría visible y versionado de forma permanente en el repositorio.
