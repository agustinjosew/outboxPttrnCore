# Que mostramos aqui:
Patron Outbox en conjunto con .NetCore y background workers en sistemas distribuidos.

## Que es el El patrón Outbox
También conocido como "Outbox Pattern", es una técnica utilizada en el desarrollo de sistemas distribuidos para garantizar la consistencia y la fiabilidad en la realización de operaciones secundarias o asíncronas. Su principal objetivo es manejar estas operaciones de manera confiable, incluso en entornos donde la aplicación principal pueda experimentar fallas o problemas.

La idea central de este patrón es mantener un registro de las operaciones secundarias pendientes en una aplicación. En lugar de ejecutar estas operaciones de inmediato, se registran de forma segura en una estructura de datos, generalmente una tabla de base de datos llamada "Outbox". Cada registro en esta tabla representa una operación secundaria que debe realizarse en algún momento.

Luego, un proceso o componente independiente, como un Background Worker o un servicio de fondo, monitorea constantemente esta tabla Outbox y ejecuta las operaciones pendientes de manera asincrónica. Esto asegura que las operaciones secundarias se realicen de manera confiable, incluso si la aplicación principal se bloquea, se detiene o experimenta problemas temporales.

Tamb ien este patrón es especialmente útil en sistemas distribuidos, donde la consistencia y la confiabilidad son esenciales. Permite que las operaciones secundarias, como el envío de correos electrónicos, la generación de eventos, la actualización de datos en otros sistemas, entre otras, se realicen de manera consistente, lo que contribuye a la robustez y la escalabilidad de la aplicación en entornos distribuidos.

## ¿Cuándo es bueno usar el Outbox Pattern?

El Outbox Pattern es útil en diversas situaciones, especialmente en aplicaciones distribuidas con alta concurrencia y múltiples servicios. Algunos escenarios en los que es recomendable utilizar el Outbox Pattern son:

* Envío de correos electrónicos y notificaciones: Cuando necesitas enviar correos electrónicos de confirmación, notificaciones o mensajes a los usuarios después de ciertas operaciones, como registros o compras, el Outbox Pattern garantiza que estos correos electrónicos se entreguen de manera confiable sin afectar el rendimiento de la operación principal.

* Generación de eventos y notificaciones: Si tu aplicación genera eventos para otros servicios o sistemas, el Outbox Pattern asegura que estos eventos se propaguen incluso si ocurre un fallo en la operación principal.

* Procesamiento asincrónico: Cuando una operación secundaria es asincrónica y no necesita completarse inmediatamente después de la operación principal, el Outbox Pattern te permite realizar el procesamiento en segundo plano sin afectar el flujo principal.

* Mejora de la escalabilidad: Utilizar un Background Worker para procesar los mensajes del Outbox permite liberar recursos de la aplicación principal, lo que contribuye a una mejor escalabilidad y rendimiento del sistema.

* Trazabilidad y seguimiento de eventos: Almacenar los mensajes del Outbox en una tabla de la base de datos permite rastrear las operaciones secundarias realizadas y tener un registro claro de los eventos procesados.

El Outbox Pattern es una valiosa técnica para garantizar la consistencia y confiabilidad en sistemas distribuidos, especialmente cuando se necesitan realizar operaciones secundarias asincrónicas o no transaccionales. Al separar las operaciones secundarias de la operación principal y utilizar un proceso en segundo plano para llevar a cabo estas tareas, podemos mejorar la robustez y escalabilidad de nuestras aplicaciones.

## Diagrama de implementación:
![Outbox](https://github.com/agustinjosew/outboxPttrnCore/assets/76487325/68abf9b5-9d07-46d5-bd2b-00b12774613e)

* **Cliente**: Representa el usuario o sistema que realiza una acción específica, en este caso, realiza una compra en la aplicación enviando una solicitud HTTP POST al API.

* **API**: Web API que recibe la solicitud de compra del cliente. En esta etapa, el API valida los datos recibidos, realiza las comprobaciones necesarias y, si todo está correcto, registra una "orden de cobro" que indica que se debe procesar un pago pendiente. Además, guarda el evento relacionado con la compra en la base de datos.

* **Base de datos**: Es la base de datos donde el API almacena tanto la orden de cobro como el evento relacionado con la compra. Es importante destacar que ambos registros se realizan en una sola transacción, lo que garantiza la consistencia y evita posibles problemas de integridad.
Respuesta al Cliente: El API envía una respuesta HTTP exitosa (código de estado 2xx) al cliente para indicar que la compra ha sido recibida y que el pago está pendiente.

* **Bucle del Procesamiento del Outbox**: Aquí comienza el proceso en segundo plano que se encarga de procesar los eventos pendientes, conocido como el Worker. Se inicia un bucle para revisar constantemente si existen mensajes en el "Outbox" (un registro en la base de datos que contiene eventos pendientes).

* **Worker**: Representa el proceso en segundo plano que realiza el procesamiento de los mensajes pendientes del "Outbox". En este caso, busca los mensajes pendientes en la base de datos.
Consulta de mensajes pendientes: El Worker consulta la base de datos para obtener los mensajes pendientes de procesar.

* **Procesamiento del mensaje**: Una vez que el Worker obtiene un mensaje del "Outbox", procede a procesarlo. En este punto, el Worker puede interactuar con sistemas externos u otros servicios según sea necesario para completar la operación secundaria asociada al mensaje.

* _**Nota sobre el Worker**: El Worker tiene la flexibilidad de interactuar con sistemas externos o realizar acciones adicionales según la naturaleza del evento. Esto permite que el procesamiento de eventos secundarios sea independiente y escalable._

* **Actualización de la base de datos**: Después de procesar con éxito el mensaje, el Worker actualiza el estado del mensaje en la base de datos para indicar que ha sido procesado satisfactoriamente.

* **Fin del bucle del Procesamiento del Outbox**: Una vez que el Worker ha procesado todos los mensajes pendientes en el "Outbox", finaliza el bucle y espera el próximo ciclo para verificar nuevamente si hay nuevos mensajes pendientes.

El diagrama representa cómo funciona el patrón Outbox en una aplicación: 
- Cuando un cliente realiza una acción que genera un evento secundario (como una compra), el evento se registra inicialmente en el "Outbox" de la base de datos, lo que garantiza la consistencia y durabilidad. 
- Luego, un proceso en segundo plano (el Worker) se encarga de procesar los eventos pendientes, lo que permite realizar acciones secundarias de manera asíncrona y confiable sin afectar la operación principal. Esto mejora la escalabilidad y la confiabilidad del sistema, ya que el cliente recibe una respuesta inmediata mientras que las tareas adicionales se procesan de manera asincrónica y robusta.