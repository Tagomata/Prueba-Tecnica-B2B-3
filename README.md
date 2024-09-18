# Creación de MySQL mirror server and attach config MySQL

### 1. Instalar MySQL en el servidor principal y en el servidor espejo


```shellscript
# En ambos servidores:
sudo apt-get update
sudo apt-get install mysql-server
```

En este paso instala MySQL en ambos servidores. Es el primer paso necesario para configurar la replicación.

### 2. Configurar el servidor MySQL principal


```ini
# Editar el archivo de configuración de MySQL en el servidor principal
# Ubicación típica: /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = nombre_de_tu_base_de_datos
```

Aquí configuramos el servidor principal. El `server-id` debe ser único, habilitamos el registro binario y especificamos qué base de datos replicar.

### 3. Reiniciar el servicio MySQL en el servidor principal


```shellscript
sudo systemctl restart mysql
```

Reiniciamos MySQL para que los cambios de configuración surtan efecto.

### 4. Crear un usuario de replicación en el servidor principal


```sql
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;
```

Creamos un usuario específico para la replicación con los permisos necesarios.

### 5. Obtener la información de posición del registro binario


```sql
SHOW MASTER STATUS;
```

Este comando nos da la información necesaria para configurar el servidor espejo.

### 6. Configurar el servidor MySQL espejo


```ini
# Editar el archivo de configuración de MySQL en el servidor espejo
# Ubicación típica: /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = nombre_de_tu_base_de_datos
```

Configuramos el servidor espejo. Nótese que el `server-id` es diferente al del principal.

### 7. Reiniciar el servicio MySQL en el servidor espejo


```shellscript
sudo systemctl restart mysql
```

Reiniciamos MySQL en el servidor espejo para aplicar la configuración.

### 8. Configurar la replicación en el servidor espejo


```sql
CHANGE MASTER TO
MASTER_HOST='ip_del_servidor_principal',
MASTER_USER='repl_user',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='mysql-bin.XXXXXX',
MASTER_LOG_POS=YYYY;

START SLAVE;
```

Aquí configuramos el servidor espejo para que se conecte al principal y comience la replicación. Reemplaza 'XXXXXX' y 'YYYY' con los valores obtenidos en el paso 5.

### 9. Verificar el estado de la replicación


```sql
SHOW SLAVE STATUS\G
```

Este comando nos muestra si la replicación está funcionando correctamente.

### 10. Configurar la aplicación para usar el servidor espejo para transacciones


```python
# Ejemplo en Python usando mysql-connector

import mysql.connector

config = {
    'user': 'usuario',
    'password': 'contraseña',
    'host': 'ip_del_servidor_espejo',
    'database': 'nombre_de_tu_base_de_datos',
    'raise_on_warnings': True
}

cnx = mysql.connector.connect(**config)
cursor = cnx.cursor()

# Realizar transacciones aquí
```

Este es un ejemplo de cómo configurar una aplicación para usar el servidor espejo para transacciones. Asegúrate de usar el servidor espejo para escrituras y transacciones.

Es importante destacar que esta configuración está optimizada para transacciones y no para consultas de información, como se indicó en la prueba. El servidor espejo se utiliza principalmente para operaciones de escritura y para garantizar la disponibilidad de los datos en caso de que el servidor principal falle.

Para mejorar el rendimiento de las transacciones, considera las siguientes recomendaciones:

1. Utiliza transacciones cortas para reducir el tiempo de bloqueo.
2. Optimiza tus consultas de escritura para que sean lo más eficientes posible.
3. Considera el uso de índices apropiados para mejorar el rendimiento de las escrituras.
4. Monitorea regularmente el rendimiento de la replicación y ajusta la configuración según sea necesario.


Recuerda que esta configuración es una base y puede necesitar ajustes dependiendo de las necesidades específicas y la carga de trabajo de la aplicación.
