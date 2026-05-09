# ECO-RIDE
Proyecto de Desarrollo Full Stack I

PROYECTO: ECO-RIDE

1.Definición del Negocio

Optimiza la movilidad urbana mediante el uso compartido de vehículos eléctricos. Resuelve la falta de transporte sostenible, permitiendo a los usuarios localizar vehículos cercanos, gestionar la carga de baterías para asegurar la viabilidad del trayecto y procesar cobros automáticos basados en la distancia y el impacto ambiental ahorrado.

2.Arquitectura de Microservicios

 //BENJA(2.1_2.3)
 
  2.1 Registro de usuario. 

  2.2 Login de Usuario.

  2.3 Gestión de datos personales, licencias de conducir y documentos de identidad.
  
 //VICTOR(2.4_2.6)
 
  2.4 Catálogo de vehículos, nivel de batería, modelo y estado técnico.

  2.5 Rastreo de coordenadas (Latitud/Longitud) en tiempo real de los autos activos.

  2.6 Orquestación de la solicitud: busca conductor, calcula ruta y gestiona el viaje.
  
 //SERGIO(2.7_2.9)
 
  2.7 Calcula la tarifa dinámica según demanda, hora y tipo de vehículo.

  2.8 Manejo de billetera virtual, recarga de saldo y procesamiento del cobro post-viaje.

  2.9 Alertas o emails (Viaje confirmado, Conductor llegó, Recibo de pago).
  
 //VEREMOS()
 
  2.10 Sistema de calificación (1-5 estrellas) y comentarios para conductores y pasajeros.

  2.11 Alertas de revisión técnica basadas en los kilómetros recorridos y salud de la batería.

  2.12 MS-Usuario
  
3.Desacoplamiento y Comunicación

Se utilizará API REST e  implementará el principio de base de datos independiente. Cada servicio tendrá su propia MARIADB. Ningún servicio puede acceder a tablas ajenas; la información se solicita mediante endpoints JSON.

4.Estructura de Datos:

Servicio de Usuarios: Entidad Usuario con atributos id (Long), nombreUsuario (String), correoElectronico (String), contrasena (String), rol (String), activo (Boolean).

Servicio de Perfiles: Entidad Perfil con atributos id (Long), usuarioId (Long), nombre (String), apellido (String), telefono (String), numeroLicencia (String), fechaNacimiento (LocalDate). 

Servicio de Vehículos: Entidad Vehiculo con atributos id (Long), patente (String), marca (String), modelo (String), nivelBateria (Integer), estado (String). 

Servicio de Geolocalización: Entidad Ubicacion con atributos id (Long), vehiculoId (Long), latitud (Double), longitud (Double), ultimaActualizacion (LocalDateTime). 

Servicio de Viajes: Entidad Viaje con atributos id (Long), pasajeroId (Long), conductorId (Long), vehiculoId (Long), puntoOrigen (String), puntoDestino (String), estadoViaje (String), fechaInicio (LocalDateTime), fechaFin (LocalDateTime). 

Servicio de Precios: Entidad Tarifa con atributos id (Long), precioBase (Double), costoPorKilometro (Double), factorDemanda (Double), descuentoEco (Double). 

Servicio de Pagos: Entidad Transaccion con atributos id (Long), viajeId (Long), billeteraId (Long), monto (Double), metodoPago (String), estadoPago (String). 

Servicio de Notificaciones: Entidad Notificacion con atributos id (Long), usuarioId (Long), titulo (String), mensaje (String), leido (Boolean), fechaEnvio (LocalDateTime). 

Servicio de Calificaciones: Entidad Calificacion con atributos id (Long), viajeId (Long), emisorId (Long), receptorId (Long), estrellas (Integer), comentario (String). 

Servicio de Mantenimiento: Entidad RegistroMantenimiento con atributos id (Long), vehiculoId (Long), tipoServicio (String), fechaServicio (LocalDate), costo (Double), observaciones (String).

5. Reglas de negocio

Validación de Autonomía de Batería: Antes de que el Servicio de Viajes confirme una solicitud, debe consultar al Servicio de Vehículos para asegurar que el nivelBateria sea superior al 15%, bloqueando la asignación si la carga es insuficiente. 

Validación de Deuda Pendiente: El Servicio de Pagos debe verificar que el pasajero no tenga transacciones en estado PENDIENTE antes de permitir la creación de un nuevo registro en el Servicio de Viajes. 

Control de Disponibilidad de Vehículo: Al iniciar un viaje, el Servicio de Vehículos debe cambiar el estado del móvil de DISPONIBLE a EN_USO de forma atómica para evitar que dos pasajeros reserven el mismo auto simultáneamente. 

Validación de Documentación Vigente: El Servicio de Perfiles debe rechazar el cambio de estado a "Activo" de cualquier conductor cuyo numeroLicencia no esté cargado o cuya fecha de vigencia haya expirado.

 Sincronización de Geolocalización: El Servicio de Viajes solo permitirá marcar un viaje como COMPLETADO si las coordenadas del Servicio de Geolocalización se encuentran dentro de un radio de 100 metros del puntoDestino registrado. 
 
Cálculo Dinámico de Tarifa: El Servicio de Precios debe aplicar obligatoriamente el factorDemanda si el horario actual coincide con las "horas punta" definidas en el sistema, aumentando el precioBase automáticamente. 

Restricción de Calificación: El Servicio de Calificaciones solo permitirá crear un registro si el viajeId referenciado existe y tiene el estadoViaje como COMPLETADO.

Alerta de Mantenimiento Preventivo: El Servicio de Mantenimiento debe disparar una notificación automática a través del Servicio de Notificaciones cuando un vehículo alcance un total acumulado de 5,000 kilómetros recorridos.

Seguridad de Transacción: El Servicio de Pagos debe validar que el monto de una transacción de tipo PAGO_VIAJE coincida exactamente con el montoTotal calculado previamente por el Servicio de Precios. 
 
Validación de Roles: El Servicio de Usuarios debe interceptar cualquier petición a los servicios de Precios o Mantenimiento para asegurar que el rol del usuario sea estrictamente ADMIN.
