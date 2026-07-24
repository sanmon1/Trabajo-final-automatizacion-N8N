# Trabajo-final-automatizacion-N8N

Gestión de Reseñas y Reclamos

Este proyecto automatiza la gestion de reseñas y reclamos de cliente que lleguen por mail. Sin esta automatizacion, un operador humano se tendria que hacer cargo. Perdiendo el tiempo redactando cada mail, su respuesta y viendo si se tiene que escalar el caso. Con esta automatizacion todo ocurre automaticamente solo con la intervencion de un operador humano en el caso que sea una urgencia.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Requitos

N8N / lo puede tener como localhost o pagando a n8n oficial
Airtable / Se requiere una cuenta de google y la API key para poder conectarlo.
Motor de IA / en este caso se usa groq ya que es la mejor opcion y mas economica. Requiere un Api key para conectarlo
Canal de Entrada / Se re quiere un mail o gmail para que pueda recibir los correos. Con su respectivo Clien ID y la Api key para poder conectarlo
Canal de salida / nuevamente se requiere un gmail que este conectado al workflow y telegram que este asociado a un numero en casos de urgencia. Para poder usar telegram se requiere crear un bot en la plataforma de telegram y asociarlo al numero de telefono para que envile las notificaciones

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Arquitectura del sistema

El sistema se divie en 2 flujos:

El flujo principal contiene

1. GmailTrigger-Detecta los email cada 5 minutos
2. IF Asunto - Filtra los email con palabras claves en el asunto(reclamo, queja, problema, reseña)
3. IF Campo - Valida que el email tenga cuerpo y remitente
4. Edit Fields (Set) — Normaliza los campos del email
5. Airtable Create — Registra la reseña con estado Pendiente
6. HTTP Request (Groq) — La IA analiza el sentimiento, categoría y urgencia
7. Airtable Create - En el caso que la Api de groq falle, se registra un error automatico en la tabla de errores
8. Code (JavaScript) — Parsea la respuesta JSON de Groq
9. Airtable Update — Actualiza el registro con el análisis de IA (estado: Procesado por IA)
10. IF Urgencia — Decide si el caso es urgente o no (en el caso que sea una urgencia se manda una notificacion al telegram y si es baja el bot contesta automaticamente y se actualiza estado a contestado por IA

Flujo Humant-in-the-loop

Webhook — Recibe la decisión del manager cuando hace clic en los links
IF Decisión — Evalúa si aprobó o decidió enviar manualmente
Aprobar - Gmail envía la respuesta al cliente y se actualiza el estado a enviado
Enviar manualmente - un operador humano se pone en contaco con el cliente y se actualiza estado a enviado manual

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Base de datos- Airtable

Tabla: Reseñas

Cliente_email - Email del remitente
Reseña ID - ID de la consulta
Canal - Origen de mensaje
Asunto - Asunto de mail
Texto_Original - Cuerpo del mail
Sentimiento - Forma en la que se dirije la otra persona
Categoria - Si es Reclamo / Sugerencia / Elogio
Urgencia - si es Bajo / Medio / Alto
Puntuacion_Negativo - puntuacion que asigna la IA para ver el estado del caso
Respuesta - Respuesta generado por IA
Estado - Si esta 	Pendiente / Procesado por IA / Aprobado por Humano / Enviado / Error
Fecha_Entrada - Fecha de creacion de solicitud
Fecha_cierre - Fecha de cierre de solicitud

Tabla: Errores

Error_ID - ID del error
Tipo_error - Tipo de error
Detalle - Descripcion del error 
Nodo_origen - En que nodo se origino el error
Fecha_error - Fecha del error
resuelto - Si se resolvio el error

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Documentacion de pruebas

Prueba 1 Email urgente con amenaza de denuncia 

Input:	Email con asunto "reclamo" y el mensaje es "El producto llegó roto, quiero devolución o los denuncio"
Comportamiento: Groq clasifica como Urgente / Alta, se registra en Airtable y se notifica al manager por Telegram para que tome una decision
Resultado: Groq asignó urgencia Alta, llegó la notificación a Telegram con los links de aprobación o mandar manual. Se aprobo la respuesta, se envia la respuesta al cliente y se actualiza el estado a Enviado

Prueba 2 

Input: Email el asunto "reseña" y mensaje: "Muy buen servicio, quedé muy conforme con la atención. Los recomiendo"
Comportamiento: Groq clasifica como Elogio / Baja. Se responde automáticamente al cliente
Resultado: Groq asignó urgencia Baja. Se respondió automáticamente al cliente sin que un operador humano intervenga y Airtable se actualizó a estado "Enviado"

Prueba 3

Input: Email el asunto "Hola" y mensaje: "Hola, como estas?"
Comportamiento: El IF 1 ignoro el mail por que tiene palabras claves
Resultado: El flujo ignoro el mail y no se creo ningun registro en airtable

Prueba 4

Input: Email con asunto "reclamo" y el cuerpo esta vacío
Comportamiento: El IF 2 detecta que el campo texto está vacío y el flujo no continúa
Resultado:l cliente de email rechazó el envío con error Client network socket disconnected before secure TLS connection was established — confirmando que un email sin cuerpo no puede procesarse. 

Prueba 5

Input: Email con el asunto "Hola, el producto me vino roto y lo quiero devolver para antes del fin de semana"
Comportamiento: El nodo de Http request se queda cargando buscando la url de groq. No la puede encontrar y se manda a la tabla errores
Resultado: Se registró en la tabla Errores con detalle: 404 - Unknown request URL


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Proyecto final - Gestión de Reseñas y Reclamos con IA
Por las dudas subi las 2 imagenes del diagrama de flujo el PNG y PDF

Dejo los links de Airtable y youtube donde subi el video demo

https://youtu.be/qOk4YYd04Xw
https://airtable.com/invite/l?inviteId=invpSUMM4TPEfc2by&inviteToken=b98336f29f2ba633fcf04770badce2522cb48e65bf164554459243277094fe6d&utm_medium=email&utm_source=product_team&utm_content=transactional-alerts

