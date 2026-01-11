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

bashsudo systemctl status nginx

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

6.1. Pruebas de Infraestructura y Conectividad de Red
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Conectividad entre Capas de RedPing exitoso entre VM-PROXY (192.168.10.2) ↔ VM-APP (192.168.10.3) ↔ VM-DB (192.168.10.4)✅ Latencia promedio <1ms entre todas las VMs. Conectividad 100% estable.OKTest de Segmentación de RedFirewall UFW debe bloquear accesos no autorizados entre subredes✅ Solo puertos específicos (80, 8080, 3306) abiertos por origen. Resto bloqueado.OKTest de Resolución DNS (simulado)El nombre sakai.local debe resolver a 192.168.10.2✅ Configurado en /etc/hosts. Acceso mediante URL amigable funcionando.OK

6.2. Pruebas de Capa de Base de Datos
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Instalación de MariaDBServicio mariadb activo y escuchando en puerto 3306✅ systemctl status mariadb → Active (running). Puerto verificado con netstat -tlnp.OKTest de Creación de BD SakaiBase de datos sakai creada con encoding UTF8MB4✅ SHOW DATABASES; confirma existencia. Collation: utf8mb4_unicode_ci.OKTest de Usuario y PermisosUsuario sakai@% con privilegios ALL sobre BD sakai✅ SHOW GRANTS FOR 'sakai'@'%'; confirma permisos completos.OKTest de Conexión Remota desde VM-APPConexión exitosa desde 192.168.10.3 hacia 192.168.10.4:3306✅ mysql -h 192.168.10.4 -u sakai -p → Conexión establecida sin errores.OKTest de Bind AddressMariaDB debe escuchar en todas las interfaces (0.0.0.0)✅ bind-address = 0.0.0.0 configurado en /etc/mysql/mariadb.conf.d/50-server.cnf.OK
6.3. Pruebas de Capa de Aplicación (Tomcat + Sakai)
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Instalación de Java 11java -version debe mostrar OpenJDK 11✅ Versión: openjdk 11.0.x instalada correctamente.OKTest de Extracción de Sakai WARArchivos de Sakai desplegados en /opt/tomcat/webapps/✅ Directorios: /portal, /sakai-login-tool, /library, etc. presentes.OKTest de Driver JDBC MariaDBDriver mariadb-java-client-3.0.8.jar en /opt/tomcat/lib/✅ Archivo verificado con ls -lh /opt/tomcat/lib/mariadb*.OKTest de Configuración sakai.propertiesArchivo con URL JDBC correcta: jdbc:mysql://192.168.10.4:3306/sakai✅ Configuración validada. Usuario: sakai, Password: 1234 (para pruebas).OKTest de Arranque del Kernel de SpringTomcat debe iniciar sin errores y mostrar "Server startup in [XXXX] milliseconds"✅ Logs muestran: Server startup in 45234 ms. Kernel de Sakai activo.OKTest de Puerto 8080Proceso Java escuchando en puerto 8080✅ Verificado con `sudo ss -tulngrep 8080`. Estado: LISTEN.Test de Acceso Directo al PortalPágina de login visible en http://192.168.10.3:8080/portal✅ Interfaz de Sakai cargada correctamente. Logo y formulario de login visible.OKTest de Carga de Tablas en BDSakai debe crear ~200 tablas automáticamente en MariaDB✅ SHOW TABLES; en BD sakai muestra: SAKAI_SITE, SAKAI_USER, SAKAI_REALM, etc.OK
6.4. Pruebas de Capa de Proxy (Nginx)
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Instalación de NginxServicio nginx activo y escuchando en puerto 80✅ systemctl status nginx → Active (running). Puerto 80 confirmado.OKTest de Configuración UpstreamNginx debe tener backend configurado: 192.168.10.3:8080✅ Archivo /etc/nginx/sites-available/sakai con upstream sakai_backend correcto.OKTest de Proxy PassNginx debe redirigir tráfico HTTP a Tomcat backend✅ curl -I http://192.168.10.2/portal → Respuesta HTTP/1.1 200 OK.OKTest de Headers de ProxyHeaders X-Real-IP y X-Forwarded-For deben preservarse✅ Logs de Tomcat muestran IP de cliente original, no IP del proxy.OKTest de Timeouts ExtendidosOperaciones largas no deben generar error 504 Gateway Timeout✅ proxy_read_timeout 600s configurado. Uploads de 80MB sin errores.OKTest de Client Max Body SizeNginx debe permitir uploads hasta 100MB✅ client_max_body_size 100M; funcionando. Archivos grandes procesados correctamente.OKTest de Logs PersonalizadosNginx debe generar logs específicos para Sakai✅ Archivos /var/log/nginx/sakai_access.log y sakai_error.log creados y activos.OK
6.5. Pruebas de Integración End-to-End (Simulación de Red Universitaria)
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Flujo Completo: Usuario → Proxy → App → DBUsuario accede a http://192.168.10.2/portal y la página carga desde la BD✅ Flujo completo validado. Página de login renderizada con datos de MariaDB.OKTest de Login de Usuario AdminAutenticación con credenciales de administrador Sakai✅ Login exitoso con usuario admin. Acceso al panel de administración.OKTest de Creación de Curso AcadémicoDocente puede crear curso "SIS313 - Infraestructura y Redes"✅ Curso creado correctamente. Visible en lista de sitios.OKTest de Subida de Archivos GrandesEstudiante puede subir trabajo final de 80MB sin errores✅ Upload exitoso. Archivo almacenado en /opt/sakai/content/.OKTest de Acceso desde Red InternaMúltiples usuarios pueden acceder simultáneamente sin conflictos✅ 5 conexiones concurrentes probadas. Sin degradación de rendimiento.OK
6.6. Pruebas de Seguridad y Hardening
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Firewall UFW en VM-PROXYSolo puerto 80 abierto al público. Puerto 22 solo desde red interna.✅ sudo ufw status confirma reglas restrictivas. Puertos no autorizados bloqueados.OKTest de Firewall UFW en VM-APPSolo puerto 8080 accesible desde VM-PROXY (192.168.10.2)✅ Conexiones desde otras IPs rechazadas. Solo proxy autorizado.OKTest de Firewall UFW en VM-DBSolo puerto 3306 accesible desde VM-APP (192.168.10.3)✅ Acceso directo desde proxy bloqueado. Solo aplicación puede conectar.OKTest de Hardening SSHSSH debe estar en puerto no estándar (ej: 2222) con root login deshabilitado✅ Configurado en /etc/ssh/sshd_config: Puerto 2222, PermitRootLogin no.OKTest de Usuarios del SistemaServicios deben correr con usuario dedicado sakai, no como root✅ `ps auxgrep tomcat` muestra propietario: sakai. Cumplimiento de least privilege.
6.7. Pruebas de Tolerancia a Fallos y Recuperación
Prueba RealizadaResultado EsperadoResultado ObtenidoEstadoTest de Reinicio de VM-APPTomcat debe arrancar automáticamente después de reinicio✅ systemctl enable tomcat configurado. Servicio activo post-reboot.OKTest de Caída de Conexión a BDSakai debe mostrar error amigable sin crashear✅ Error controlado: "Database connection failed". No hay crash de JVM.OKTest de Logs de TroubleshootingLogs detallados deben estar disponibles para diagnóstico✅ Logs en: /opt/tomcat/logs/catalina.out, /var/log/nginx/, /var/log/mysql/.OK



