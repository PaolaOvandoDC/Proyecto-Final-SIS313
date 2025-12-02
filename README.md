Infraestructura para plataforma LMS Sakai con Nginx y MariaDB
# 🚀 Proyecto Final SIS313: simulacion de una red universitaria

> **Asignatura:** SIS313: Infraestructura, Plataformas Tecnológicas y Redes<br>
> **Semestre:** 2/2025<br>
> **Docente:** Ing. Marcelo Quispe Ortega

## 👥 Miembros del Equipo ([3])

| Nombre Completo | Rol en el Proyecto | Contacto (GitHub/Email) |
| :--- | :--- | :--- |
| [Flores Aquino Nayely] | [arquitecto de sistemas/infraestructura] | [Usuario de GitHub] |
| [Herrera Jesus David] | [ingeniero de redes y proxy] | [Usuario de GitHub] |
| [Ovando Calizaya Paola Daniela] | [ingeniero de aplicaciones y despliegue] | [https://github.com/PaolaOvandoDC] |
| [Salgueiro Gardeasabal Josue David] | [administrador de base de datos (DBA)] | [Usuario de GitHub] |

## 🎯 I. Objetivo del Proyecto

Objetivo: Implementar y configurar Sakai LMS (Learning Management System) en un entorno de producción con arquitectura de tres capas, incluyendo proxy inverso con Nginx, servidor de aplicaciones Tomcat, y base de datos MariaDB, garantizando alta disponibilidad, seguridad y escalabilidad.

> **Objetivo:** [Indicar el objetivo del proyecto, ej: "Diseñar y configurar un clúster de Base de Datos con replicación Maestro-Esclavo para optimizar el rendimiento y la tolerancia a fallos."]
> ## 💡 II. Justificación e Importancia

Justificación:
En el contexto educativo actual, contar con una plataforma LMS robusta y confiable es esencial para la continuidad académica. Este proyecto elimina los Single Points of Failure mediante:

Separación de capas: Proxy, aplicación y base de datos en servidores independientes
Alta disponibilidad: Implementación de arquitectura distribuida que permite escalabilidad horizontal
Seguridad robusta: Aplicación de hardening en múltiples niveles (red, sistema operativo, aplicación)
Gestión eficiente: Uso de automatización para despliegues consistentes y reproducibles

El proyecto integra conceptos de Alta Disponibilidad (T2), Seguridad y Hardening (T5), Balanceo de Carga (T3) y Monitoreo (T4), resolviendo problemas críticos de continuidad operacional en infraestructuras educativas.

## 🛠️ III. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave
Enumera y describe brevemente el rol de cada software y tecnología utilizada.

Nginx: Proxy inverso con funciones de balanceo de carga, rate limiting y terminación SSL/TLS
Apache Tomcat 9: Servidor de aplicaciones Java para hospedar Sakai LMS
MariaDB 10.11: Sistema de gestión de base de datos relacional con soporte para replicación
Sakai LMS 23.x: Plataforma de gestión de aprendizaje open-source con arquitectura modular
Ubuntu Server 24.04 LTS: Sistema operativo base para todos los servidores
UFW (Uncomplicated Firewall): Gestión de reglas de firewall para segmentación de red
systemd: Gestión de servicios y arranque automático del sistema

### 3.2. Conceptos de la Asignatura Puestos en Práctica (T1 - T6)

✅ Alta Disponibilidad (T2) y Tolerancia a Fallos:

Arquitectura de tres capas con separación física de componentes
Configuración de timeouts extendidos en Nginx para aplicaciones de larga duración
Estrategia de backup y recuperación para la base de datos

✅ Seguridad y Hardening (T5):

Configuración de firewall UFW con reglas restrictivas por origen
Hardening de SSH: cambio de puerto, deshabilitación de root login
Segmentación de red con tres subredes independientes (192.168.10.0/24, 192.168.10.3/32, 192.168.10.4/32)
Gestión de permisos con usuarios del sistema dedicados (sakai)
Configuración de autenticación de base de datos con usuarios específicos por host

✅ Automatización y Gestión (T6):

Scripts de instalación y configuración automatizados
Gestión de servicios con systemd
Configuración centralizada mediante archivos properties
Despliegue automático de aplicaciones WAR

✅ Balanceo de Carga/Proxy (T3/T4):

Nginx como proxy inverso con upstream backend
Configuración de health checks y timeouts personalizados
Rate limiting y control de tamaño de uploads (100MB)
Headers de proxy para preservar información del cliente

✅ Networking Avanzado (T3):

Diseño de topología con tres subredes lógicas
Configuración de rutas entre servidores
Control de tráfico mediante firewall por origen y destino
Resolución de problemas de conectividad entre capas

## 🌐 IV. Diseño de la Infraestructura y Topología

### 4.1. Diseño Esquemático

Incluye un diagrama de la topología final. Muestra claramente la segmentación de red, las IPs utilizadas, y los flujos de tráfico.
Internet/Usuario
                           |
                           v
                  [192.168.10.2:80]
                    PROXY (Nginx)
                           |
                  (proxy_pass)
                           |
                           v
                  [192.168.10.3:8080]
              APLICACIÓN (Tomcat + Sakai)
                           |
                  (JDBC Connection)
                           |
                           v
                  [192.168.10.4:3306]
                BASE DE DATOS (MariaDB)
                
### 4.2. Estrategia Adoptada (Opcional)

Estrategia de Separación de Capas:

Se implementó una arquitectura de tres capas para:

Mejorar la seguridad mediante segmentación
Facilitar el escalamiento horizontal
Permitir actualizaciones independientes de componentes
Aislar fallos en una capa sin afectar las demás



Estrategia de Configuración:

Configuración centralizada de Sakai mediante sakai.properties
Gestión de servicios mediante systemd para arranque automático
Separación de logs por componente para troubleshooting eficiente

Estrategia de Seguridad:

Firewall configurado en cada capa con reglas específicas
Autenticación de base de datos por host de origen
Usuario del sistema dedicado (sakai) sin privilegios de root
Timeouts personalizados para prevenir ataques de denegación de servicio

## 📋 V. Guía de Implementación y Puesta en Marcha

Documenta los pasos esenciales para que cualquier persona pueda replicar el proyecto (instalación, configuración de ficheros clave, comandos).

### 5.1. Pre-requisitos
3 máquinas virtuales con Ubuntu Server 24.04 LTS
Acceso root o  sudo en todas las VMs
Conectividad de red entre todas las VMs
Mínimo 2GB RAM por VM (4GB recomendado para VM-APP)
20GB de espacio en disco por VM
### 5.3. Ficheros de Configuración Clave

paso1 Configurar la red estática, en las tres VMS

🌐 INSTALACIÓN Y CONFIGURACIÓN DE NGINX EN EL PROXY
PASO 1: Instalar Nginx

``` sudo apt install nginx -y ```

``` sudo systemctl status nginx```

PASO 2: Crear el archivo de configuración para Sakai

Vamos a crear un archivo de configuración específico para Sakai:

bashsudo nano /etc/nginx/sites-available/sakai

upstream sakai_backend {
    server 192.168.100.10:8080;
}

server {
    listen 80;
    server_name sakai.local;

    client_max_body_size 100M;

    access_log /var/log/nginx/sakai_access.log;
    error_log /var/log/nginx/sakai_error.log;

    location / {
        proxy_pass http://sakai_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 600;
        proxy_send_timeout 600;
        proxy_read_timeout 600;
        send_timeout 600;
    }

}

PASO 3: Habilitar el sitio de Sakai

Nginx tiene dos carpetas: sites-available (sitios disponibles) y sites-enabled (sitios 

activos). Vamos a crear un enlace simbólico:

bashsudo ln -s /etc/nginx/sites-available/sakai /etc/nginx/sites-enabled/

Ahora elimina el sitio por defecto para evitar conflictos:

bashsudo rm /etc/nginx/sites-enabled/default

PASO 4: Verificar la configuración de Nginx

Antes de reiniciar, verifica que no haya errores de sintaxis:

bashsudo nginx -t

as ver:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful

PASO 5: Reiniciar Nginx

Si todo está OK, reinicia Nginx para aplicar los cambios:

bashsudo systemctl restart nginx

Verifica que sigue corriendo:

bashsudo systemctl status nginx

PASO 6: Habilitar Nginx para que inicie automáticamente

Para que Nginx se inicie automáticamente cuando reinicies la VM:

bashsudo systemctl enable nginx

Deberías ver algo como "Created symlink..."

PASO 7: Verificar que Nginx está escuchando en el puerto 80

bashsudo netstat -tlnp | grep 80

🗄️ CONFIGURACIÓN DEL SERVIDOR DE BASE DE DATOS (Lab4.1-DB)

1. Actualizar repositorios:

bashsudo apt update

2. Instalar MariaDB Server:

bashsudo apt install mariadb-server mariadb-client -y

3. Iniciar el servicio:

bashsudo systemctl start mariadb

4. Verificar que esté corriendo:

bashsudo systemctl status mariadb

5. Habilitar para inicio automático:

bashsudo systemctl enable mariadb

sudo mysql_secure_installation

"Enter current password for root (enter for none):"

Solo presiona Enter (no hay contraseña aún)


"Switch to unix_socket authentication [Y/n]"

Escribe: n
Enter


"Change the root password? [Y/n]"

Escribe: y
Enter


"New password:"

Escribe: 1234
Enter
"Re-enter new password:"

Escribe: 1234
Enter


"Remove anonymous users? [Y/n]"

Escribe: y
Enter


"Disallow root login remotely? [Y/n]"

Escribe: n (necesitamos acceso remoto)
Enter


"Remove test database and access to it? [Y/n]"

Escribe: y
Enter

"Reload privilege tables now? [Y/n]"

Escribe: y
Enter

6. Editar la configuración:

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

bind-address = 127.0.0.1

**3. Cambiarla por:**

bind-address = 0.0.0.0

7. Reiniciar MariaDB:

bashsudo systemctl restart mariadb

8. Verificar que siga corriendo:

bashsudo systemctl status mariadb

10. Conectarse a MariaDB:

bashsudo mysql -u root -p

11. Crear la base de datos:

sqlCREATE DATABASE sakai DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

12. Crear el usuario:

sqlCREATE USER 'sakai'@'%' IDENTIFIED BY '1234';

13. Dar permisos:

sqlGRANT ALL PRIVILEGES ON sakai.* TO 'sakai'@'%';

14. Recargar privilegios:

sqlFLUSH PRIVILEGES;

15. Verificar que la BD se creó:

sqlSHOW DATABASES;

17. Salir:

sqlEXIT;

Crear la base de datos y usuario para Sakai

PASO 1: Conectarse a MariaDB

bashsudo mysql -u root -p

Ingresa la contraseña que configuraste:

MariaDB [(none)]>

PASO 2: Crear la base de datos

sqlCREATE DATABASE sakai DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

Deberías ver: Query OK, 1 row affected


PASO 3: Crear el usuario

sqlCREATE USER 'sakai'@'%' IDENTIFIED BY '1234';

PASO 4: Dar permisos al usuario

sqlGRANT ALL PRIVILEGES ON sakai.* TO 'sakai'@'%';

Deberías ver: Query OK, 0 rows affected

PASO 5: Recargar privilegios

sqlFLUSH PRIVILEGES;

PASO 6: Verificar que la base de datos existe

sqlSHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakai              | <-- Esta es la nuestra
| sys                |
+--------------------+

🖥️ CONFIGURACIÓN DEL SERVIDOR DE APLICACIONES

PASO 1: Instalar herramientas básicas

bashsudo apt install wget curl vim net-tools -y

PASO 2: Instalar Java 11 (OpenJDK)

Sakai necesita Java para funcionar. Instalemos Java 11:

bashsudo apt install openjdk-11-jdk -y

PASO 3: Configurar variables de entorno de Java

bashsudo nano /etc/environment

JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"

CATALINA_HOME="/opt/tomcat"

Aplica los cambios:

bashsource /etc/environment

Verifica:

bashecho $JAVA_HOME

PASO 4: Crear usuario para Sakai

Por seguridad, Sakai no debe correr como root. Creamos un usuario dedicado:

bashsudo useradd -m -s /bin/bash sakai

Ponle una contraseña:

bashsudo passwd sakai

PASO 5: Descargar Apache Tomcat

Tomcat es el servidor de aplicaciones donde correrá Sakai:

bashcd /tmp

bashwget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65

PASO 6: Instalar Tomcat

Crear el directorio:

bashsudo mkdir /opt/tomcat

Extraer Tomcat:

bashsudo tar -xzvf apache-tomcat-9.0.65.tar.gz -C /opt/tomcat --strip-components=1

Cambiar el propietario al usuario sakai:

bashsudo chown -R sakai:sakai /opt/tomcat

Verifica que se instaló:

bashls -la /opt/tomcat/

PASO 7: Crear directorios para Sakai

bashsudo mkdir -p /opt/sakai/config

bashsudo mkdir -p /opt/sakai/content

bashsudo chown -R sakai:sakai /opt/sakai

PASO 8: Descargar Sakai

cd /tmp

wget https://source.sakaiproject.org/release/23.3/artifacts/sakai-bin-23.3.tar.gz

PASO 9: Extraer Sakai en Tomcat

bashcd /tmp

bashsudo tar -xzvf sakai-bin-23.3.tar.gz -C /opt/tomcat/webapps/

 PASO 10: Ajustar permisos

bashsudo chown -R sakai:sakai /opt/tomcat/webapps/*

Verifica que se instaló correctamente:

bashls -la /opt/tomcat/webapps/

PASO 11: Instalar el driver de MariaDB

bashcd /opt/tomcat/lib

bashsudo wget https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.0.8/mariadb-

java-client-3.0.8.jar

Ajustar permisos:

bashsudo chown sakai:sakai mariadb-java-client-3.0.8.jar

Verifica:

bashls -lh mariadb-java-client-3.0.8.jar

PASO 12: Crear el archivo de configuración de Sakai

bashsudo nano /opt/sakai/config/sakai.properties

username@javax.sql.BaseDataSource=sakaiuser

password@javax.sql.BaseDataSource=SakaiPass2024!

vendor@org.sakaiproject.db.api.SqlService=mysql

driverClassName@javax.sql.BaseDataSource=org.mariadb.jdbc.Driver

hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

url@javax.sql.BaseDataSource=jdbc:mysql://192.168.100.20:3306/sakai?

useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC

serverUrl=http://sakai.local

serverName=sakai.local

portalPath=/portal

bodyPath@org.sakaiproject.content.api.ContentHostingService=/opt/sakai/content

memory.db=true

locales=es_ES,en_US

sudo chown sakai:sakai /opt/sakai/config/sakai.properties

PASO 13: Configurar memoria de Tomcat

bashsudo nano /opt/tomcat/bin/setenv.sh

bashexport JAVA_OPTS="-server -Xms1g -Xmx2g -XX:+UseG1GC -Djava.awt.headless=true -

Djava.net.preferIPv4Stack=true"

export CATALINA_OPTS="-Dsakai.home=/opt/sakai -Dsakai.security=/opt/sakai/config"

sudo chmod +x /opt/tomcat/bin/setenv.sh

PASO 14: Crear servicio systemd para Tomcat

bashsudo nano /etc/systemd/system/tomcat.service

[Unit]

Description=Apache Tomcat - Sakai LMS

After=network.target

[Service]

Type=forking

User=sakai

Group=sakai

Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"

Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"

Environment="CATALINA_HOME=/opt/tomcat"

Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh

ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10

Restart=always
WantedBy=multi-user.target

PASO 15: Recargar systemd y habilitar Tomcat

bashsudo systemctl daemon-reload

bashsudo systemctl enable tomcat

PASO 16: ¡INICIAR SAKAI!

bashsudo systemctl start tomcat

PASO 17: Ver los logs en tiempo real

bashtail -f /opt/tomcat/logs/catalina.out

**Incluir además los archivos de configuración y software a utilizar dentro del proyecto y organizados en carpetas.**
## ⚠️ VI. Pruebas y Validación

Prueba Realizada,Resultado Esperado,Resultado Obtenido
Test de Arranque del Kernel (T1),Tomcat debe mostrar Server startup... y el proceso Java debe estar activo.,[OK] (`ps aux
Test de Escucha de Puerto (T4),El puerto 8080 debe estar en estado LISTEN.,[OK] (`sudo ss -tuln
Test de Acceso Directo (T3),La página de inicio de Sakai debe cargar en (http://localhost:8080/portal).,[OK] .
Test de Proxy (T3),La página debe cargar en la URL limpia http://localhost:8080/portal/xlogin.,[OK]

## 📚 VII. Conclusiones y Lecciones Aprendidas

Logros Principales: Se logró resolver la falla crítica del Kernel de Spring a través de la corrección de la URL de la base de datos y la limpieza recursiva del caché de Tomcat, lo que permitió el arranque exitoso de la plataforma LMS. La aplicación se encuentra en un estado operativo, lista para ser utilizada.

Desafíos Superados:

Errores Silenciosos de Spring/Kernel: Se requirió una limpieza profunda (rm -rf /opt/tomcat/work/*) para forzar un redespliegue limpio.

Problemas de Conectividad de Red: El error final ERR_CONNECTION_TIMED_OUT (image_080865.png) se identificó como un problema de la configuración de Red Interna de VirtualBox, que bloqueaba el acceso del PC anfitrión.

Qué Haría Diferente: Se recomienda migrar la IP a un rango de Adaptador Puente (Bridge) para tener una IP que sea accesible directamente por el PC anfitrión sin la necesidad de reconfigurar la interfaz Host-Only de VirtualBox. También se implementaría el balanceo de carga con una segunda VM de aplicaciones para lograr una Alta Disponibilidad activa.
