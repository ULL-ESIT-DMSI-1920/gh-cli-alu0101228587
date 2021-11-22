# Github Cli
### Jaime Simeón Palomar Blumenthal - alu0101228587@ull.edu.es

Se debe de utilizar las funcionalidades de _gh_ para crear un repositorio, conectarse a él y eliminarlo; crear alias y funcionalidades nuevas utilizando nodeJS.

## Autenticación
En primer lugar, debemos generar un token en Github accediendo a _Settings > Developer Settings > Personal access token_.

![Personal access token](img/img1.png)

Una vez ahí, generamos un nuevo token con todos los permisos activos (por comodidad), y guardamos la configuración. Tendremos que copiar y guardar el token que hemos generado en algún fichero seguro, puesto que no podremos volver a obtenerlo una vez salgamos de la página actual.

Tras haberlo salvado, en la terminal modificaremos la variable local _GITHUB_TOKEN_ y la igualaremos a nuestro token:

```export GITHUB_TOKEN=ghp_a41qMU21gyhgLzwMZz6Gy434rzkQhvZ4dNzrm```

Finalmente nos dirigiremos a nuestro fichero _~/.profile_ en Ubuntu. Otras distribuciones pueden tener un fichero de configuración distinto. Al final del documento vamos a pegar la línea anterior para que se configure nuestro token cada vez que iniciemos la shell.

## Alias

Muchas veces queremos ejecutar comandos mútilples veces y estos son muy largos. Es por esto que disponemos del comando _gh alias_. La nomenclatura genérica para este comando es la siguiente.

```sh
gh alias set <alias> '<comando>'    # Para crear un alias.
gh alias delete <alias>             # Para eliminar un alias.
gh alias list                       # Para listar todos los alias.
```

### Alias para crear repositorios
Vamos a hacer un alias para crear un repositorio en una organización dada:
```sh 
gh alias set create-repo 'repo create "$2"/"$1"' #gh create-repo [repositorio] [organización o propietario]
```
Si ejecutamos el comando empleando el alias:

![Gh alias repo create](img/img2.png)

### Alias para eliminar repositorios
Vamos a generar una alias para eliminar un repositorio de una organización indicada:
```sh
gh alias set delete-repo 'api -X DELETE /repos/"$2"/"$1"' #gh delete-repo [repositorio] [organización o propietario]
```

### Alias para ver organizaciones
```sh
gh alias set org-list "api --paginate /user/memberships/orgs --jq '.[].organization | .login, .url'"
```
¿Qué hace cada una de las opciones y argumentos?

* **_--paginate_**: Junta todos los diferentes request que nos llegan desde el serivicio web.
* **_--jq_**: Filtra la entrada en JSON.
    * **_.[]_**: Devuelve todos los elementos del array del fichero Json.
    * **_.organization_**: Filtra los elementos que se llamen _organization_.  
    * **| _.login, .url_**: Indica que queremos los elementos _login y url_ de _organization_.

En resumen, '.[].organization | .login, .url' significa:
* Quiero todos los elementos del array que se llamen _organization_.
* Dentro de los elementos _organization_ quiero los atributos _login y url_.

### Extensiones GH
En numerosas ocasiones requeriremos de funcionalidades que de las que no dispone GH nativamente. Con los conocimientos suficientes podremos desarrollarlas y publicarlas nosotros mismos.

La única condición para publicarla una vez desarrollada es que el nombre de la extensión (y por ende, la forma en la que se invoca) tenga la forma _gh-[nombre_funcionalidad]_.

Para esta práctica, elaboraremos una extensión que nos liste los participantes en una organización y los repositorios en esta. La llamaremos _gh-members_.

En primer lugar, ejecutaremos _npm init_ para inicializar el directorio actual como proyecto node y configuraremos nuestro fichero _package.json_:

```json
{   // Información de la aplicación
  "name": "gh-members",
  "version": "1.1.0",
  "description": "This extension allows you to list every member and repository of a given organization",
  "main": "gh-members.js",
  "scripts": {
    "start": "node gh-members",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
    // Información del repositorio
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ULL-ESIT-DMSI-1920/gh-members.git"
  },
    // Información del autor
  "keywords": [gh, git, github, api, organization, owner, members, repositories],
  "author": "alu0101228587",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/ULL-ESIT-DMSI-1920/gh-members/issues"
  },
    // Dependencias principales
  "homepage": "https://github.com/ULL-ESIT-DMSI-1920/gh-members#readme",
  "dependencies": {
    "commander": "^8.3.0",
    "shelljs": "^0.8.4"
  }
}
```

