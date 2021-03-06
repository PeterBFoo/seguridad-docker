## Seguridad en Docker

#### Seguridad del host

No se debe conceder de forma inconsciente permisos de superusuario a todos los usuarios existentes, ya que Docker accede al kernel. Por lo tanto, hay que crear un grupo de docker para limitar el acceso:

- Crear grupo de docker ->
    newgrp docker

- Conceder acceso al usuario ->
    sudo usermod -aG docker $USER username

---

#### Seguridad en el deamon

El demonio que utiliza docker escucha las peticiones a la API y maneja los objetos de Docker que serían las imágenes, contenedores, volúmenes y redes.

Al ser algo tan importante, también es necesario poner seguridad en él, ya que corre como superusuario. Hay que comprobar que el archivo daemon.json se encuentre en la ruta 'etc/docker/' (en caso contrario, crearlo):

```json
{
  debug : false
}
```

De esta forma, el demonio correrá como si estuviera en producción. También sería de interés configurar el parámetro ulimits que permite definir el peso de los archivos que pueden cargar los contenedores y dejar unos límites por defecto, además de crear un archivo key.json para indicar que ningún usuario que no sea root pueda acceder a él.

---

#### Seguridad en contenedores

Se puede limitar la capacidad de la imagen que se descargue, para así mejorar una gestión de almacenamiento y evitar la descarga de imágenes sospechosamente pesadas:

  docker run --ulimit nofile=512:512 --rm debian sh -c "unlimit -n"

Donde 512:512 representan 512MB de descarga máxima de imágenes.

También es altamente recomendado limitar el alcance de un ataque DDoS que saturaría el contenedor. Por lo tanto se pueden limitar los recursos para evitar que se sature todo el equipo:

```
docker run -it --cpus=".5" ubuntu /bin/bash
```
Donde --cpus=".5" hace referencia al porcentaje de rendimiento de la CPU.

#### Privilegios

Usualmente por defecto el usuario es root pero se puede cambiar el usuario por defecto al correr la imagen y podemos forzar a ello con el flag `-u uuid`

``` 
docker run -u 4400 alpine ls /root
```

Si se corre un contenedor con el propio usuario, dará un error ya que root es el único con permisos para hacerlo.

Con el flag --privileged se otorgan permisos de root:

    ```
    docker run -it --privileged ubuntu
    mount -t tmpfs none /mnt
    `df -h`
    ```
Sin la flag --privileged, sería incapaz de montar por temas de permisos.

#### Seguridad en la imagen

#### Trivy

Trivy es un escáner de vulnerabilidad / configuración incorrecta simple y completo para contenedores y otros artefactos. Una vulnerabilidad de software es una falla, falla o debilidad presente en el software o en un sistema operativo. Trivy detecta vulnerabilidades de paquetes de SO (Alpine, RHEL, CentOS, etc.) y paquetes específicos del idioma (Bundler, Composer, npm, yarn, etc.). Además, Trivy analiza archivos de infraestructura como código (IaC) como Terraform y Kubernetes para detectar posibles problemas de configuración que exponen sus implementaciones al riesgo de ataques. Trivyes fácil de usar Simplemente instale el binario y estará listo para escanear. Todo lo que necesita hacer para escanear es especificar un objetivo, como el nombre de una imagen del contenedor.

Para instalar Trivy:

- En el caso de no tener wget, introducir el siguiente comando:

    sudo apt-get install wget apt-transport-hhtps gnupg lsb-release
    
- Seguidamente:

    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    sudo apt-get install trivy

Antes de instalar Trivy, se recomienda usar **apt-get update** para actualizar la información sobre paquetes y dependencias, para descargar la última versión estable.

Ahora ejecutando:

    trivy image <imagen>:<versión>
    
Muestra las posibles vulnerabilidades de la versión de la imagen.

<img width="829" alt="Captura de pantalla 2022-05-19 a las 22 28 21" src="https://user-images.githubusercontent.com/91556382/169398900-58c4d569-e049-4680-b7b8-ae3c8c2f7640.png">




