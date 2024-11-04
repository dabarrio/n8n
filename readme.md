# Hacer que Docker Compose se ejecute al iniciar el servidor
------------

Para asegurarte de que los contenedores se inicien automáticamente cuando el servidor se reinicie, puedes utilizar systemd para crear un servicio que ejecute docker-compose up al iniciar el servidor.

### 1. Crear un archivo de servicio de systemd:
Crea un archivo llamado `docker-compose-n8n.service` en `/etc/systemd/system/` con el siguiente contenido:
```
[Unit]
Description=Docker Compose N8N Service
Requires=docker.service
After=docker.service

[Service]
Restart=always
WorkingDirectory=/ruta/al/directorio/de/tu/docker-compose  # Cambia esto a la ruta de tu archivo docker-compose.yml
ExecStart=/usr/local/bin/docker-compose up -d
ExecStop=/usr/local/bin/docker-compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```
Asegúrate de cambiar la ruta en WorkingDirectory a la ubicación donde se encuentra tu archivo docker-compose.yml.

### 2. Habilitar y iniciar el servicio:
Luego, habilita el servicio para que se inicie automáticamente al arrancar el servidor:
```
sudo systemctl enable docker-compose-n8n.service
sudo systemctl start docker-compose-n8n.service
```

### 3. Verificar el estado del servicio:
Puedes verificar el estado del servicio con el siguiente comando:
```
sudo systemctl status docker-compose-n8n.service
```

# Pasos para Realizar un Backup Manual de PostgreSQL en Docker
------------

### Realizar Backup usando docker exec
Abre tu terminal y ejecuta el siguiente comando para crear un backup de la base de datos:

```
docker exec -t postgres pg_dump -U n8n_user n8n_database > backup_n8n_database_$(date +%Y-%m-%d_%H-%M-%S).sql
```

Detalles del comando:
- `postgres`: Nombre del contenedor de PostgreSQL.
- `pg_dump -U n8n_user n8n_database`: Comando para hacer el dump de la base de datos n8n_database con el usuario n8n_user.
- `> backup_n8n_database_$(date +%Y-%m-%d_%H-%M-%S).sql`: Redirige la salida del comando pg_dump a un archivo .sql en el host con una marca de tiempo.

Crear el backup directamente en el contenedor y copiarlo al host (OPCIONAL)
Crear el backup dentro del contenedor:
```
docker exec postgres pg_dump -U n8n_user n8n_database -f /var/lib/postgresql/data/backup_n8n_database.sql
```

Copiar el archivo de backup al host:
```
docker cp postgres:/var/lib/postgresql/data/backup_n8n_database.sql ./backup_n8n_database.sql
```

### Realizar Backup usando Docker Compose
Si estás usando Docker Compose, ejecuta el siguiente comando para realizar el backup:
```
docker-compose exec postgres pg_dump -U n8n_user n8n_database > backup_n8n_database_$(date +%Y-%m-%d_%H-%M-%S).sql
```
Este comando crea un archivo .sql con el contenido de la base de datos.

### Verificar el Backup
Una vez completado el backup, verifica el contenido del archivo con el siguiente comando:
```
head -n 20 backup_n8n_database_$(date +%Y-%m-%d_%H-%M-%S).sql
```
Esto mostrará las primeras 20 líneas del archivo para asegurarte de que el backup se realizó correctamente.

### Restaurar el Backup
Para restaurar el backup, utiliza el siguiente comando:
```
cat backup_n8n_database.sql | docker exec -i postgres psql -U n8n_user -d n8n_database
```
Esto leerá el contenido del archivo backup_n8n_database.sql y lo restaurará en la base de datos n8n_database.