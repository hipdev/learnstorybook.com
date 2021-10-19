---
title: 'Desplegar Storybook'
tocTitle: 'Desplegar'
description: 'Desplegar Storybook online con GitHub y Netlify'
---

En este tutorial hemos ejecutado Storybook en nuestra máquina de desarrollo. También se puede compartir ese Storybook con el equipo, especialmente con los miembros no técnicos. Implementemos Storybook en línea para ayudar a los compañeros de equipo a revisar la implementación de la UI.

## Exportando como una app estática

Para desplegar Storybook primero necesitamos exportarlo como una aplicación web estática. Esta funcionalidad ya está integrada en Storybook y preconfigurada.

Corriendo `yarn build-storybook`, obtendrás un Storybook estático en el directorio `storybook-static`, que luego se puede desplegar en cualquier servicio de alojamiento de sitios estáticos.

## Publicar Storybook

Este tutorial utiliza <a href="https://www.chromatic.com/"> Chromatic </a>, un servicio de publicación gratuito creado por los mantenedores de Storybook. Nos permite desplegar y alojar nuestro Storybook de forma segura en la nube.

### Configurar un repositorio en GitHub

Antes de comenzar, nuestro código local debe sincronizarse con un servicio de control de versiones remoto. Cuando nuestro proyecto se inicializó en el [capítulo Empezando](/intro-to-storybook/react/es/get-started/), ya inicializamos un repositorio local. En esta etapa, ya tenemos un conjunto de confirmaciones que podemos enviar a un repositiorio remoto.

Vaya a GitHub y cree un nuevo repositorio para nuestro proyecto [aquí](https://github.com/new). Nombra el repositorio “taskbox”, igual que nuestro proyecto local.

![GitHub setup](/intro-to-storybook/github-create-taskbox.png)lea

En el nuevo repositorio, tome la URL de origen del repositorio y agréguela a su proyecto git con este comando:

```bash
$ git remote add origin https://github.com/<su usuario>/taskbox.git
```

Finalmente, envíamos nuestro repositorio local al repositorio remoto en GitHub con:

```bash
$ git push -u origin main
```

### Obtener Chromatic

Agregue el paquete como una dependencia de desarrollo.

```bash
yarn add -D chromatic
```

Una vez que el paquete esté instalado, [inicie sesión en Chromatic](https://www.chromatic.com/start) con su cuenta de GitHub (Chromatic solo solicitará permisos ligeros). Luego crearemos un nuevo proyecto llamado "taskbox" y lo sincronizaremos con el repositorio de GithHub que hemos configurado.

Haga clic en `Choose GitHub repo` debajo de colaboradores y seleccione su repositorio.

<video autoPlay muted playsInline loop style="width:520px; margin: 0 auto;">
  <source
    src="/intro-to-storybook/chromatic-setup-learnstorybook.mp4"
    type="video/mp4"
  />
</video>

Copie el `project-token` único que se generó para su proyecto. Luego ejecútelo, escribiendo lo siguiente en la línea de comando, para construir e implementar nuestro Storybook. Asegúrese de reemplazar `project-token` con el token de tu proyecto.

```bash
yarn chromatic --project-token=<project-token>
```

![Chromatic running](/intro-to-storybook/chromatic-manual-storybook-console-log.png)

Cuando termine, obtendrá un enlace `https://aleatorio-uuid.chromatic.com` a su Storybook publicado. Comparta el enlace con su equipo para recibir comentarios.

![Storybook deployed with chromatic package](/intro-to-storybook/chromatic-manual-storybook-deploy-6-0.png)

¡Hurra! Publicamos Storybook con un comando, pero ejecutar manualmente un comando cada vez que queremos obtener comentarios sobre la implementación de la UI es repetitivo. Idealmente, publicaríamos la última versión de los componentes cada vez que empujamos el código. Necesitaremos implementar Storybook continuamente.

## Integración continua con Chromatic

Ahora que nuestro proyecto está alojado en un repositorio de GitHub, podemos usar un servicio de integración continua (CI) para desplegar nuestro Storybook automáticamente. [Acciones de GitHub](https://github.com/features/actions) es un servicio gratuito de CI integrado en GitHub que facilita la publicación automática.

### Agregar una acción de GitHub para implementar Storybook

En la carpeta raíz de nuestro proyecto, cree un nuevo directorio llamado `.github` y luego cree otro directorio` workflows` dentro de él.

Cree un nuevo archivo llamado `chromatic.yml` como el que se muestra a continuación. Asegúrese de reemplazar `CHROMATIC_PROJECT_TOKEN` con el token de su proyecto.

```yaml:title=.github/workflows/chromatic.yml
# Nombre del flujo de trabajo
name: 'Chromatic Deployment'

# Evento para el flujo de trabajo
on: push

# Lista de trabajos
jobs:
  test:
    # Sistema operativo
    runs-on: ubuntu-latest
    # Pasos del trabajo
    steps:
      - uses: actions/checkout@v1
      - run: yarn
        #👇 Agrega Chromatic como un paso en el flujo de trabajo
      - uses: chromaui/action@v1
        # Opciones necesarias para la acción de GitHub de Chromatic
        with:
          #👇 Chromatic projectToken, ver https://storybook.js.org/tutorials/intro-to-storybook/react/en/deploy/ para obtenerlo
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          token: ${{ secrets.GITHUB_TOKEN }}
```

<div class = "aside"> <p> 💡 Para ser breves <a href = "https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted- secretos ">los `secrets`(secretos) de GitHub </a> no se mencionaron. Los secretos son variables de entorno seguras proporcionadas por GitHub así no necesitas complicarte al codificar el<code> project-token </code>. </p> </div>

### Comprometer la acción

En la línea de comando, escriba el siguiente comando para agregar los cambios que se realizaron:

```bash
git add .
```

Luego, confírmelo escribiendo:

```bash
git commit -m "GitHub action setup"
```

Finalmente, envíelos al repositorio remoto con:

```bash
git push origin main
```

Una vez que haya configurado la acción de GitHub. Su Storybook se implementará en Chromatic cada vez que envíe el código. Puedes encontrar todos los Storybooks publicados en la pantalla de compilación de tu proyecto en Chromatic.

![Chromatic user dashboard](/intro-to-storybook/chromatic-user-dashboard.png)

Haga clic en la última compilación, debería ser la que está en la parte superior.

Luego, haga clic en el botón `View Storybook` para ver la última versión de su Storybook.

![Storybook link on Chromatic](/intro-to-storybook/chromatic-build-storybook-link.png)

Utilice el enlace y compártalo con los miembros de su equipo. Esto es útil como parte del proceso de desarrollo de aplicaciones estándar o simplemente para mostrar el trabajo 💅.
