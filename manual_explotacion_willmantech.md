# Manual de Explotación del Sistema ERP WillmanTech S.L.

### Alberto Rueda Romero

## 1. Introducción y Arquitectura del Sistema

Este manual describe los procedimientos técnicos y operativos necesarios para la
explotación del sistema ERP/CRM implantado en **WillmanTech S.L.**, empresa ficticia creada para el desarrollo de esta actividad.
Su audiencia objetivo son los administradoresde sistemas, el equipo de desarrollo (DAM/DAW) y los usuarios funcionales con
perfil técnico.

El documento cubre los aspectos de despliegue, configuración, seguridad,
mantenimiento y generación de informes, siguiendo los principios de calidad
documental definidos en la norma **ISO/IEC/IEEE 26514:2022**.

### 1.1 Módulos activados en el ERP

El sistema ERP de WillmanTech S.L. tiene activados los siguientes módulos
funcionales:

- Ventas = Gestión de presupuestos, pedidos y tarifas de clientes

- Facturación = Emisión, validación y contabilización de facturas  

- CRM = Gestión del pipeline de oportunidades comerciales        

- Inventario =  Control de existencias, movimientos y almacenes    

- Contactos = Directorio centralizado de clientes y proveedores    

- Calendario = Planificación de actividades, reuniones y seguimientos  

- Conversaciones = Mensajería interna, chatter y notificaciones del sistema

La interconexión entre módulos refleja el flujo real de trabajo de la empresa:
cuando un comercial cierra una oportunidad en el modulo **CRM**, el sistema genera
automáticamente un presupuesto en gracias al módulo **ventas**, al confirmarse ese presupuesto,
En el módulo **inventario** registra el movimiento de mercancía correspondiente y
el módulo **facturación** emite el documento contable asociado. Por su parte, en
**contactos** se centraliza los datos de clientes y proveedores, evitando
duplicidades entre módulos. El **calendario** se integra en el día a día comercial
permitiendo programar llamadas, visitas y seguimientos directamente desde una
oportunidad o un pedido. El módulo de **conversaciones** vertebra la comunicación del equipo:
cada registro del sistema lleva incorporado un hilo de mensajes donde los usuarios
 comentan, adjuntan documentos y reciben avisos automáticos
sin salir de la aplicación.

### 1.2 Topología lógica del despliegue

El sistema está desplegado mediante Docker Compose, lo que permite reproducir
el entorno en cualquier máquina anfitrión con Docker instalado.
La infraestructura se compone de dos contenedores y dos volúmenes
persistentes orquestados dentro de una red interna aislada.

#### Los componentes son:

- **`odoo`:** Contenedor principal que ejecuta la aplicación Odoo en su versión
  más reciente disponible (`latest`). Sirve la interfaz web en el puerto
  `8200/tcp` del host (mapeado internamente al `8069` del contenedor) y aloja
  el motor de generación de informes (QWeb + wkhtmltopdf). 

- **`db`:** Contenedor de base de datos que ejecuta PostgreSQL 16. Almacena
  todos los datos relacionales del ERP. Su puerto `5432` no se expone fuera
  de la red Docker interna. 

- **`odoo-data`:** Volumen persistente que conserva documentos adjuntos, imágenes de
 productos y reportes generados de Odoo:

- **`db-data`:** Volumen persistente que almacena los archivos de datos de
  PostgreSQL, garantizando que la información no se pierde al reiniciar o
  recrear los contenedores.

## 2. Guía de Instalación y Reinstalación

### 2.1 Requisitos previos

- Docker Desktop : Instalamos Docker Desktop desde el navegador en nuestrro PC

- Docker Compose : Necesitamos un documento yml para desplegar los contenedores, 
 en nuestro caso se nos fué proporcionao

- Puerto de red = 8200/tcp accesible 

### 2.2 Dependencias del SGBD

El sistema usa **PostgreSQL 16**. El usuario necesita permisos de
creación de bases de datos  y la codificación del servidor debe
ser **UTF-8**. En el despliegue dockerizado esto se configura automáticamente
mediante las variables de entorno del **docker-compose.yml**.

### 2.3 Credenciales del entorno


- `POSTGRES_USER`    | `odoo` | Usuario de PostgreSQL         
- `POSTGRES_PASSWORD`| `odoo` | Contraseña de base de datos    
- `POSTGRES_DB`      | `odoo` | Nombre de la base de datos     
- `HOST`             | `db`   | Hostname del contenedor de DB  

### 2.4 Instalación desde cero

Una vez instalado docker, abrimos el documento docker-compose.yml en nuestro IDE,
Abrimos la terminal y ejecutamos **docker compose up -d** para arrancar los contenedores
y verificamos su funcionamiento ejecuntando **docker compose ps**.
Una vez hecho esto, abrimos el localhost con el puerto indicado en el navegaor e instalamos los módulos
que estamos usando.

### 3. Seguridad y Control de Acceso:

Contamos con una serie de roles en la empresa (Administrador, Comercial 01, Contable y Operario de Almacén)

- Administrador: Tiene permisos de administrador y puede crear productos, es el único que puede acceder a todo. Además, en cuanto a Seguridad, Puede habilitar autenticación de dos factores, claves API o de Acceso. EL resto de usuarios incluido Administrador, también pueden cambiar la contraseña en aspectos de Seguridad.

- Comercial 01: Rol de usuario, solo puede ver los productos creados, mostrar documentos propios de ventas y no tiene más permisos
debido  a su rol

- Contable: Rol de usuario que solo puede ver los productos creados y tiene permisos para acceder a la facturación y validar
la cuenta bancaria

- Operario de almacén: Tiene permisos de usuario, solo puede ver la lista de productos

## 4. Procedimiento de Backup y Restauración

Ejecutar en PowerShell:

docker exec db pg_dump -U odoo -d odoo -F c -f /tmp/backup_odoo.dump
docker cp db:/tmp/backup_odoo.dump .\backups\backup_odoo.dump

Este comandoentra dentro del contenedor db y ejecuta pg_dump, que es la herramienta de PostgreSQL para hacer volcados de base de datos. Le dice que use el usuario odoo, que vuelque la base de datos odoo, que lo guarde en formato comprimido (-F c) y que el archivo de salida se llame backup_odoo.dump dentro de la carpeta /tmp del propio contenedor. Posteriormente El archivo en backups.

## 5. Flujo Operativo de Facturación e Informes

### 5.1 Generación de una factura

En primer lugar accedemos al módulo de facturación, entramos en clientes, facturas
y generamos uno nuevo, añadimos el cliente y los productos, confirmamos la factura y seguidamente
se asigna el número de factura que imprimiremos o enviaremos consiguiendo así nuestro PDF

### 5.2 Pipeline de generación del PDF

Cuando el usuario pulsa **Imprimir**, el sistema recoge los datos de la factura y
los introduce dentro de una plantilla, con esto, esl sistema crea una página web con 
el aspecto de la factura. Esa página pasa por **wkhtmltopdf** que la procesa y la convierte en
PDF
