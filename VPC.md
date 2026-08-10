cat << 'EOF' > VPC.md
# ☁️ Despliegue de Arquitectura VPC en AWS con LocalStack

Proyecto técnico enfocado en el diseño, provisión, enrutamiento y verificación de una infraestructura de red privada (**Virtual Private Cloud**) en AWS utilizando la **AWS CLI** sobre un entorno de simulación local con **LocalStack**.

---

## 📐 Diagrama de la Arquitectura

![Diagrama AWS VPC](Images/preview%20(8).webp)

### 📌 Especificaciones Técnicas
* **VPC:** CIDR `100.0.0.0/16` (`floci-vpc`) | ID: `vpc-8076c853`
* **Subred Pública:** `100.0.1.0/24` | ID: `subnet-7af0455c` (Zona: `us-east-1a`)
* **Subred Privada:** `100.0.2.0/24` | ID: `subnet-2f0593a0` (Zona: `us-east-1a`)
* **Internet Gateway:** `igw-96e96236` (Atornillado a `floci-vpc`)
* **Tabla de Rutas Pública:** `rtb-5b1cc3b7` (Ruta `0.0.0.0/0 -> IGW`)
* **Security Group:** `SG-floci` (`sg-66e40645534f10caa`) | Inbound Rules: HTTP 80 & HTTPS 443

---

## 🛠️ Paso a Paso por Terminal (AWS CLI)

### 1. Creación de la VPC
Se define el espacio de direccionamiento IP principal para la red virtual aislada.

aws --endpoint-url=http://localhost:4566 ec2 create-vpc \
    --cidr-block 100.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=floci-vpc}]'

![Creación de VPC](Images/preview%20(4).webp)

---

### 2. Segmentación de Subredes (Pública y Privada)
Se divide la VPC en dos subredes `/24` independientes dentro de la misma Zona de Disponibilidad (`us-east-1a`).

# Subred Pública
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Publica}]'

# Subred Privada
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.2.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Privada}]'

![Creación de Subredes](Images/preview%20(5).webp)

---

### 3. Conexión Externa (Internet Gateway)
Se aprovisiona el componente edge para dar salida y entrada de tráfico público a la red virtual.

# Crear Internet Gateway
aws --endpoint-url=http://localhost:4566 ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Internet-Gateway}]'

# Vincular (Attach) el IGW a la VPC
aws --endpoint-url=http://localhost:4566 ec2 attach-internet-gateway \
    --vpc-id vpc-8076c853 \
    --internet-gateway-id igw-96e96236

![Configuración del IGW](Images/preview%20(2).webp)

---

### 4. Configuración de Tabla de Rutas y Enrutamiento
Se crea la tabla de rutas, se inyecta la ruta por defecto (`0.0.0.0/0` apuntando al IGW) y se asocia **exclusivamente** a la Subred Pública.

# Crear Tabla de Rutas
aws --endpoint-url=http://localhost:4566 ec2 create-route-table \
    --vpc-id vpc-8076c853 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Tabla-Rutas-Publica}]'

# Inyectar ruta por defecto hacia el IGW
aws --endpoint-url=http://localhost:4566 ec2 create-route \
    --route-table-id rtb-5b1cc3b7 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-96e96236

# Asociar la tabla a la Subred Pública
aws --endpoint-url=http://localhost:4566 ec2 associate-route-table \
    --subnet-id subnet-7af0455c \
    --route-table-id rtb-5b1cc3b7

![Tabla de Rutas](Images/preview%20(1).webp)

---

### 5. Configuración de Firewall Perimetral (Security Group)
Se define un Security Group *stateful* para permitir tráfico web entrante (*Inbound Rules*) en los puertos 80 y 443.

# Crear el Security Group
aws --endpoint-url=http://localhost:4566 ec2 create-security-group \
    --group-name SG-floci \
    --description "firewall para web" \
    --vpc-id vpc-8076c853

# Habilitar puerto 80 (HTTP)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

![Creación Security Group](Images/preview.webp)

![Reglas Ingress Security Group](Images/preview%20(6).webp)

---

## 🔍 Auditoría e Inspección de Recursos

Ejecución de consultas `describe-*` para auditar la provisión del estado JSON de los recursos en LocalStack:

aws --endpoint-url=http://localhost:4566 ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-8076c853"

![Auditoría de Subredes](Images/preview%20(3).webp)

![Terminal Interactivo CLI](Images/18e2a1e2-80cd-4065-8f01-d7dc4a760f7c.jpeg)

---

## 🚀 Prueba de Concepto (Validación Servidor Web Nginx)

Se valida la alcanzabilidad de la red simulando la ejecución de una carga de trabajo en la subred pública expuesta en el puerto 80:

docker run -d -p 80:80 --name servidor-web-prueba nginx

![Respuesta Servidor Nginx](Images/preview%20(7).webp)

---

## 📂 Organización de Archivos del Proyecto

Estructura de la carpeta local y organización de recursos del laboratorio:

![Estructura del Proyecto](Images/ead9c602-2fb8-459f-a83e-47d3370767fe.jpeg)
EOF