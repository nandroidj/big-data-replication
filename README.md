# big-data-replication


Implementar Replicación en BD MongoDB

1.  Implementar en MongoDB un ReplicaSet con 3 servidores que contengan la
información de la BD Finanzas. Un nodo Primary, un secondary y un arbiter.

```
mongod --replSet rs --dbpath ~/03-big-data/projects/big-data-replication/db/0 --port 27017 --oplogSize 50
mongod --replSet rs --dbpath ~/03-big-data/projects/big-data-replication/db/1 --port 27018 --oplogSize 50
mongod --replSet rs --dbpath ~/03-big-data/projects/big-data-replication/db/2 --port 27019 --oplogSize 50

mongo --port 27017
	
cfg = {
  _id:"GVDrs",
  members:[
    {_id:0, host:"mongo1:27017"},
    {_id:1, host:"mongo2:27018"},
    {_id:2, host:"mongo3:27019", arbiterOnly:true}
  ]
};

rs.initiate(cfg)

rs.initiate( {
   _id : "rs",
   members: [
      { _id: 0, host: "localhost:27017" }
   ]
})
```

2.  Conectarse al Nodo PRIMARY

```
mongo --port 27017 
```


3.  Crear la db finanzas.

```
use finanzas
```

4.  Ejecutar el script facts.js 4 veces para crear volumen de datos.

```
load("facts.js") //x4
```

5.  Buscar los datos insertados, en el nodo PRIMARY.

```
db.facturas.insert({X:1})
db.facturas.count()
```

6.  Buscar los datos insertados, en el nodo SECONDARY.

```
mongo --port 27018
	use finanzas
	rs.slaveOk
	db.facturas.count()
```

7.  Realizar un ejemplo de Fault Tolerance simulando una caída del Servidor PRIMARY.

  Se ejecuta el comando *//CTRL+C* en la terminal del primary donde está corriendo el *rs*. 


  1.  Explicar que sucedió.
      
      La shell del servidor SECONDARY pasa a ser el PRIMARY.

  2.  Verificar el estado de cada servidor.

      `mongo --port 27017`

  3.  Insertar un nuevo documento.

      ```
        use finanzas
        db.facturas.insert({X:1})
      ```

  4.  Levantar el servidor caído.

      ```
        mongod --replSet rs --dbpath c:\data\db\rs\3 --port 27020 --smallfiles  --oplogSize 50 --nojournal
      ```

  5.  Validar la información en cada servidor.


8.  Agregar un nuevo nodo con slaveDelay de 120 segundos.

    ```
      mongo --port 27020
      rs.add({host:"localhost:27003", slaveDelay: 120})
    ```

9.  Ejecutar nuevamente el script facts.js, asegurarse antes de ejecutarlo que el nodo con
slaveDelay esté actualizado igual que el PRIMARY.

  `mongo --port 27020 rs.secondaryOk()`

  1.  Luego de ejecutado chequear el SECONDARY.

    ```
      //  Agrego el delay propuesto en el punto 8
      rs.add({host:"localhost:27020", priority:0, slaveDelay: 120})

      // Compruebo la conexion
      rs.conf()

      // Compruebo sync
      show.dbs
    ```

  2.  Consultar el nuevo nodo y ver cuando se actualizan los datos.
    
    ```
      db.facturas.count()
      load("facts.js")


