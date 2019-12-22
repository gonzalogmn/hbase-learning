# HBase-learning

Slides disponibles [aquí](https://next-ggimeno.github.io/hbase-learning/).


## Requisitos

* Para las pruebas en local, utilizaremos [este repositorio](https://github.com/dajobe/hbase-docker) para generar un contenedor de Docker con HBase.
    * Bajar repositorio.
    * Ejecutar `docker build -t dajobe/hbase .` en la carpeta del repositorio, para generar la imagen.
    * Ejecutar `./start-hbase.sh` para arrancar el contenedor a partir de la imagen.
    * Para abrir una shell en el contenedor, ejecutar `docker exec -it <CONTAINER_ID> bash`
    * Una vez abierta la shell, ejecutar `hbase shell` para abrir la shell de HBase.


## Slides

Generadas con [remark](https://github.com/gnab/remark).

Para ejecutarlas en local, iniciar servidor http, como por ejemplo `python3 -m http.server` (ver [enlace](https://github.com/gnab/remark/wiki#external-markdown)).




