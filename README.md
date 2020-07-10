# Pentaho-Almacen-de-Productos-DW

Se tiene un archivo excel con informacion de Pedidos, Clientes y Productos y se requiere extraer, limpiar y almacenar toda esa informacion en una base de datos MS SQL Server, desarrollando un proceso de ETL a traves de la herramienta Pentaho.



### Funcionamiento del Proceso:

Las interfaces principales del proyecto son 3: PEDIDO, CLIENTE y PRODUCTO.

El input del proceso se compone de 2 archivos Excel(xlsx). Un archivo llamado "Relaciones.xlsx" con 3 hojas que se corresponden con los Pedidos, Clientes y Productos y, por otro lado, se tiene un catalogo de Paises que utilizaremos como lookup para obtener los nombres completos de los paises a traves de su codigo de pais.

Las tablas que carga el proceso una vez completo son:
* PEDIDO
* CLIENTE
* PRODUCTO
* PROCESS_ERROR: Bitacora de errores
* LOGTABLE: Tabla creada por Pentaho donde almacena metricas de cada ejecucion

El proceso una vez ejecutado, lo primero que hace es crear(en caso de que no existiecen) las tablas PEDIDO, CLIENTE, PRODUCTO Y PROCESS_ERROR y el batch_id. Por cada ejecucion, el proceso genera un BATCH_ID que utilizaremos para realizar un seguimiento de las ejecuciones

Posterior a esto, comienza el proceso de ETL leyendo cada uno de los inputs files y le realiza un tratamiendo de limpieza de datos, es decir, quita caracteres erroneos, elimina campos que no necesita, reemplaza nulos por valores default, etc.

Una vez completado el paso de extraccion, se realiza la validacion, donde basicamente se valida cada registro de entrada y se aplican los codigos de errores en caso de ser necesario. Estos codigos de errores se aplican cuando un registro contiene informacion funcionalmente invalida, o contiene campos con valores que no corresponden. Aqui debemos distinguir 2 tipos de errores, los errores criticos y los warnings. Los errores criticos son aquellos en los que el registro es rechazado del flujo principal. Los warnings, en cambio, pueden continuar siendo parte del flujo principal de informacion valida. Independientemente del tipo de error, cada registro con error será insertado en la tabla PROCESS_ERROR y se guardará en un archivo sumarizado acorde a cada interfaz.

Luego de la validacion, viene el paso de Carga, donde se realiza el insert/update de cada interfaz y se realizan algunos pasos de transformacion menores como cruces con el catalogo de paises, agregado de nuevos campos, etc.

Posteriormente, se unen los archivos sumarizados de todas las interfaces y se unen en un solo reporte excel con el total de errores por tipo. Este reporte utiliza un template predefinido.

Finalmente, una vez completado el proceso correctamente, se envia una notificacion por correo electronico informando el status del mismo.

En caso de algun error en medio de la ejecucion del proceso, se realiza el borrado de la tabla PROCESS_ERROR a traves de batch_id generado en tiempo de ejecucion y se envia su correspondiente notificacion via e-mail



### Distribucion de las carpetas:
* "etl": Contiene tanto los jobs como las transformaciones utilizadas
* "datasource": Contiene los files que sirven como input del proceso
* "intermedite files": Contiene archivos intermedios del proceso
* "backup files": Contiene el backup de los archivos input del proceso por cada vez que se ejecuta el proceso
* "reportes": Contiene el reporte de cifras de control con los errores generados de forma sumarizada
* ".kettle": Contiene el archivo kettle.properties con las variables globadas utiliza el proceso

### Requisitos previos a ejecutar:

-Crear una base de datos llamada "bdAlmacen" en MS SQL Server 2014 o posterior.

-Abrir el archivo kettle.properties y editar las siguientes variables con la ruta de donde se haya descargado el proyecto en su computadora: 

* AI_DATASOURCE
* AI_PROJECT_DIR
* AI_REPORTE
* AI_ETL
* AI_SQL
* AI_INTERMEDIATE
* BD_MSSQL_USER: ingresar su usuario de conexion para MS SQL Server
* BD_MSSQL_PWD: ingresar la password de conexion para MS SQL Server

-Abrir el acceso directo "Spoon_Despacho_de_Pedidos.bat" y editar la siguiente linea con la ruta de donde se haya descargado el proyecto en su computadora: set KETTLE_HOME = < directorio > \ Proyecto Despacho de Pedidos

### ¿Como ejecutar el proyecto? 

Una vez completados los requisitos previos, ejecutar el acceso directo "Spoon_Despacho_de_Pedidos.bat". Una vez abierto Spoon, se debe abrir el job "main.kjb" ubicado dentro de la carpeta "etl" y ejecutarlo.
