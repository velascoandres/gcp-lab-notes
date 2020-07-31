
Ver las zonas


```shell script
    gcloud compute zones list | grep us-central1
```

Configurar una zona

```shell script
    gcloud config set compute/zone us-central1-b
```


Crear una maquina virtual con:

* tipo de maquina: `n1-standard-1`.
* image-project: `debian-cloud`.
* image: `debian-9-stretch-v20170918`.
* subnet: `default`.

```shell script
gcloud compute instances create "my-vm-2" \
--machine-type "n1-standard-1" \
--image-project "debian-cloud" \
--image "debian-9-stretch-v20190213" \
--subnet "default"

```

Desde SSH  conectarse a my-vm-2

```shell script
    ping my-vm-1
```

Conectarse a my-vm-1

Instalar un servidor web:

```shell script
    sudo apt-get install nginx-light -y
```


Modificar su pagina web

```shell script
sudo nano /var/www/html/index.nginx-debian.html
```


curl http://localhost/


Desde my-vm-2

curl http://my-vm-1/





## Cloud Storage and Cloud-SQL


Crear un bucket

### Desde la consola de GCP:

Primero exportamos la ubicacion:

```shell script
export LOCATION=US
```

Creamos el bucket en donde el nombre de este sera con nuestro `projectID`:

```shell script
    gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```


> `$DEVSHELL_PROJECT_ID` esto ya esta por defecto exportado.


Recuperamos una imagen desde otro bucket y la renombramos

```shell script
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
```

La imagen que recuperamos la vamos a guardar en nuestro bucket:

```shell script
    gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```


Modificamos la lista de acceso a la imagen que hemos almacenado:

```shell script
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```


## Crear una instancia de Cloud SQL

Podemos ver que tipos de instancias hay:

```shell script
gcloud sql tiers list
```

> Los valores que comienzen con `db-`, deberian ser escogidos para crear instancias de segunda generacion.

Comando generico para crear instancias:

```shell script
gcloud sql instances create [INSTANCE_NAME] --tier=[MACHINE_TYPE] --region=[REGION]

```
Existen mas parametros de configuracion como el tipo de almacenamiento (SDD, HDD), capacidad del almacenamiento entro otros (revisar la [documentacion](https://cloud.google.com/sql/docs/mysql/create-instance#gcloud)). 


```shell script
gcloud sql instances create instance1 --tier=db-n1-standard-2 --region=europe-west2
```

## GKE

> Desde el Cloud Shell

Exportamos una zona:

```shell script
export MY_ZONE=us-central1-a
```

Empecamos un Cluster de Kubernets el cual lo llamamos `webfrontend` y lo configuramos para dos nodos:

```shell script
gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
```

Luego verificamos que el cluster haya sido creado:

```shell script
kubectl version
```


### Ejecutar y desplegar el contenedor

> Desde el Cloud Shell

Desplegamos un servidor web llamado `nginx`:

```shell script
kubectl create deploy nginx --image=nginx:1.17.10
```
 Ver el pod en ejecucion:

 ```shell script
 kubectl get pods
 ```

 Exponemos el contenedor en el internet:

 ```shell script
kubectl expose deployment nginx --port 80 --type LoadBalancer
 ```

 Ver el servicio:

 ```shell script
 kubectl get services
 ```

 Podemos escalar el numero de pods:

 ```shell script
 kubectl scale deployment nginx --replicas 3
 ```

 Confirmamos que el numero de pods cambio:

 ```shell script
 kubectl get pods
 ```

 Confirmamos que el servicio web no cambio:

 ```shell script
 kubectl get services
 ```


 ## App Engine

> Desde el Cloud Shell

 Inicializamos la aplicacion con el id del proyecto:

 ```shell script
 gcloud app create --project=$DEVSHELL_PROJECT_ID
 ```


Clonamos el repositorio de muestra:

```shell script
git clone https://github.com/GoogleCloudPlatform/python-docs-samples
```


Navegamos hacia el directorio de la aplicacion clonada:

```shell script
cd python-docs-samples/appengine/standard_python37/hello_world
```

### Correr la aplicacion

Actualizamos el sistema:

```shell script
sudo apt-get update
```

Establecemos un entorno virtual:

```shell script
sudo apt-get install virtualenv
virtualenv -p python3 venv
```

Activamos el entorno virtual

```shell script
source venv/bin/activate
```

Dentro del directorio del proyecto actualizamos las dependencias y corremos la aplicacion:

```shell script
pip install  -r requirements.txt
python main.py
```


### Desplegar la aplicacion

Dentro del directorio del proyecto:


```shell script
gcloud app deploy
```
Desplegamos al navegador para poder ver la ejecucio real de la aplicacion en la nube:

```shell script
gcloud app browse
```



## Deployment manager and cloud Monitoring


```shell script
export MY_ZONE=us-central1-a
```

Descargamos el template `ymal`

```shell script
gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
```


Reemplazamos los placeholders en el archivo `mydeploy.yaml`

```shell script
sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
```

El archivo deberia verse asi:

```ymal
 resources:
  - name: my-vm
    type: compute.v1.instance
    properties:
      zone: us-central1-a
      machineType: zones/us-central1-a/machineTypes/n1-standard-1
      metadata:
        items:
        - key: startup-script
          value: "apt-get update"
      disks:
      - deviceName: boot
        type: PERSISTENT
        boot: true
        autoDelete: true
        initializeParams:
          sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-9-stretch-v20180806
      networkInterfaces:
      - network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-dcdf854d278b50cd/global/networks/default
        accessConfigs:
        - name: External NAT
          type: ONE_TO_ONE_NAT
```

Realizamos el despliegue desde el template


```shell script
gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml
```


Podemos actualizar el despliegue del archivo actualizando los scripts de inicio:


```shell script
nano mydeploy.yaml
```

En este caso se agrego el comando para instalar un servidor servidor web

```
     value: "apt-get update; apt-get install nginx-light -y"
```

```shell script
gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml
```

