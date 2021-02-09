Jira
===

Archivo **docker-compose.yml** que permite desplegar el servicio de **jira** como servidor.

Para la configuración del archivo se tuvieron varias consideraciones:

Primero crear un **volumen** para los datos de jira.

```
$PWD/jira-data:/var/atlassian/application-data/jira
````

Para que los datos de la configuración software se persistan como por ejemplo plugins y tener un punto de retorno.

Segundo, crear un segundo **contenedor** para almacenar los datos que son almacenados en jira, como por ejemplo los proyectos y los usuarios.

```
$PWD/pg_data:/var/lib/postgresql/data
````

Tercero, las **variables de entorno** necesarias para que el contenedor de **jira**, se conecte a una base de datos en el contenedor de **postresql**.

```
environment:
- TZ=America/Los_Angeles
- JVM_MINIMUM_MEMORY=4096m
- JVM_MAXIMUM_MEMORY=6144m
- JVM_RESERVED_CODE_CACHE_SIZE=512m
- ATL_JDBC_URL=jdbc:postgresql://postgres-j:5432/jiradb
- ATL_JDBC_USER=atlassian
- ATL_JDBC_PASSWORD=atlassian
- ATL_DB_DRIVER=org.postgresql.Driver
- ATL_DB_TYPE=postgres72
- ATL_TOMCAT_PORT=8080
- ATL_TOMCAT_CONTEXTPATH=/jira
```

Cuarto, se establece una política de reinicio de los contenedores, la cual permite que cada vez que inicie el servidor onpremise, los contenedores arranquen.

```
restart: always
```

Quinto, se estableció una **red con un adaptador bridge** para que los contenedores se puedan ver y conectar entre ellos.

Sexto, se recomienda el uso de un nuevo contenedor adminer que permita administrar de forma gráfica la base de datos de postgresql.

Las pruebas realizadas fueron las siguientes:

Desplegar los contenedores y verificar su funcionamiento, se hace destacar que el software de jira necesita al menos 2gb de ram para funcionar normalmente, con menos se pone muy lento.

Se utilizó una imagen con la versión 8.6.1, con el fin de agregar datos y luego, proceder a cambiar la versiòn de la imagen a la última y como resultado no hubo problemas con el cambio de versiòn, se actualizo satisfactoriamente.

Dentro del software de jira se utilizó la herramienta para hacer **backups y restaurarlos**, funciono correctamente, aspectos a tener en cuenta:

El backup queda almacenado dentro del contenedor en el siguiente directorio:

```
-/jira/jira-data/export/copia.zip
```

Para restaurarlo se debe mover el comprimido dentro del contenedor a la carpeta import, de la siguiente manera:

```
mv copia.zip ../import
```

Luego desde el panel de administraciòn se indica la ruta del archivo y se procede a restaurarla, este procedimiento fue exitoso.

Y por último pero no menos importante, en la documentaciòn oficial mencionan que los backups de jira no son muy exactos, que se recomienda que se contacte con los proveedores de bases de datos para realizar el procedimiento adecuado. Entonces se probó haciendo una copia o dump y restaurandola directamente en la base de datos del contenedor postgresql.

Este **comando** es para realizar la copia:

```
pg_dump -U atlassian -W -h localhost jiradb > jiradb.sql
```

y este para restaurarla:

```
psql -U atlassian -W -h localhost jiradb < jiradb.sql
```
