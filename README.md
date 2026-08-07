1. Despliegue del Emulador Cloud (Floci)
Se inició el servidor local de Floci utilizando Docker Compose para emular el endpoint de AWS en el puerto 4566.

Bash
docker compose up -d
2. Configuración Global de la AWS CLI
Para evitar errores de autenticación (NoCredentials) al interactuar con el entorno local, se configuraron credenciales ficticias mediante el comando interactivo:

Bash
aws configure
Parámetros ingresados:

AWS Access Key ID: test

AWS Secret Access Key: test

Default region name: us-east-1

Default output format: json

3. Creación del primer Bucket en S3 (mb)
Se utilizó la instrucción mb (Make Bucket) apuntando a la URL del endpoint de Floci:

Bash
aws --endpoint-url http://localhost:4566 s3 mb s3://test-bucket
4. Inspección y Subida de Archivos (cp)
Se realizaron pruebas de carga individual de archivos desde la máquina local hacia el contenedor S3 con la instrucción cp (Copy):

Bash
aws --endpoint-url http://localhost:4566 s3 cp ./perro.avif s3://test-bucket/
5. Verificación de Contenido en S3 (ls)
Se listaron los objetos almacenados en la raíz del bucket para confirmar la recepción exitosa del archivo:

Bash
aws --endpoint-url http://localhost:4566 s3 ls s3://test-bucket
6. Descarga de Archivos desde S3 hacia el Host (cp)
Se recuperó un objeto alojado en S3 bajándolo a la máquina local con un nombre diferente (ñau.descargado.jpeg) para validar la integridad del archivo:

Bash
aws --endpoint-url http://localhost:4566 s3 cp s3://test-bucket/gatito.jpeg ./ñau.descargado.jpeg
7. Eliminación de Objetos Específicos (rm)
Se eliminó un objeto individual dentro del bucket mediante la instrucción rm (Remove):

Bash
aws --endpoint-url http://localhost:4566 s3 rm s3://test-bucket/perro.avif
8. Sincronización de Directorios Locales (sync)
Se sincronizó una carpeta local completa (./syncro) con el almacenamiento S3, subiendo de forma masiva los archivos nuevos detectados:

Bash
aws --endpoint-url http://localhost:4566 s3 sync ./syncro s3://test-bucket/
9. Inspección Visual en Floci UI
Se accedió al panel de control web desde el navegador (http://localhost:4500) para validar en una interfaz gráfica (GUI) la presencia de los buckets creados y los archivos subidos.

10. Eliminación Forzada del Bucket y Limpieza (rb --force)
Finalmente, se eliminó el bucket completo junto a todos los objetos almacenados en su interior en un solo comando mediante la bandera --force:

Bash
aws --endpoint-url http://localhost:4566 s3 rb s3://test-bucket --force
