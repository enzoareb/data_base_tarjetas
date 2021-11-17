= Bases De Datos : Sistema de pago con tarjetas de credito

{docdate}

:title-page:
:numbered:
:source-highlighter: coderay
:tabsize: 4

== Introducción


El trabajo práctico tiene como objetivo implementar una pequeña base de datos encargada de almacenar y consultar 
informacion relacionada a la compras realizadas con  tarjetas de credito por distintos clientes, como estas 
pueden o no interactuar con los mismo y los diferentes negocios involucrados en el sistema. Contaremos con las tablas _cliente_, _tarjeta_, _comercio_, _compra_, _rechazo_, _cierre_, _cabecera_, _detalle_, _alerta_, y por ultimo _consumo_.
Las tablas _cliente_, _tarjeta_, _comercio_ y _cierre_ contendran informacion desde su creacion. Las demas iran almacenando informacion dinamicamente dada diferentes relaciones y respetando pautas entre las tablas ya cargadas.


== Descripción

La aplicacion cuenta con un menu principal donde podemos acceder a las distintas opciones para cargar datos y consultarlos a traves de una base de datos Postgres, como asi tambien utilizar una aproximacion al uso de datos en una BD nosql con el fin de comparar ambos sistemas de persistencia

== Implementación

La implementacion del programa se lleva a cabo sobre el archivo _main.go_. Aqui se encuentran las funciones necesarias 
para la creacion de la Base de Datos, las tablas, la implementacion y eliminacion de las llaves primarias y foraneas, 
con las mismas tambien podemos cargar las tablas mencionadas y crear los storage procedures y triggers. Por separado
podemos cargar la tabla _consumo_ y usar una funcion para aprobar o no esos consumos.

- Tablas que se crearan en la base de datos.  
 * cliente.
 * tarjeta.
 * comercio.
 * compra.
 * rechazo.
 * cierre.
 * cabecera.
 * detalle.
 * alerta.
 * consumo.

- Llaves primarias y foraneas que se crearan en cada tabla.

 * Cliente. _nrocliente_ [PK].
 * Tarjeta. _nrotarjeta_ [PK], _nrocliente_ [FK].
 * Comercio. _nrocomercio_ [PK].
 * Compra. _nrooperacion_ [PK], _nrotarjeta_, [FK] _nrocomercio_ [FK].
 * Rechazo. _nrorechazo_ [PK], _nrotarjeta_ [FK], _nrocomercio_ [FK].
 * Cierre. _año_ _mes_ _terminacion_ [PK].
 * Cabecera. _nroresumen_ [PK], _nrotarjeta_ [FK].
 * Detalle. _nroresumen_  _nrolinea_ [PK], _nroresumen_ [FK]
 * Alerta. _nroalerta_[PK], _nrotarjeta_ [FK], _nrorechazo_ [FK].

- Funciones con las que contamos en la interfaz de usuario*.

 * `crearDataBase()`
 * `crearTablas()`
 * `crearPKsFKs()`
 * `borrarPKsFKs()`
 * `cargarTablas()`
 * `cargarConsumos()`
 * `crearSPyTriggers()`
 * `pasarCosasAcompraORechazo()`
 * `generarResumenes()`
 * `MainNOSQL()`
 * `salir()`

crearDataBase()::
Crea la base de datos.
crearTablas()::
Crea las tablas necesarias para la correcta administracion de informacion: _cliente_, _tarjeta_, _comercio_, _compra_, _rechazo_, _cierre_, _cabecera_, _detalle_, _alerta_.
crearPKsFKs()::
Asigna a cada tabla creada las llaves primarias y foraneas necesarias con respecto a los campos que poseen y la relaciones entre ellas.
borrarPKsFKs()::
Elimina las llaves primarias y foraneas de las tablas existentes. Esto lo hace en un orden especifico ya que si se borran en cualquier orden perjudica a las tablas que dependen de la llave que pudo haber sido borrada.
cargarTablas():: 
Carga las tablas con la informacion tanto de los clientes, de las tarjetas asociadas a cada uno de ellos, comercios, y cierres.
cargarConsumos()::
Carga informacion en la tabla _consumo_, cada consumo tiene un _nrotarjeta_ (valido o no), un _codseguridad_ (codigo de seguridad asociado a la tarjeta, puede ser correcto o incorrecto), _nrocomercio_(numero de comercio existente o no) y un _monto_(monto del consumo).
crearSPyTriggers()::
Crea las funciones en go que crean los Storage Procedures y disparan los triggers necesarios para la correcta administracion de los datos que fueron ingresados, excepto el Storage Procedure `cargarCierresSP()` que fue creado cuando se ejecutó la funcion `cargarTablas()` y `generar_resumenes()` que sera creada y ejecutada en el momento cuando el usuario presione la opcion 9 del menu.

- Storage Procedures que se crean en SQL al ejecutar esta funcion (hay más que se crearon antes o despues (2 para ser exactos `cargarCierresSP()` y `generar_resumenes()`)):


* `autorizar_compra` (la crea la funcion `autorizarCompraSP()` de go ). 

* `simular_pasar_consumos_a_compra_o_rechazo()` (la crea la funcion `simularPasarConsumosAcompraORechazoSP()` de go)

* `compras_pendientes_de_pago()` (la crea la funcion `comprasPendientesDePagoSP() de go`). 

* `generacion_de_resumen()` (la crea la funcion generarResumenesSP() de go)

* `generar_resumenes()` (la crea la funcion generarResumenes() de go tiene muchos `generacion_de_resumen()`).


* `t_a()` (la crea la funcion `alertaClienteTrigger()`)

