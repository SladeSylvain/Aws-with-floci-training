# 🌩️ Floci: AWS VPC Infrastructure Deployment

> AWS VPC provisioning with LocalStack | Infrastructure as Code | AWS CLI

---

## 📚 Tabla de Contenidos

- [Descripción](#descripción)
- [Arquitectura](#arquitectura)
- [Requisitos](#requisitos)
- [Implementación](#implementación)
- [Validación](#validación)
- [Prueba de Concepto](#prueba-de-concepto)

---

## 🎯 Descripción

**Floci** — Portafolio de infraestructura cloud en AWS. Este proyecto demuestra:

✅ Diseño y provisión de **VPCs** con segmentación de red  
✅ Implementación de **Security Groups** con reglas stateful  
✅ Configuración de **Internet Gateways** y tablas de rutas  
✅ Automatización mediante **AWS CLI** (Infrastructure as Code)  
✅ Validación con **LocalStack** antes de desplegar en AWS real  

### Caso de Uso

Establecer un **segmento de red aislado y seguro** en AWS para alojar aplicaciones web (Nginx) con control granular de acceso y conectividad controlada.

**Tecnologías:** AWS, AWS CLI, LocalStack, Docker, Bash, Git

---

## 🏗️ Arquitectura

### Diagrama Visual

![Arquitectura VPC - Floci](./Images/11-vpc-architecture-diagram.png)

### Especificación

| Componente | CIDR/ID | Descripción | Estado |
|-----------|---------|-------------|--------|
| **VPC** | `100.0.0.0/16` (vpc-8076c853) | Red virtual principal | ✅ Active |
| **Subred Pública** | `100.0.1.0/24` (subnet-7af0455c) | Recursos accesibles desde Internet | ✅ Active |
| **Subred Privada** | `100.0.2.0/24` (subnet-2f0593a0) | Recursos aislados | ✅ Active |
| **Internet Gateway** | igw-96e96236 | Conectividad externa | ✅ Attached |
| **Route Table** | rtb-5b1cc3b7 | Enrutamiento (0.0.0.0/0 → IGW) | ✅ Associated |
| **Security Group** | sg-66e40645534f10caa | Firewall (Puerto 80, 443) | ✅ Active |

---

## 📦 Requisitos

### Software Necesario

```bash
# AWS CLI v2+
aws --version

# Docker (para LocalStack)
docker --version

# Git
git --version
```

### Configuración Inicial

```bash
# 1. Configurar AWS CLI
aws configure --profile localstack
# AWS Access Key ID: test
# AWS Secret Access Key: test
# Default region: us-east-1

# 2. Iniciar LocalStack
docker run -d -p 4566:4566 \
  -e SERVICES=ec2,vpc \
  localstack/localstack:latest

# 3. Verificar conexión
aws --endpoint-url=http://localhost:4566 ec2 describe-vpcs
```

---

## 🔧 Implementación

### 1. Crear VPC

```bash
aws --endpoint-url=http://localhost:4566 ec2 create-vpc \
    --cidr-block 100.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=floci-vpc}]'
```

![VPC Creation](./Images/01-vpc-create-first.png)

**Resultado:** `vpc-8076c853`

---

### 2. Crear Subredes

#### Subred Pública
```bash
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Publica}]'
```

#### Subred Privada
```bash
aws --endpoint-url=http://localhost:4566 ec2 create-subnet \
    --vpc-id vpc-8076c853 \
    --cidr-block 100.0.2.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Subred-Privada}]'
```

![Subnets Validation](./Images/05-subnets-describe.png)

---

### 3. Internet Gateway

```bash
# Crear
aws --endpoint-url=http://localhost:4566 ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=floci-igw}]'

# Adjuntar a VPC
aws --endpoint-url=http://localhost:4566 ec2 attach-internet-gateway \
    --vpc-id vpc-8076c853 \
    --internet-gateway-id igw-96e96236
```

![Internet Gateway](./Images/04-internet-gateway-describe.png)

---

### 4. Tablas de Rutas

```bash
# Crear
aws --endpoint-url=http://localhost:4566 ec2 create-route-table \
    --vpc-id vpc-8076c853 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Tabla-Rutas-Publica}]'

# Agregar ruta default
aws --endpoint-url=http://localhost:4566 ec2 create-route \
    --route-table-id rtb-5b1cc3b7 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-96e96236

# Asociar a subred pública
aws --endpoint-url=http://localhost:4566 ec2 associate-route-table \
    --subnet-id subnet-7af0455c \
    --route-table-id rtb-5b1cc3b7
```

![Route Tables](./Images/03-route-tables-describe.png)

---

### 5. Security Groups

```bash
# Crear
aws --endpoint-url=http://localhost:4566 ec2 create-security-group \
    --group-name SG-floci \
    --description "Firewall para aplicaciones web" \
    --vpc-id vpc-8076c853

# Autorizar HTTP (80)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Autorizar HTTPS (443)
aws --endpoint-url=http://localhost:4566 ec2 authorize-security-group-ingress \
    --group-id sg-66e40645534f10caa \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0
```

![Security Groups](./Images/07-sg-complete-inspect.png)

---

## ✅ Validación

### Inspeccionar VPC

```bash
aws --endpoint-url=http://localhost:4566 ec2 describe-vpcs --vpc-ids vpc-8076c853
```

![VPC Validation](./Images/06-vpc-describe-output.png)

### Inspeccionar Security Groups

```bash
aws --endpoint-url=http://localhost:4566 ec2 describe-security-groups \
    --group-ids sg-66e40645534f10caa
```

![Security Groups Validation](./Images/02-sg-describe-output.png)

### Inspeccionar Route Tables

```bash
aws --endpoint-url=http://localhost:4566 ec2 describe-route-tables \
    --route-table-ids rtb-5b1cc3b7
```

---

## 🧪 Prueba de Concepto

### Desplegar Nginx

```bash
docker run -d -p 80:80 --name floci-nginx nginx:latest
```

### Verificar Conectividad

```bash
curl http://localhost
# Expected: HTTP 200 OK + Welcome to nginx!
```

![Nginx Validation](./Images/10-nginx-welcome-page.png)

---

## 📊 Resumen

| Métrica | Valor |
|---------|-------|
| Comandos AWS CLI | 10 |
| Recursos creados | 6 |
| Direcciones IP | ~500 |
| Puertos abiertos | 2 (80, 443) |
| Tiempo de setup | ~5 minutos |
| Costo AWS | $0 (LocalStack) |

---

## 🚀 Próximos Proyectos en Floci

1. **Floci-RDS** — Base de datos relacional
2. **Floci-LB** — Load Balancer & Auto Scaling
3. **Floci-MultiAZ** — Alta disponibilidad
4. **Floci-VPN** — Conectividad VPN
5. **Floci-CDN** — CloudFront
6. **Floci-Monitoring** — CloudWatch
7. **Floci-Terraform** — IaC avanzado
8. **Floci-ECS** — Contenedores
9. **Floci-Lambda** — Serverless
10. **Floci-Pipeline** — CI/CD

---

## 📚 Referencias

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [LocalStack Docs](https://docs.localstack.cloud/)

---

**Floci** — Cloud Infrastructure Portfolio | AWS | Networking | IaC

*Versión 1.0.0 | Agosto 2026 | MIT License*
