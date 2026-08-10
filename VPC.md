Markdown
# ☁️ Despliegue de Arquitectura VPC en AWS con LocalStack

Proyecto técnico enfocado en el diseño, provisión y verificación de una infraestructura de red privada (**Virtual Private Cloud**) en AWS utilizando la CLI y simulación local con **LocalStack**.

---

## 📐 Diagrama de la Arquitectura

![Arquitectura AWS VPC](images/01-arquitectura-vpc.png)

### Resumen de Componentes:
* **VPC:** CIDR `100.0.0.0/16` (`floci-vpc`)
* **Subred Pública:** `100.0.1.0/24` (Con salida a Internet mediante IGW)
* **Subred Privada:** `100.0.2.0/24` (Segmento aislado)
* **Internet Gateway:** `igw-96e96236`
* **Tabla de Rutas Pública:** `rtb-5b1cc3b7` (Ruta `0.0.0.0/0 -> IGW`)
* **Security Group:** `SG-floci` (`sg-66e40645534f10caa`) con puertos 80 (HTTP) y 443 (HTTPS) habilitados

---

## 🛠️ Paso a Paso por Terminal (AWS CLI)

### 1. Creación de la VPC
Se define el espacio de direccionamiento IP para la red virtual principal.

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-vpc \
    --cidr-block 100.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=floci-vpc}]'
2. Creación de Subredes (Pública y Privada)
Se divide la VPC en dos segmentos lógicos dentro de la misma Zona de Disponibilidad.

Bash
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
3. Conexión a Internet (Internet Gateway)
Se crea el gateway de Internet y se vincula a la VPC para permitir el tráfico hacia/desde el exterior.

Bash
# Crear Internet Gateway
aws --endpoint-url=http://localhost:4566 ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Internet-Gateway}]'

# Vincular a la VPC
aws --endpoint-url=http://localhost:4566 ec2 attach-internet-gateway \
    --vpc-id vpc-8076c853 \
    --internet-gateway-id igw-96e96236
4. Configuración de Tabla de Rutas y Enrutamiento
Se habilita el acceso público asociando la ruta por defecto (0.0.0.0/0) hacia el IGW únicamente en la Subred Pública.

Bash
# Crear Tabla de Rutas
aws --endpoint-url=http://localhost:4566 ec2 create-route-table \
    --vpc-id vpc-8076c853 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Tabla-Rutas-Publica}]'

# Agregar regla hacia Internet Gateway
aws --endpoint-url=http://localhost:4566 ec2 create-route \
    --route-table-id rtb-5b1cc3b7 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-96e96236

# Asociar la tabla a la Subred Pública
aws --endpoint-url=http://localhost:4566 ec2 associate-route-table \
    --subnet-id subnet-7af0455c \
    --route-table-id rtb-5b1cc3b7
5. Configuración de Firewall (Security Group)
Se define un Security Group con reglas de entrada (Inbound Rules) para habilitar el tráfico web.

Bash
# Crear el Security Group
aws --endpoint-url=http://localhost:4566 ec2 create-security-group \
    --group-name SG-floci \
    --description "firewall para web" \
    --vpc-id vpc-8076c853

# Regla de entrada: HTTP (Puerto 80)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Regla de entrada: HTTPS (Puerto 443)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0
🔍 Inspección y Verificación de Infraestructura
Para validar el correcto estado y asociación de los recursos provistos, se consultan las APIs correspondientes:

🚀 Prueba de Concepto (Servidor Web)
Se simula la ejecución de una instancia EC2 desplegada en la Subred Pública exponiendo un servidor web Nginx en el puerto 80:

Bash
docker run -d -p 80:80 --name servidor-web-prueba nginx
Resultado de la conectividad:

---

### 🚀 Para actualizarlo en tu GitHub:

En tu terminal, ejecuta:

```bash
git add VPC.md images/
git commit -m "docs: reestructura VPC.md con formato profesional para portafolio"
git push origin main
