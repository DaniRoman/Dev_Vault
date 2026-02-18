## Cargar un script antes de iniciar  desde compose


[Rec     urso](https://community.caribbean.dev/t/how-can-i-import-a-sql-file-once-i-start-up-my-docker-compose-file/295/2)

`/docker-entrypoint-initdb.d/` Es un directorio dentro del contenedor de MySQL que la imagen oficial de MySQL revisa automáticamente este al iniciar el contenedor por primera vez cargando cualquier archivo `.sql, .sh`

## Comandos

```sql
#Conectarse a MySQL dentro del contenedor Docker
docker exec -it mysql_db mysql -u root -p

# (introduce la contraseña cuando la pida)

#Ver todas las bases de datos
SHOW DATABASES;

# 3️⃣ Usar una base de datos
USE appdb;

# 4️⃣ Ver tablas de la base seleccionada
SHOW TABLES;

# 5️⃣ Ver estructura de una tabla
DESCRIBE nombre_tabla;

# 6️⃣ Ver datos de una tabla
SELECT * FROM nombre_tabla;

# 7️⃣ Ver datos con límite
SELECT * FROM nombre_tabla LIMIT 10;

# 8️⃣ Ver usuarios y método de autenticación
SELECT user, host, plugin FROM mysql.user;

# 9️⃣ Ver en qué base estás ahora
SELECT DATABASE();

# 🔟 Salir de MySQL
EXIT;

```

