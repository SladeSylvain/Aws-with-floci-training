## 📸 1. Listado de Imágenes Recomendadas

Crea o utiliza la carpeta **`images/`** en la raíz de tu repositorio y sube estas capturas guardándolas con estos nombres exactos:

| Nombre de archivo en `images/` | ¿Qué captura debes subir aquí? |
| --- | --- |
| **`01-arquitectura-vpc.png`** | Tu esquema visual de AWS con la VPC, subredes, EC2 e Internet Gateway (la primera foto que me enviaste). |
| **`02-creacion-vpc-subredes.png`** | Captura de la terminal ejecutando `describe-vpcs` y `describe-subnets` mostrando el JSON. |
| **`03-igw-tabla-rutas.png`** | Captura de la terminal ejecutando `describe-internet-gateways` y `describe-route-tables` con la asociación `associated`. |
| **`04-security-group.png`** | Captura de la terminal ejecutando `describe-security-groups` con las reglas de los puertos 80 y 443. |
| **`05-prueba-nginx-exitosa.png`** | Captura del navegador con la pantalla de **`Welcome to nginx!`**. |

---

## 📄 2. Contenido para tu nuevo archivo `GUIA_PASO_A_PASO.md`

Puedes crear el archivo en tu proyecto desde la terminal:

```bash
touch GUIA_PASO_A_PASO.md

```

Abre **`GUIA_PASO_A_PASO.md`** y copia este código Markdown completo:

```markdown
# ☁️ Despliegue de Infraestructura VPC en AWS con LocalStack & AWS CLI

Guía técnica paso a paso orientada a portafolio que documenta la creación, configuración y verificación de una arquitectura de red en AWS (**Virtual Private Cloud**) en un entorno simulado local mediante **LocalStack**.

---

## 📐 Diagrama de Arquitectura

![Arquitectura AWS VPC](images/01-arquitectura-vpc.png)

### Especificaciones de Red:
* **VPC CIDR:** `100.0.0.0/16` (`floci-vpc`)
* **Subred Pública:** `100.0.1.0/24` (Con acceso a Internet mediante IGW)
* **Subred Privada:** `100.0.2.0/24` (Aislada)
* **Internet Gateway:** `igw-96e96236`
* **Security Group:** `SG-floci` (Puertos HTTP 80 y HTTPS 443 habilitados)

---

## 🛠️ Guía de Ejecución Paso a Paso por Terminal

### Paso 1: Crear la VPC Base
Definimos el bloque CIDR principal para nuestro espacio de direcciones de red virtual:

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-vpc \
    --cidr-block 100.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=floci-vpc}]'

```

---

### Paso 2: Crear las Subredes (Pública y Privada)

Segmentamos la VPC en dos salas independientes dentro de la misma Zona de Disponibilidad:

#### Subred Pública (`100.0.1.0/24`):

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Publica}]'

```

#### Subred Privada (`100.0.2.0/24`):

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.2.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Privada}]'

```

---

### Paso 3: Configurar Internet Gateway y Conectarlo a la VPC

Para dar capacidad de tráfico hacia el exterior, creamos el IGW y lo atornillamos a nuestra VPC:

#### 1. Crear el Internet Gateway:

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Internet-Gateway}]'

```

#### 2. Adjuntar (Attach) el IGW a la VPC:

```bash
aws --endpoint-url=http://localhost:4566 ec2 attach-internet-gateway \
    --vpc-id vpc-8076c853 \
    --internet-gateway-id igw-96e96236

```

---

### Paso 4: Configurar la Tabla de Rutas Pública

Establecemos las reglas de tránsito de paquetes y asociamos la tabla **únicamente** a la subred pública.

#### 1. Crear la Tabla de Rutas:

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-route-table \
    --vpc-id vpc-8076c853 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Tabla-Rutas-Publica}]'

```

#### 2. Agregar la Ruta por Defecto hacia Internet (`0.0.0.0/0 -> IGW`):

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-route \
    --route-table-id rtb-5b1cc3b7 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-96e96236

```

#### 3. Asociar la Tabla de Rutas a la Subred Pública:

```bash
aws --endpoint-url=http://localhost:4566 ec2 associate-route-table \
    --subnet-id subnet-7af0455c \
    --route-table-id rtb-5b1cc3b7

```

---

### Paso 5: Crear Firewall y Definir Reglas de Entrada (Security Group)

Definimos las *Inbound Rules* a nivel de firewall de instancia para permitir tráfico web entrante.

#### 1. Crear el Security Group:

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-security-group \
    --group-name SG-floci \
    --description "firewall para web" \
    --vpc-id vpc-8076c853

```

#### 2. Habilitar Puertos HTTP (80) y HTTPS (443):

```bash
# Puerto 80 (HTTP)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Puerto 443 (HTTPS)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

```

---

## 🔍 Verificación e Inspección de Infraestructura

Podemos consultar el estado completo de los recursos desplegados en LocalStack con los siguientes comandos `describe`:

```bash
# Consultar VPC y Subredes
aws --endpoint-url=http://localhost:4566 ec2 describe-vpcs --vpc-ids vpc-8076c853
aws --endpoint-url=http://localhost:4566 ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-8076c853"

# Consultar Tabla de Rutas e IGW
aws --endpoint-url=http://localhost:4566 ec2 describe-route-tables --route-table-ids rtb-5b1cc3b7

# Consultar Firewall
aws --endpoint-url=http://localhost:4566 ec2 describe-security-groups --group-ids sg-66e40645534f10caa

```

---

## 🚀 Prueba de Conectividad de Servidor Web (Nginx)

Para validar la topología en vivo, se simula el lanzamiento de una instancia EC2 dentro de la subred pública y se expone un servidor web Nginx en el puerto 80:

```bash
docker run -d -p 80:80 --name servidor-web-prueba nginx

```

### Resultado de la Prueba:

Al acceder localmente vía HTTP, confirmamos que las reglas de enrutamiento y puertos responden correctamente:

```

---

## ⬆️ Paso Final: Subir todo a tu GitHub por terminal

Ejecuta estos tres comandos para mandar el nuevo archivo y tus imágenes a tu repositorio:

```bash
git add GUIA_PASO_A_PASO.md images/
git commit -m "docs: agrega guia detallada paso a paso de VPC y capturas del proyecto"
git push origin main

```
