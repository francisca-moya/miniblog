# README #

---

Este repositorio contiene una pequeña aplicación web de prueba realizada originalmente con Silex (hoy migrada a Symfony 7) que usé como demostración en la [Cuarta BetaBeers de Córdoba](http://betabeers.com/event/4-betabeers-cordoba-626/). Trata de un blog muy simple, con un frontend y un backend sencillo para la creación de nuevas entradas.

La aplicación está basada en la aplicación [Bilbostack](https://github.com/javiereguiluz/bilbostack) realizada por [Javier Eguiluz](https://twitter.com/javiereguiluz), al cual agradezco que me haya dejado usar su material como apoyo a mi charla. Podéis encontrar [más material sobre Silex](http://symfony.es/noticias/2013/01/30/silex-desarrollo-web-%C3%A1gil-y-profesional-con-php/) en su página web.

**Requisitos**: PHP >= 8.2 con las extensiones `pdo_sqlite`, `ctype` e `iconv`, y [Composer](https://getcomposer.org).

**Cómo instalar el proyecto**

```
$ git clone https://github.com/mpgprofe/miniblog.git
$ cd miniblog
$ composer install
$ php bin/console create-schema
$ php bin/console app:create-user admin "Administrador"   # pide la contraseña (mín. 8 caracteres)
$ php -S localhost:8000 -t public
```

`create-schema` también actualiza bases de datos de versiones anteriores (las entradas
existentes quedan como publicadas). La base de datos SQLite se guarda en `var/schema.sqlite`
(el directorio `var/` debe ser escribible por el servidor web). Cambia `APP_SECRET` en un
`.env.local` antes de desplegar en producción.

**Usuarios y roles** (acceso en `/login`, panel en `/admin`)

| Rol | Puede |
|---|---|
| Alumno | Crear entradas como borrador o enviarlas a revisión; editar/borrar solo las suyas mientras no estén publicadas. |
| Docente | Todo lo anterior sobre cualquier entrada, y publicarlas o despublicarlas. |
| Administrador | Todo lo anterior, más gestionar usuarios y categorías. |

Para crear más usuarios desde la consola: `php bin/console app:create-user NOMBRE "Nombre completo" --role=ROLE_TEACHER`
(roles: `ROLE_ADMIN`, `ROLE_TEACHER`, `ROLE_STUDENT`). El administrador también puede hacerlo desde `/admin/users/`.

**Datos del centro y tutorial de acceso**

La dirección y el teléfono que aparecen en el pie y en `/contacto` se editan en
`config/packages/twig.yaml` (clave `school`). La primera vez que un visitante entra, un
minitutorial de 3 pasos explica cómo acceder; se puede reabrir con «¿Cómo accedo?».

**Editor, imágenes, páginas, buscador y RSS (v2.2)**

- El editor es CKEditor 5 (copia local en `public/vendor/ckeditor5`, licencia GPL-2+; ver `LICENSE.md`
  de esa carpeta). Las imágenes se suben desde el propio editor a `public/uploads/` (JPG, PNG, GIF o WebP,
  máx. 3 MB; si el servidor limita menos con `upload_max_filesize`/`post_max_size`, aplica el límite del servidor).
  **`public/uploads/` debe ser escribible por el servidor web**, y conviene incluirla en las copias de seguridad.
- Todo el HTML que se guarda se sanea en el servidor (se eliminan scripts, `onclick`, `javascript:`, iframes…).
- El administrador crea páginas fijas (horarios, normas…) en `/admin/pages/`; se muestran en `/p/<dirección>` y,
  si se marca, en el menú.
- Buscador (`/buscar?q=`), paginación de 10 en 10 y feed RSS en `/rss`.
- Tras actualizar desde una versión anterior, ejecuta `php bin/console create-schema`.

**Usuarios de demostración (para probar)**

Crea un alumno y un docente de prueba desde la consola. El comando pide la contraseña
(mínimo 8 caracteres, por eso no sirve `demo`; aquí se usa `demo1234`):

```
$ php bin/console app:create-user alumno.demo "Alumno Demo" --role=ROLE_STUDENT
$ php bin/console app:create-user docente.demo "Docente Demo" --role=ROLE_TEACHER
```

Para crearlos sin que pregunte (por ejemplo, en un script):

```
$ printf 'demo1234\n' | php bin/console app:create-user alumno.demo "Alumno Demo" --role=ROLE_STUDENT
$ printf 'demo1234\n' | php bin/console app:create-user docente.demo "Docente Demo" --role=ROLE_TEACHER
```

Con ellos puedes probar el flujo completo: entra como `alumno.demo`, escribe una entrada y envíala a
revisión; después entra como `docente.demo` (o como administrador), revísala y publícala.

> **Importante:** borra estos usuarios (en `/admin/users/`, como administrador) o cambia su contraseña
> antes de publicar el blog en internet. Contraseñas conocidas en un servidor público son un riesgo.

## Cómo he desplegado el proyecto

Para realizar esta práctica he utilizado Debian, Git y GitHub. He aprendido a descargar el proyecto, instalarlo, trabajar con ramas y guardar los cambios que he ido haciendo.

### 1.- Preparar el entorno

Primero instalé los programas necesarios:

    sudo apt update
    sudo apt install git openssh-client php-cli php-sqlite3 php-xml php-mbstring php-intl composer

- `sudo apt update`: actualiza la lista de paquetes.
- `sudo apt install`: instala los programas necesarios.

Después comprobé las versiones:

    git --version
    php -v
    composer --version

También configuré Git con mi nombre y correo:

    git config --global user.name "Francisca Moya"
    git config --global user.email "francisca.moya@proton.me"

### 2.- Crear mi fork y descargar el proyecto

Primero hice un fork del repositorio original en GitHub. Después descargué mi copia al ordenador:

    git clone git@github.com:francisca-moya/miniblog.git
    cd miniblog

- `git clone`: descarga el repositorio.
- `cd`: entra en la carpeta del proyecto.

Para comprobar los repositorios remotos utilicé:

    git remote -v

### 3.- Instalar y probar el proyecto

Instalé las dependencias y preparé la aplicación:

    composer install
    php bin/console create-schema
    php bin/console app:create-user admin "Administrador"
    php -S localhost:8000 -t public

- `composer install`: instala las dependencias.
- `create-schema`: prepara la base de datos.
- `app:create-user`: crea el usuario administrador.
- `php -S`: inicia el servidor local.

Después entré en `http://localhost:8000` para comprobar que la página funcionaba correctamente.

### 4.- Conectar con el repositorio original

Añadí el repositorio original como remoto con el nombre `upstream`:

    git remote add upstream https://github.com/mpgprofe/miniblog.git
    git remote -v

Utilicé estos comandos para obtener e integrar cambios:

    git fetch upstream
    git merge upstream/master

- `git fetch`: descarga información del repositorio remoto.
- `git merge`: integra los cambios en la rama actual.

### 5.- Crear una rama y añadir la licencia

Creé una rama nueva para añadir el archivo de licencia:

    git switch -c add-license

Después añadí el archivo y guardé los cambios:

    git add LICESE
    git commit -m "Archivo de licencia de uso"

Me equivoqué al escribir el nombre del archivo (`LICESE` en lugar de `LICENSE`), así que tuve que corregirlo:

    git mv LICESE LICENSE
    git commit -m "Fix: Nombre de archivo LICENSE"
    git push --set-upstream origin add-license

- `git switch -c`: crea una rama nueva.
- `git add`: prepara los cambios.
- `git commit`: guarda los cambios.
- `git mv`: cambia el nombre de un archivo.
- `git push`: sube los cambios a GitHub.

Después creé un Pull Request en mi propio fork, lo fusioné con `master` y eliminé la rama.

### 6.- Actualizar el repositorio local

Volví a `master` y descargué los cambios de GitHub:

    git switch master
    git pull origin master
    git branch -d add-license
    git log --oneline --graph

- `git switch master`: vuelve a la rama principal.
- `git pull`: actualiza el repositorio local.
- `git branch -d`: elimina una rama.
- `git log --oneline --graph`: muestra el historial de cambios.

### 7.- Practicar los conflictos de fusión

Para practicar los conflictos creé dos ramas:

    git switch -c telefono-a

Modifiqué el teléfono en el archivo `config/packages/twig.yaml` y guardé el cambio:

    git commit -am "Teléfono de secretaría"

Después creé la segunda rama:

    git switch master
    git switch -c telefono-b

Volví a modificar la misma línea con otro teléfono:

    git commit -am "Teléfono de jefatura"

Para provocar el conflicto fusioné las dos ramas:

    git switch master
    git merge telefono-a
    git merge telefono-b

Git detectó que las dos ramas habían modificado la misma línea.
Abrí el archivo para solucionar el problema:

    nano config/packages/twig.yaml

Elegí el teléfono que quería conservar y eliminé los marcadores del conflicto.
Finalmente guardé la solución:

    git add config/packages/twig.yaml
    git commit -m "Fusiona telefono-b resolviendo el conflicto"
    git log --oneline --graph --all

### 8.- Documentación y problemas encontrados

Durante la práctica también aprendí que el archivo `README.md` sirve para explicar de qué trata un proyecto, cómo instalarlo y cómo utilizarlo.
El texto se escribe en Markdown, un lenguaje de marcado muy sencillo:

- `#`: sirve para crear títulos.
- `##`: sirve para crear subtítulos.
- `*` o `-`: sirven para crear listas.
- `` ` ``: sirve para escribir comandos o código dentro de un texto.
- ` ``` `: permite crear bloques de código.

Los principales problemas que encontré fueron el error al escribir el nombre del archivo LICENSE y el conflicto que apareció al fusionar las dos ramas de los teléfonos.
Al final conseguí corregir el nombre del archivo, resolver el conflicto y completar las fusiones.
Con esta práctica he aprendido a utilizar Git y GitHub, trabajar con ramas, guardar cambios, crear Pull Requests y solucionar conflictos.

## Licencia

El proyecto incluye un archivo `LICENSE` en la raíz del repositorio, donde se especifican las condiciones de uso y distribución del código.
