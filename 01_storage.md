# Storage engine

El motor de almacenamiento es el componente del gestor de base de datos responsable de
gestionar como los datos son almacenados tanto en memoria como en disco. MongoDB soporta
varios motores de almacenamiento, usando por defecto WiredTiger.

- WiredTiger (por defecto para cualquier implementación Community o Enterprise)
- In-memory (disponible solo en Enterprise)

## WiredTiger Storage Engine 

Motor por defecto desde la versión 3.2, diseñado para la mayoría de cargas. Proporciona entre
otras características, un modelo de concurrencia a nivel de documento.

Se puede verificar el motor desde:

use admin
db.serverStatus().storageEngine.name // Devuelve el nombre del motor

En la versión Enterprise, WiredTiger soporta también encriptación.

## Snapshots and Checkpoints

WiredTiger utiliza el control de simultaneidad de conversión múltiple (MVCC). Al comienzo
de una operación de escritura, WiredTiger proporciona una instantanea de un punto en el
tiempo de los datos de la operación.

WiredTiger crea puntos de control con las operaciones de escritura en la llamada Vista Compartida con
los datos de las operaciones entrantes, realizando una instantanea con cada nueva operación. Cada 60 segundos
a partir de la información almacenada en cada instanea desde la vez anterior persiste en disco
las operaciones.

Como el proceso flush se produce cada 60 segundos, para no perder las operaciones entre estos intervalos si
hubiera una interrupción en el servidor, WiredTiger dispone de una vista previa llamada vista privada donde
las operaciones son enviadas cada 200ms al journal.

Si hubiera una interrupción, una vez que se recupere el servidor, WiredTiger usará la información del journal
para recuperar las operaciones que no se hayan persistido desde el último flush registrado desde la vista compartida,
en un proceso totalmente transparente para el DBA.

## Características del Journal

Los archivos del Journal tiene un tamaño máximo de 100MB, y cuando se supere el tamaño de ese
archivo, WiredTiger crea un nuevo archivo para continuar almacenando la información.

De manera automática WiredTiger ira borrando los antiguos ficheros del journal que ya no sean 
necesarios para restablecer las operaciones desde el último checkpoint usado en la vista compartida.

## Uso de memoria ¡Ojo certificación!

WiredTiger usa la memoria RAM de dos formas:

- WiredTiger internal caché. 
- Puede usar tambien el filesystem cache de memoria RAM de la máquina del servidor de manera elástica, es decir
siempre que no la necesite el servidor para otros procesos.

¿Cuanta memoria usará WiredTiger?

El mayor de los siguientes valores:

- 50% de (RAM - 1GB) => (RAM(EN GB) - 1) * 0,5
ó
- 256 MB