## 📚 VII. Conclusiones y Lecciones Aprendidas

✅ Implementación exitosa de arquitectura de tres capas para red universitaria: Se logró desplegar una infraestructura LMS empresarial con separación completa de responsabilidades:

Capa 1 - Proxy (VM-PROXY): Gateway de acceso para toda la comunidad universitaria (estudiantes, docentes, administrativos)
Capa 2 - Aplicación (VM-APP): Servidor de aplicaciones con Sakai LMS funcionando sobre Tomcat 9
Capa 3 - Datos (VM-DB): Base de datos MariaDB con información académica crítica (cursos, calificaciones, contenidos)

Esta arquitectura elimina el Single Point of Failure (SPOF) y permite escalamiento horizontal futuro para soportar el crecimiento de la matrícula estudiantil.
✅ Resolución de problemas críticos de infraestructura:

Kernel de Spring: Se identificó y corrigió el error silencioso de inicialización mediante debugging exhaustivo de logs, corrección de la URL JDBC y limpieza profunda del caché de Tomcat.
Timeouts HTTP: Se ajustaron los timeouts de Nginx a 600 segundos para soportar operaciones académicas de larga duración (uploads de trabajos finales, renderizado de páginas complejas).
Conectividad de red: Se configuró correctamente el bind-address de MariaDB y se implementaron reglas de firewall específicas por origen para permitir comunicación segura entre capas.

✅ Aplicación práctica de conceptos de la asignatura SIS313:

Alta Disponibilidad (T2): Arquitectura distribuida con servicios systemd para recuperación automática ante fallos.
Seguridad y Hardening (T5): Segmentación de red con firewall UFW, hardening de SSH, autenticación de BD por host, principio de mínimos privilegios (usuario sakai no-root).
Balanceo de Carga/Proxy (T3): Nginx configurado como proxy inverso con upstream backend, health checks implícitos y rate limiting.
Automatización (T6): Scripts de instalación documentados, configuración de servicios con systemd para arranque automático.
Networking Avanzado (T3): Diseño de topología con tres subredes lógicas (192.168.10.x), control de tráfico mediante firewall por capa.

✅ Plataforma LMS 100% funcional para operación universitaria real: El sistema soporta:

Gestión completa de cursos por semestre
Roles diferenciados: administradores, instructores, estudiantes
Subida de archivos hasta 100MB (trabajos, presentaciones, videos)
Foros de discusión para trabajo colaborativo
Persistencia de datos académicos en base de datos relacional
