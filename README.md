# Respuestas del laboratorio
# Vicente Cisternas, Joaquín Muñoz
### Pregunta 1
No se recomienda trabajar directamente en `main` porque es la rama estable. Es mejor usar una rama aparte para probar los cambios antes de integrarlos.

### Pregunta 2
`--skip-git` evita crear otro repositorio Git dentro de `frontend`.

### Pregunta 3
`npm run build` verifica que Angular compile correctamente y genere los archivos en `dist/`.

### Pregunta 4
`git status` y `git diff --cached` permiten revisar qué cambios se incluirán antes del commit.

### Pregunta 5
El workflow `ci.yml` se activa con un `pull_request` hacia la rama `main`.

### Pregunta 6
`ubuntu-latest` indica que el job se ejecutará en una máquina virtual con Ubuntu proporcionada por GitHub.

### Pregunta 7
El orden es:

1. Obtener código.
2. Configurar Node.js.
3. Instalar dependencias.
4. Ejecutar pruebas.
5. Construir Angular.

`npm ci` va antes de las pruebas porque instala las dependencias necesarias.

### Pregunta 8
Falla la etapa de pruebas (`npm test`). Por eso las etapas siguientes, como `npm run build`, no se ejecutan.

### Pregunta 9
No. El Pull Request no debería integrarse mientras el pipeline esté fallando, porque podría afectar la estabilidad de `main`.

### Pregunta 10

| Elemento | Clasificación |
|---|---|
| `package.json` | Versionable |
| `API_URL` pública | Variable/configuración |
| `AWS_REGION` | Variable/configuración |
| `DB_PASSWORD` | Secreto/no versionable |
| `API_TOKEN` | Secreto/no versionable |
| `terraform.tfstate` | Secreto/no versionable |

### Pregunta 11
Porque las contraseñas y tokens quedarían expuestos en Git. Es mejor utilizar `GitHub Secrets`.

### Pregunta 12
No. `.gitignore` no elimina el secreto del historial. Se debe revocar/rotar la credencial y eliminarla del historial de Git.

### Pregunta 13
- `terraform validate`: verifica que el código sea válido.
- `terraform plan`: muestra los cambios que se realizarían.
- `terraform apply`: ejecuta esos cambios.

### Pregunta 14
`ci.yml` usa `pull_request` para validar los cambios antes de integrarlos. `cd.yml` usa `push` sobre `main` para desplegar los cambios ya integrados.

### Pregunta 15
Terraform automatiza y reproduce el despliegue del frontend en el entorno de staging.

### Pregunta 16
`${{ secrets.DEMO_TOKEN }}` permite usar el token sin escribirlo directamente en el código y evita exponerlo en el repositorio.
