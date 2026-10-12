# CEH Manager en la nube de Amazon (AWS)

Guía para instalar CEH Manager **sin servidor** en la cuenta de AWS de tu comité. No hay equipo que mantener y, con el uso de un comité, el gasto queda cerca de cero. **Cada comité usa su propia cuenta: nadie más ve sus datos**, tampoco el proyecto.

Archivos de esta carpeta:

| Archivo | Para qué |
|---|---|
| [`ceh.yaml`](ceh.yaml) | Plantilla de **CloudFormation**: crea todo lo necesario en tu cuenta con unos pocos clics |

> **Antes de usarlo con datos reales**, hacé una prueba completa con datos de demostración en una cuenta de prueba (paso 6). Si no sos una persona con un mínimo de perfil técnico, instalar en un equipo propio (ver el [README](../README.md#cómo-empezar)) es más simple de entender.

## Qué se crea en tu cuenta

- Un **bucket S3 privado**, cifrado y con versionado, con los datos y las copias. **No se borra si eliminás la pila.**
- Una **tabla de DynamoDB** con el chat, las conversaciones del asistente y el registro de cambios de la base. **No se borra si eliminás la pila.**
- La **función Lambda** (ARM) con el programa y una **dirección HTTPS** propia.
- Una **tarea cada 5 minutos** (copias automáticas, recordatorios, vencimientos) y una **limpieza semanal** de registros vencidos (solo actúa si el administrador la activó).
- Un **presupuesto** con aviso por correo si el gasto se acerca al límite.
- Permisos mínimos: la función solo toca su bucket, su tabla, su clave y su propia configuración.

No crea redes (VPC), API Gateway, CloudFront ni bases de datos con servidor: son piezas con costo fijo que el sistema no necesita.

## Antes de empezar

1. **Cuenta de AWS** con verificación en dos pasos (MFA) en el usuario raíz. Para el día a día no uses el raíz: creá un usuario administrador aparte.
2. **Región de la Unión Europea** (por ejemplo `eu-west-1`, Irlanda). Para la ley argentina de protección de datos la UE es destino adecuado; Estados Unidos y Brasil no.
3. **El plan gratuito de las cuentas nuevas vence** (según fuentes de terceros, a los 6 meses o al agotar el crédito). Pasado ese plazo hay que pasar la cuenta al plan pago o AWS puede cerrarla. Poné un recordatorio.
4. **Tené a mano papel y lapicera**: vas a anotar la frase maestra y el código de recuperación.

## Instalación

### 1. Bajá la plantilla

Descargá [`ceh.yaml`](ceh.yaml) (botón *Raw* → guardar como). Es la misma para todas las versiones.

### 2. Anotá los datos de la versión

Cada versión publica dos datos que la plantilla te pide: el **enlace** al paquete de AWS y su **huella SHA-256**.

- Entrá a la [última versión](https://github.com/ceh-teams/ceh-releases/releases/latest).
- Abrí `latest.json`, buscá el bloque con `"os": "lambda"` y copiá su `url` y su `sha256`.

Para la **4.3.0** son:

| Dato | Valor |
|---|---|
| VersionCEH | `4.3.0` |
| CodigoURL | `https://github.com/ceh-teams/ceh-releases/releases/download/v4.3.0/ceh-manager_4.3.0_lambda_arm64.zip` |
| CodigoSHA256 | `b4c6059ce790fdd8dedab0a52f2258a73ac9f53f03b7cc6aa5eecf47fee0a30c` |

Si usás una versión más nueva, tomá los datos de **su** `latest.json`, no de esta tabla. Si la huella no coincide con la del paquete, la instalación se cancela y no queda nada.

### 3. Creá la pila

1. En la consola de AWS, arriba a la derecha, elegí la región de la UE.
2. Abrí **CloudFormation → Crear pila → Con recursos nuevos (estándar) → Cargar un archivo de plantilla** y subí `ceh.yaml`.
3. Completá los parámetros:

| Parámetro | Qué poner |
|---|---|
| **VersionCEH**, **CodigoURL**, **CodigoSHA256** | Los del paso 2 |
| **CorreoAlertas** | A dónde avisa el presupuesto |
| **PresupuestoUSD** | Dejá `1` |
| **ModeloBedrock** | Dejalo **vacío**. El asistente de IA se activa después desde el propio sistema (paso 5) |
| **RegionBedrock** | Dejá `eu-central-1` |
| **ConcurrenciaMaxima** | Dejá `0`. Si AWS te lo permite, un valor chico (por ejemplo `10`) pone un techo al gasto si alguien martilla la dirección |

4. Aceptá que la plantilla crea roles de IAM y creá la pila. Tarda unos minutos.

### 4. Abrí el sistema

Cuando la pila diga `CREATE_COMPLETE`, en la pestaña **Salidas** copiá `DireccionDelSistema` y `TokenInstalacion`. Abrí la dirección agregándole `instalar?token=` y el valor de `TokenInstalacion` (`https://…lambda-url…/instalar?token=…`): aparece el asistente de instalación. Sin el código no deja instalar; es de un solo uso y trátalo como una clave. Requiere la versión 4.3.1 o posterior.

Completalo y **guardá en papel, en dos lugares distintos, la frase maestra y el código de recuperación**. Son la única forma de sacar tus datos de AWS o de recuperarlos si algo falla; nadie del proyecto puede ayudarte sin ellos. Desde ese momento el sistema se abre solo en cada arranque.

### 5. Activá las opciones (si querés)

Con el administrador, en el menú **Configuración del sistema → Integraciones**:

- **Correo**: para que el sistema mande avisos.
- **Copiloto (inteligencia artificial)**: cargá la URL del proyecto y la clave de API de [Scaleway](https://www.scaleway.com/en/security-and-compliance/) (París, sin retención de lo que se consulta).
- **Búsqueda en internet de médicos e instituciones** (versión 4.3.0 o posterior): claves de uno o más buscadores (Exa, Linkup, Brave Search, Tavily), todos con plan gratuito. Funciona a través del copiloto y **solo envía una especialidad y un lugar** (o el nombre y la ciudad de una institución cargada), nunca datos de pacientes ni nombres de personas. Varios buscadores están en EE. UU.: cargá solo los datos de contacto que publica el propio profesional o su institución.

Todo es opcional: sin claves, esas funciones no existen y no sale nada a internet.

### 6. Primera prueba

Antes de cargar datos reales, en una **cuenta de prueba**: creá la pila, instalá, cargá un par de registros, abrí el sistema desde dos navegadores a la vez, usá el chat, generá una copia, actualizá y volvé atrás. Revisá en **Budgets** y **Cost Explorer** que el gasto es el esperado.

## Seguridad: lo que cambia y lo que no

- **Se mantiene:** los datos personales de terceros siguen cifrados por campo con la clave del comité; en el bucket solo hay datos cifrados. Las copias se abren solo con tu frase o tu código.
- **Cambia:** en un equipo propio, apagarlo deja los datos ilegibles hasta que alguien escribe la frase. En AWS la clave de apertura vive en un **parámetro cifrado de SSM** (`/ceh/<nombre de la pila>/boveda`) de **tu** cuenta, para que el sistema se abra solo. **Quien tenga permiso de AWS para leerlo puede descifrar todo.** Por eso: protegé la cuenta con MFA, no compartas credenciales y, si querés ver quién lo lee, activá CloudTrail (los eventos de administración son gratis).
- **El bucket nunca es público:** acceso público bloqueado y solo HTTPS.

## Costos

Estimación con el uso de un comité. **Verificá los precios en la página de AWS.**

| Pieza | Costo esperado |
|---|---|
| Lambda | Dentro del nivel gratuito permanente |
| S3 (base, archivos, copias) | Centavos por mes |
| DynamoDB (10/10 provisionada) | Dentro del nivel gratuito permanente, según fuentes de terceros |
| Copiloto y buscadores (opcionales) | Se pagan a Scaleway y a cada buscador (todos tienen plan gratuito con cuota); es lo único que crece con el uso |
| SSM, Presupuesto | Gratis |
| CloudWatch Logs (7 días) | Centavos o gratis |

Lo que puede subir el gasto: muchísimos pedidos (usá **ConcurrenciaMaxima** y la alarma del presupuesto) y adjuntos o copias muy grandes.

## Límites que conviene conocer

- **Adjuntos:** una función de Lambda acepta pedidos de hasta unos 6 MB: **una foto o un archivo de más de unos 4 MB no se puede subir.**
- **Restaurar una copia grande** por el navegador no funciona: subí el `.cehbk` al bucket, a la carpeta `copias/`, y restauralo desde *Mantenimiento → Restaurar* («…o una copia guardada»).
- **Tareas programadas:** cada 5 minutos, así que un recordatorio puede llegar hasta 5 minutos tarde.
- **Arranque en frío:** el primer pedido después de un rato sin uso tarda unos segundos.

## Copias de seguridad

Se guardan **en el bucket** (`copias/`). Descargá una copia a tu equipo de vez en cuando y guardala en otro lugar: **el bucket y la cuenta son un único punto de falla.** El bucket tiene versionado: cada versión de los archivos importantes se conserva 30 días.

## Actualizar

### Con un clic

En *Mantenimiento → Actualizaciones* el sistema avisa si hay versión nueva (lee un archivo público **firmado**, sin enviar datos de tu comité) y ofrece **Actualizar**: baja el paquete, verifica firma y huella, guarda un punto de restauración y cambia de versión. **Volver atrás** deja código y datos como estaban al actualizar (se pierde lo cargado desde entonces).

### Cuando hay que actualizar la plantilla

Si una versión lo pide (`requiere_stack`), el botón no aparece. También conviene hacerlo si instalaste **antes de la 4.3.0**: la plantilla nueva agrega el permiso para que el copiloto con Scaleway responda en segundo plano y la limpieza semanal de registros vencidos. Para actualizar la pila:

1. Bajá la [`ceh.yaml`](ceh.yaml) actual.
2. En CloudFormation elegí tu pila → **Actualizar → Reemplazar plantilla actual** y subila.
3. En **VersionCEH**, **CodigoURL** y **CodigoSHA256** poné los de la versión **que está instalada o una posterior** (si no, la pila vuelve la función a la versión de sus parámetros).
4. Revisá los cambios que muestra CloudFormation y confirmá.

Antes de actualizar, generá una copia de seguridad.

## Pasar entre un equipo propio y la nube

Los datos viajan en una copia `.cehbk`: generala en el origen, subila al bucket (`copias/`) del destino y restaurala desde *Mantenimiento*. Necesitás la frase o el código **del origen**. Sirve en las cuatro direcciones (equipo ↔ nube).

## Desinstalar

1. Descargá una copia y verificá que se abre.
2. Borrá la pila de CloudFormation. **El bucket y la tabla no se borran** (tienen tus datos): vacialos y eliminalos a mano cuando estés seguro (la tabla tiene protección contra borrado: desactivala antes).
3. Borrá el parámetro `/ceh/<nombre de la pila>/boveda` de SSM.

## Si algo falla

| Síntoma | Qué hacer |
|---|---|
| La pila falla con «La huella del paquete no coincide» | Copiaste mal la huella o el enlace, o son de otra versión: volvé a tomarlos del `latest.json` de la versión. No queda nada instalado |
| Error 503 «El sistema no está disponible por un momento» | No se pudo leer el bucket o la tabla. Reintentá; si sigue, mirá los registros de CloudWatch de la función |
| Pantalla de desbloqueo | La clave de SSM no existe o no abre esta bóveda (por ejemplo, tras restaurar una copia de otra instalación). Escribí la frase o el código: el sistema la repara solo |
| El copiloto dice que la respuesta se cortó | Volvé a preguntar; si se repite, mirá CloudWatch |
| La actualización falló a mitad | Si falló antes de terminar, sigue la versión anterior. Si algo anda mal después, usá *Volver atrás* |

## Aviso

Este material se entrega «tal cual», sin garantías. El comité es responsable de su cuenta de AWS, de sus datos y de inscribir la base ante la AAIP (ver el [README](../README.md#inscripción-ante-la-aaip)). El código y las decisiones técnicas están en el repositorio privado de la organización (se puede pedir acceso desde el README).
