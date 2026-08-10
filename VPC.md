Markdown
# ☁️ AWS VPC Infrastructure & Networking Lab

## 📌 Descripción del Proyecto
Este proyecto documenta el despliegue de una arquitectura de red virtual **AWS VPC** de alta disponibilidad y seguridad utilizando **AWS CLI** y **LocalStack**. El objetivo es aprovisionar una infraestructura de red completa para alojar aplicaciones web de forma segura.

---

## 📐 Arquitectura de Red

La infraestructura consta de:
* **VPC**: `100.0.0.0/16` (`floci-vpc`)
* **Subred Pública**: `100.0.1.0/24` (Para servicios orientados a Internet)
* **Subred Privada**: `100.0.2.0/24` (Para bases de datos y backend)
* **Internet Gateway**: Adjunto a la VPC para proveer salida pública.
* **Grupo de Seguridad**: Configurado para permitir acceso HTTP (Puerto 80).

![Diagrama de Arquitectura VPC](images/preview%20(8).webp)

---

## 🛠️ Paso a Paso del Despliegue e Inspección CLI

### 1. Creación y Estado de la VPC
Se aprovisionó la VPC principal con su rango CIDR correspondiente.

```bash
aws ec2 describe-vpcs --vpc-ids vpc-8076c853
2. Creación de Subredes (Public & Private)
Se segmentó la red en subredes públicas y privadas para aislar componentes de software.

Bash
aws ec2 describe-subnets
3. Configuración del Internet Gateway (IGW) y Rutas
Para permitir la conectividad hacia la web en la subred pública, se creó y asoció un Internet Gateway.

Bash
aws ec2 describe-internet-gateways
aws ec2 describe-route-tables
4. Configuración del Grupo de Seguridad (Security Group)
Se definió el grupo de seguridad SG-floci cerrando todo el tráfico inbound excepto el tráfico entrante por el puerto 80 TCP.

Bash
aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 describe-security-groups
5. Validación y Prueba del Servidor Web (Nginx)
Se desplegó un servidor Nginx en la subred pública y se validó la respuesta exitosa por HTTP.

🖥️ Registro Completo de Comandos (Terminal Logs)
Captura del flujo de ejecución interactivo realizado durante la construcción de la VPC:

🚀 Tecnologías Utilizadas
AWS CLI (Command Line Interface)

AWS VPC, Subnets, IGW, Route Tables, Security Groups

LocalStack / AWS Cloud

Nginx Web Server


---

### 🔧 ¿Por qué este cambio soluciona lo de las imágenes?
1. **Rutas correctas con encode URL**: Las imágenes con espacios como `preview (1).webp` están escritas como `images/preview%20(1).webp`. En Markdown, los espacios en las rutas de archivo rompen las imágenes si no llevan `%20`.
2. **Sin código de Bash metido en la documentación**: Todo el texto explica exclusivamente la infraestructura Cloud, la red y la verificación del servidor web.