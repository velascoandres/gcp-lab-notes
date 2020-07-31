# Laboratorios: Getting Started with application development.

## App Dev - Setting up a Development Environment: Node.js


 ### 1. Crear una VM usando la consola de google cloud
    * Nombre: `dev-instance`
    * Region: `us-central1`
    * Zona: `us-central1-a`
    * Permitir el accesso : `Allow full access to all cloud APIs`
    * Permitir el trafico: `http`

### 2. Instalar software dentro de la VM

    Instalar dependencias:

    ```shell script
        sudo apt-get update
    ```

    ```shell script
        sudo apt-get install git
    ```

    Instalamos `node.js`: 

    ```
        curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
        sudo apt install nodejs
    ```

### 3. Configurar la VM para poder ejecutar una aplicacion `node.js`.

    Verificamos la version de `node.js`

    ```shell script
        node -v
    ```

    Clonamos el repositorio con el codigo de la aplicacion de practica:

    ```shell script
        git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    ```

        

    Creamos un enlace suave como acceso directo al directorio de trabajo.

    ```shell script
    ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/devenv ~/devenv
    ```

    Dentro del directorio que contiene los archivos de practica ejecutamos el siguiente comando:

    ```shell script
    cd ~/devenv
    ```

    Ejectuamos la aplicacion:


    ```shell script
       sudo node server/app.js  
    ```

    ```shell script
        npm install
        node list-gce-instances.js
    ```
