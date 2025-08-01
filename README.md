# Práctica final - Contenedores: más que VMs - Docker
**XII Edición Bootcamp DevOps & Cloud Computing Full Stack**

**Evaristo García Zambrana | 3 de agosto de 2025**

---

## Diagrama de arquitectura
Diagrama que contempla el resultado final de la arquitectura de microservicios desplegada por el *docker-compose.yml* de este repositorio.
![Diagrama de arquitectura de la aplicación desplegada con Docker Compose](https://github.com/KeepCodingCloudDevops12/EvaristoGZ_02-Docker/blob/main/02_Diagrama%20Docker%20Compose%20-%20Evaristo%20GZ.drawio.jpg)

## Descripción de la aplicación

*kc-visit-counter* es una aplicación web desplegada en arquitectura de microservicios con Docker Compose. Desarrollada en Python y PostgreSQL, permite registrar cada visita HTTP y guarda fecha/hora en una base de datos PostgreSQL. Además, devuelve las 10 últimas visitas con fecha y hora.

- **Frontend**: Nginx como proxy inverso.
- **Backend**: Python 3.13 + Flask proporcionando directamente el HTML. Aloja los logs de Python en un volumen persistente de Docker.
- **Base de datos**: PostgreSQL 15 alojando los datos en un volumen persistente de Docker. Aloja los logs de PostgreSQL en un volumen persistente de Docker.
- **Logging**: Filebeat leyendo los logs de los volúmenes, pasándolo a Elasticsearch y visualizándolos a través de Kibana mediante un navegador web.
- **Orquestación**: Docker Compose, sirviendo como orquestador de servicios Docker.
- **Administrador de contenedores**: Portainer para la visualización y administración del stack a través de navegador web.

## Requisitos

- Docker Engine (o Docker Desktop).
- Docker Compose.
- Git.

Ejecutado en Windows 11 con Docker Desktop 4.43.2, Docker Engine 28.3.2, Docker Compose v2.38.2-desktop.1 mediante WSL 2.

## Estructura del repositorio
```
├── app                               <--- Directorio para el contenido del servicio Python.
│   ├── Dockerfile                    <--- Dockerfile para el contenedor de Python.
│   ├── Dockerfile_sin_multistage     <--- Dockerfile sin la tarea de multistage.
│   ├── app.py                        <--- Código Python + Flask de la app.
│   ├── requirements.txt              <--- Archivo requirements.
│   └── static                        <--- Directorio para los estáticos de la app.
│       └── style.css                 <--- Hoja de estilo aplicada al HTML de la app.
├── docker-compose.yml                <--- docker-compose.yml del entorno de producción.
├── docker-compose_test.yml           <--- docker-compose_test.yml del entorno de test.
├── filebeat                          <--- Directorio para el contenido del servicio Filebeat.
│   ├── filebeat.yml                  <--- Fichero de configuración de Filebeat.
├── nginx                             <--- Directorio para el contenido del servicio Nginx.
│   └── nginx.conf                    <--- Fichero de configuración de Nginx.
└── postgresql                        <--- Directorio para el contenido del servicio db.
    ├── Dockerfile                    <--- Dockerfile para el contenedor de PostgreSQL.
    ├── init-scripts                  <--- Directorio para los entrypoints del contenedor.
    │   ├── 01-schema.sql             <--- Fichero de creación de tablas de la base de datos.
    │   └── 02-users.sh               <--- Fichero de creación de usuarios de la base de datos.
    └── postgresql.conf               <--- Fichero de configuración de PostgreSQL.
```

---

## 🚀 Cómo desplegar kc-visit-counter

### 1. Clona el repositorio de GitHub
```
git clone https://github.com/KeepCodingCloudDevops12/EvaristoGZ_02-Docker
```

### 2. Ubícate en el directorio Git
```
cd EvaristoGZ_02-Docker
```

Si usas WSL 2, exporta la variable PWD con `export PWD=$(realpath .)`

### 3. Crea el fichero .env
con el siguiente contenido:
```
# Aprovisionamiento PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=SUPER-PA$$WORD
POSTGRES_DB=kc-visit-counter

# Cadena conexión BBDD
DB_NAME=kc-visit-counter
DB_USER=kc-user
DB_PASSWORD=kc-PA$$WORD
DB_HOST=db
DB_PORT=5432

# Portainer
PORTAINER_ADMIN_USERNAME=admin
PORTAINER_ADMIN_PASSWORD=SUPER-PA$$WORD

# Otros
TZ=Europe/Madrid
```

### 4. Levanta el Docker Compose
Este paso, además, construirá las imágenes de Docker y las descargará en caso de ser necesario.
`docker compose up --build -d`

Puedes usar `docker compose --build` para que se muestre la salida de los logs de contenedores en pantalla.

### 5. Verifica el estado del Compose
Ejecutamos `docker compose ls` para conocer el estado del compose.

En la columna status deberá poner *running(7)*.

### 6. Accede al servicio web
A través de un navegador web, accede a http://localhost

### 7. Comprueba el acceso al resto de servicios
- **Portainer**
  
  Será el primero, puesto que dispone de un máximo de 5 minutos para establecer la contraseña de admin.

  Accedemos a través de http://localhost:9000

  El resultado esperado es un número, que representa el total de las visitas o peticiones a esa dirección URL y una tabla con las últimas 10 visitas.

- **Kibana**
  
  Accedemos a través de http://localhost:5601/.

  La primera vez que iniciamos Kibana tenemos que crear el Data View:

  1. Accedemos en el menú lateral a  Analytics>Discover.
     - Name: filebeat-*
     - Index pattern: filebeat-*
  3. A la derecha aparecerá los índices que coinciden con el patrón.
  4. Pulsamos sobre el botón “Save data view to Kibana”.
 

### 8. Para el Docker Compose
Tras realizar las comprobaciones de los servicios y comprobar el funcionamiento de la infraestructura, el penúltimo paso es parar el Docker Compose.

Ubicados en el directorio del repositorio ejecutamos:
```
docker compose down
```
Esto parará los servicios y los contenedores, pero no los eliminará. En caso de querer volver a ejecutarlos, basta con ejecutar `docker compose up -d`

### 9. Elimina los servicios creados con Docker Compose
Por último, si deseas no tener rastro de este proyecto en tu sistema, utiliza el parámetro *-d*:
```
docker compose down -d
```

Esto hará que, además de parar los servicios y contenedores, elimine las redes y volúmenes creados por el fichero *docker-compose.yml*.

Recuerda que, la eliminación de los volúmenes no es una eliminación del recurso virtual, si no que también conlleva también la eliminación de los datos alojados en él. Estos no se alojan en ningún otro lugar y no serán recuperables.

---

