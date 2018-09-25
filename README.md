
# Arquitectura 2.0

En la versión 2 de la arquitectura he decidido simplificar la infraestructura **prescindiendo del segundo nivel de containerización (docker).**

Por ser muy *problemático e inestable* y por **no aportar grandes ventajas por el momento**. Las aplicaciones c onstan por lo general de base de datos, API GraphQL y APP Node. Por lo que actualmente se utiliza una imagen de Alpine Linux con Node preinstalado para crear los contenedores.
	
## Visión general

La capa "física" de la arquitectura, si es que podemos llamarla así, consiste en un VPS de OVH corriendo Ubuntu Server 18.04 (BIONIC).

Internamente las aplicaciones son levantadas en contenedores LXC/LXD. Cada contenedor es como una máquina independiente con su propia dirección IP dentro de la red de LXD. Esto hace posible la comunicacion completa dentro del vps.

## Reverse Proxy

Para el direccionamiento entrante existe un NGINX escuchando el puerto 80 (redirección 301) y el puerto SSL 443. *(Ver /etc/nginx/sites-available/default)*

El nginx redirige el tráfico filtrando por URLs. Cuando se crea un nuevo contenedor, deberemos modificar el fichero anterior para añadir los servidores virtuales pertinentes (generalmente uno en el puerto 443 con la url base de la web y que redirije a la IP del contenedor).

## CDN CloudFlare

Acualmente todo el tráfico dirigido al VPS (marcosrgz.com) pasa por CloudFlare. Desde el panel de administración manejamos los dominios. Si se crea un nuevo contenedor para alojar aplicaciones web, deberemos crear las entradas necesarias para que el nuevo dominio apunte a la IP del VPS. Luego el NGINX hace la
parte interna de redirección.

## Certificados y SSL/TLS

Cloudflare nos proporciona un certificado firmado por una entidad certificadora de primer nivel. Para habilitar SSL debemos activar la opción "FULL" en el apartado SSL de cloudflare.

De esta manera CF recibe en puerto 443 y envía al 443. Con el SSL Flexible (por defecto), de CF todo sale hacia el puerto 80, lo que implica que entre cloudflare y nosotros la información va sin cifrar. 

Para esta comunicación utilizamos un **certificado autofirmado**, ya que no es necesario que lo firme una entidad certificadora. Lo demás es configuración en el VPS y en NGINX

## Despliegue
Aunque esto pueda variar de cara al futuro, se propone la estructura de directorios siguiente para la ubicación de las aplicaciones dentro de cada contenedor.

El contenedor deberá tener **3 directorios** dentro de /root, que son: **apps, backups y deploy.**

### Directorio /root/apps
Dentro de este directorio, existirán N subdirectorios correspondientes a proyectos web. De momento se alojará un proyecto por contenedor, pero
es posible que de cara al futuro exista más de uno. Adicionalmente se ubicará la aplicación esclava de DPL (explicación en breve)

### Directorio /root/apps/:proyecto
Dentro de cada directorio de proyecto, se ubicarán los directorios de cada subproyecto. Siempre habrá al menos un subproyecto para 
respetar la misma topología de directorios. Ejemplos son: /root/apps/:proyecto/api y /root/apps/:proyecto/app.

### Directorio /root/backups
Aquí se guardan los backups de las aplicaciones. Antes de desplegar una nueva versión de una aplicación debemos crear un tar.gz de la misma
en este directorio. IMPORTANTE: Para evitar el desaprovechamiento masivo del espacio, las aplicaciones deben desplegarse junto a su package.json de manera que 
una vez en el servidor, podamos instalar los módulos de node necesarios en el entorno de producción. A la hora de realizar este backup, borraremos node_modules.
Si fuese necesario cargar un backup, al existir el package.json siempre podremos reinstalar dichos paquetes.

## DPL

Como parte de la nueva arquitectura, estoy desarrollando una **infraestructura cliente servidor para automatizar el despliegue de aplicaciones**. La idea básica (y que contar con que aún no he pensado profundamente en los detalles, es montar una micro API en cada contenedor LXC, y una aplicación web que nos permitirá hacer entre otras, las siguientes cosas:

 - Creación, Parada y Ejecución de Contenedores, así como configuración de los mismos.
 
 - Despliegue de proyectos mediante peticiones JSON sencillas.
 - Monitorización del estado de los contenedores así como de las   aplicaciones que hay en ellos.

La idea básica es que la web (dpl-app) envíe peticiones HTTP/JSON a la API (dpl-api). Esta a su vez guarda un registro de los contenedores existentes, de losproyectos en cada contenedor y de las aplicaciones en cada proyecto. De esta manera podremos saber rápidamente cómo acceder a una aplicación de producción, así como realizar acciones rápidas con unos pocos clicks.

La API se comunicará con las apis esclavas (existe una y solo una por cada contenedor LXC) y estas realizarán las acciones necesarias. Como ejemplo de ello tenemosel despliegue. Desde el cliente crearemos un nuevo contenedor, Crearemos un nuevo proyecto y en el las aplicaciones necesarias. También indicaremos un repositorio.

La orden de despliegue clonará el repositorio indicado en la petición dentro del directorio de deploys (/root/deploy). A continuación, ejecutará el script de despliegue de la aplicación que hayamos indicado 

	({name: 'pixel-sushi', project: 'pixel-sushi', repo: 'https://marcosrgz...pixel-sushi.git'})
	
Ideal sería que crease un nuevo bloque server en un fichero dentro de /etc/nginx/sites-available, de manera que se autoimportase.
