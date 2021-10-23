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
Vamos a hacer un alias para crear repositorios en la organización _ULL_ESIT_DMSI_1920_:
```sh 
gh alias set create-repo 'repo create ULL-ESIT-DMSI-1920/"$1"'
```
Si ejecutamos el comando empleando el alias:

![Gh alias repo create](img/img2.png)



Token de acceso: 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
export GITHUB_TOKEN=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

[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-f059dc9a6f8d3a56e377f745f24479a46679e63a5d9fe6f495e02850cd0d8118.svg)](https://classroom.github.com/online_ide?assignment_repo_id=6022595&assignment_repo_type=AssignmentRepo)