* `alertas_a_compras()` (la crea la funcion `alertasComprasTrigger()`)

- Triggers que se crean en SQL. 
* `t_a_trigger`
* `alertas_a_compras_trg`

pasarCosasAcompraORechazo()::
Evalua todos los consumos que estan cargados en la tabla _consumo_, cada uno pasa por revision del SP `autorizar_compra`, si un consumo es invalido no pasa a la tabla compra y pasa a la tabla rechazo a la vez que la se genera un nuevo registro en la tabla _alerta_ con esos datos.
Si un _consumo_ es valido, pasa a la tabla _compra_. 
generarResumen()::
carga en las tablas cabecera y detalle los datos correspondientes con respecto a las compras realizadas por un cliente en un periodo determinado, estas peticiones de resumenes seran ejecutadas cuando en el momento se cree  el SP `generar_resumenes()` que dentro hace la llamada a `generacion_de_resumen()` de un cliente determinado. 

MainNOSQL()::
Mediante un archivo externo de go que conecta con una base de datos NoSQL se ingresaran datos a mano de 3 clientes, 3 tarjetas y 3 compras en 3 distintos objetos, se vera el resultado de estos objetos en la terminal.


=== Explicacion de SP en _negocio.sql_

cargar_cierres()::
Se encarga de ingresar datos a la tabla _cierre_ dinamicamente iniciando en una fecha.

autorizar_compra()::
	Se tiene consumos, cada consumo para pasar a ser una compra debe antes pasar por una autorizacion, esta funcion se encarga de hacer brindar esa autorizacion.
	Toma como parametros dos char's de maximo 16 caracteres, y dos enteros y devuelve un booleano. El primer parametro lo nombramos _i_n_tarjeta_, el segundo _i_cod_seguridad_, el tercero _i_n_comercio_ y el _cuarto i_monto_.
	Devuelve true si una tarjeta es valida y false si no lo es.
	- ¿En que se basa para determinar que una tarjeta sea valida?: 
	
	* Una tarjeta es valida cuando el codigo de tarjeta existe en la base de datos, el codigo asociado a esa tarjeta el correcto && cuando el estado de esta tarjeta no es _vencida_, _anulada_, _suspendida_, y _el monto total de compras sin pagar + i_monto < el limite de compra de la tarjeta_ (encargado de chequearlo el SP `compras_pendientes_de_pago`). Sacamos el numero de la tarjeta del parametro _i_n_tarjeta_, el codigo de la tarjeta del parametro _i_cod_seguridad_ y verificamos si no sobrepasa, sumandole _i_monto_ ,al monto total registrado hasta la fecha en compras no pagadas de esa tarjeta.

compras_pendientes_de_pago::
Es una funcion interna que utiliza `autorizar_compra()`.
Esta funcion toma como parametro un char de 16 caracteres al que denominamos _i_nrotarjeta_ y devuelve un entero. 
Se encarga de sumar los montos de todas las compras sin abonar que realizo una determinada tarjeta ingresada como parametro. 
Esta funcion por si sola no tiene mucho uso, es necesaria para evaluar si el monto total adeudado no excede el limite permitido.

simular_pasar_consumos_a_compra_o_rechazo()::
Esta funcion no recibe parametros y no devuelve nada.
Se utiliza para recorrer la tabla _consumos_ y ver uno por uno si va a parar a la tabla _compra_ o a la tabla _rechazo_. ¿Cuales son las condiciones para que un consumo sea una compra y vaya a _compra_?
Dado un consumo (_nrotarjeta_, _codseguridad_, _nrocomercio_, _monto_) se *verifica* que la tarjeta sea valida con `autorizar_compra()` se le pasa el _nrotarjeta_, _codseguridad_ y _monto_ del consumo, si autorizar_compra con esos parametros da true es una compra valida y se completa un registro de la tabla _compra_ con los datos del consumo i, si da false no lo es y va a parar a la tabla _rechazo_ de la misma forma ademas de que se generan alerta por fraude que se disparan por triggers.

t_a():: es un SP que se ejecuta cada vez que se ingresa un nuevo registro a la tabla rechazo mediante el trigger `t_a_trigger`, ademas de chequear en el interior de la funcion si la tarjeta que ingresa a rechazo ya posee dos o mas rechazos en el mismo dia Y ademas se genero rechazo por 'limte de compra', si es asi genera una alerta con el codigo de alerta 32 y descripcion de la alerta 'limite', si no es asi el codigo de alerta sera 0 y la descripcion sera 'rechazo'.
alertas_a_compras()::
Es un SP que alerta cada vez que se genera una nueva compra mediante el trigger `alertas_a_compras_trg`, generara el 
alerta
corespondiente si detecta 2 compras realizadas en menos de 1 min en comercios de la misma localidad o si las 
compras evaluadas
tienen un lapso menor a 5min en locales de distinta localidad 

generacion_de_resumen::
Esta funcion recibe  un numero de cliente y un periodo del año.
Se encarga de completar los datos en las tablas correspondientes (_cabecera_ y _detalle_)teniendo en cuenta todas las compras realizadas por el cliente, dado el ultimo digito de la tarjeta, el año y el mes que recibe la funcion como parametros. Entraran las compras que hizo durante la fecha valida de cierre de la tarjeta.

=== Explicacion de Triggers en _negocio.sql_
t_a_trigger()::
Se ejecuta luego de que se ingrese algun registro nuevo a la tabla _rechazo_, ejecuta el SP `t_a`
alertas_a_compras_trg()::
Se ejecuta luego de que se ingrese algun registro nuevo a la tabla _compra_, ejecuta el SP `alertas_a_compras`.