Lo realmente importante de este fichero son las dependencias. Son librerías que utilizaremos para la ejecución de nuestro programa. En este fichero también especificaremos qué versiones queremos de estas dependencias.

Ahora, si ejecutamos _npm install_, NPM leerá las dependencias previamente especificadas e instalará esos módulos principales y los módulos secundarios que estos requieran para funcionar.

Todo lo instalado se encuentra en el directorio _node_modules_, y las versiones correspondientes en el fichero _package-lock.json_.

Seguidamente configuraremos un pequeño script de bash (_gh-members_) que compruebe si Node está instalado y localice la dirección absoluta del script JavaScript donde implementaremos todas las funcionalidades de la aplicación.

```sh
### Especificación del intérprete bash
#!/usr/bin/env bash
set -e

# Si Node no está instalado lanza un menzaje de error
# y finaliza la ejecución del programa
if ! type -p node >/dev/null; then
   echo "Node not found on the system" >&2
   exit 1
fi

# Calcula la dirección absoluta del script.
SCRIPT_DIR="$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"

# Ejecuta el script JavaScript pasándole cualquier argumento
${SCRIPT_DIR}/gh-members.js "$@"
```

Finalmente, pasaremos a implementar todas las funcionalidades en el fichero _gh-members.js_. En primer lugar instanciaremos los módulos que vayamos a utilizar y que previamente especificamos en las dependencias (_package.json_):

```js
const fs = require("fs");
const shell = require('shelljs');
const { Command } = require('commander');
const program = new Command();
```

Podemos llamar a cualquier método o atributo público de _program_, por ejemlo, de la siguiente manera: _program.[atributo o método]_.

Ahora debemos comprobar si _git_ y _gh_ están instalados en el sistema y, sino, lanzar un mensaje de error e interrumpir la ejecución:

```js
if (!shell.which('git')) {
	shell.echo("Git not installed.");
	shell.exit(1);
}

if (!shell.which('gh')) {
	shell.echo("gh not installed.");
	shell.exit(1);
}
```

Continuamos especificando las opciones con las que podremos ejecutar la aplicación. También le indicaremos a la aplicación que tiene que ir consumiendo estas opciones o, dicho de otro modo, parseando el array _argv_:

```js
program                                                     // Equivale a:
	.version(config.version)                                // progarm.version(config.version)
	.option('-r, --repo', 'List repositories of owner')     // program.option('-r, --repo', 'List repositories of owner')
	.option('-o, --owner <owner>', 'Specify owner')         // program.option('-o, --owner <owner>', 'Specify owner')

program.parse(process.argv);                                // Parsea argv.
```

Una cosa a tener en cuenta es que al utilizar el método _.option_ de _program_ (o el módulo Commander) se crean variables para cada opción cuyo nombre es el de la opción larga de esa opción: para la primera opción es _program.opts().repo_ y para la segunda _program.opts().owner_. Por mayor comodidad vamos a procesar _program.opts()_ y vamos a quedarnos con las dos variables _repo y owner_.

```js
const { repo, owner } = program.opts();
```

Finalmente implementaremos la lógica de la aplicación: Si no nos da un argumento, lanza la ayuda y termina la ejecución; si nos da el nombre de una organización (opción -o), devuelve la lista con sus miembros; y si le pasamos la opción -r también nos dará la lista de repositorios de esa organización.

```js
if (!owner) {
	console.log("Owner not specified. Sending help...");

	program.help();
	shell.exit(1);
}

console.log(`\n-- Members of ${owner} --\n`);
shell.exec(`gh api -X GET /orgs/${owner}/members --jq ".[].login"`);

if (repo) {
	console.log(`\n-- Repositories of ${owner} --\n`);
	shell.exec(`gh api -X GET /orgs/${owner}/repos --jq '.[].name'`);
}
```

El repo de la extensión tiene que ser un subrepo del de la práctica.

Hacemos gh extension create gh-nombre en el repo de la práctica
Nos movemos al directorio de la extensión y hacemos gh repo create --public ULL-ESIT-DMSI-1920/gh-nombre
git submodule add direccionhttps

Para clonar un super repo git clone --recurse-submodules direccion

Token de acceso: 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
export GITHUB_TOKEN=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

gh api -X GET /user/orgs

[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-f059dc9a6f8d3a56e377f745f24479a46679e63a5d9fe6f495e02850cd0d8118.svg)](https://classroom.github.com/online_ide?assignment_repo_id=6022595&assignment_repo_type=AssignmentRepo)
