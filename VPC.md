#!/bin/bash

################################################################################
# Script: push-to-github.sh
# Propósito: Automatizar la subida del proyecto AWS VPC a GitHub
# Autor: Cloud Infrastructure Team
# Uso: ./push-to-github.sh <username> <repository>
################################################################################

set -e  # Exit on error

# Colores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Función para imprimir con colores
print_info() {
    echo -e "${BLUE}ℹ️  $1${NC}"
}

print_success() {
    echo -e "${GREEN}✅ $1${NC}"
}

print_error() {
    echo -e "${RED}❌ $1${NC}"
}

print_warning() {
    echo -e "${YELLOW}⚠️  $1${NC}"
}

################################################################################
# VALIDACIÓN DE PARÁMETROS
################################################################################

if [ $# -lt 2 ]; then
    print_error "Faltan parámetros"
    echo ""
    echo "Uso: $0 <github-username> <repository-name>"
    echo ""
    echo "Ejemplo:"
    echo "  $0 SladeS vaw-aws-floci-training"
    echo ""
    exit 1
fi

GITHUB_USERNAME="$1"
REPOSITORY_NAME="$2"
REPO_URL="https://github.com/${GITHUB_USERNAME}/${REPOSITORY_NAME}.git"

print_info "Configuración:"
print_info "  Usuario GitHub: ${GITHUB_USERNAME}"
print_info "  Repositorio: ${REPOSITORY_NAME}"
print_info "  URL: ${REPO_URL}"
echo ""

################################################################################
# PASO 1: VALIDAR GIT
################################################################################

print_info "Paso 1: Validando Git..."

if ! command -v git &> /dev/null; then
    print_error "Git no está instalado"
    echo "Instala Git con: sudo apt-get install git"
    exit 1
fi

GIT_VERSION=$(git --version)
print_success "Git disponible: ${GIT_VERSION}"
echo ""

################################################################################
# PASO 2: VALIDAR ESTRUCTURA DEL PROYECTO
################################################################################

print_info "Paso 2: Validando estructura del proyecto..."

if [ ! -f "README.md" ]; then
    print_error "README.md no encontrado"
    exit 1
fi

if [ ! -d "Images" ]; then
    print_error "Carpeta 'Images' no encontrada"
    exit 1
fi

FILE_COUNT=$(find Images -type f | wc -l)
print_success "Estructura validada:"
print_success "  - README.md (encontrado)"
print_success "  - Carpeta Images/ con ${FILE_COUNT} archivos"
echo ""

################################################################################
# PASO 3: INICIALIZAR REPOSITORIO GIT
################################################################################

print_info "Paso 3: Inicializando repositorio Git..."

if [ -d ".git" ]; then
    print_warning "El repositorio Git ya existe"
    read -p "¿Deseas continuar? (s/n): " CONTINUE
    if [ "$CONTINUE" != "s" ]; then
        print_error "Operación cancelada"
        exit 1
    fi
else
    git init
    print_success "Repositorio Git inicializado"
fi

echo ""

################################################################################
# PASO 4: CONFIGURAR GIT (opcional)
################################################################################

print_info "Paso 4: Configurando Git..."

CURRENT_USER=$(git config user.name 2>/dev/null || echo "")
CURRENT_EMAIL=$(git config user.email 2>/dev/null || echo "")

if [ -z "$CURRENT_USER" ]; then
    read -p "Ingresa tu nombre (ej: Slade Sylvain): " GIT_USER
    git config user.name "$GIT_USER"
fi

if [ -z "$CURRENT_EMAIL" ]; then
    read -p "Ingresa tu email: " GIT_EMAIL
    git config user.email "$GIT_EMAIL"
fi

print_success "Usuario Git: $(git config user.name)"
print_success "Email Git: $(git config user.email)"
echo ""

################################################################################
# PASO 5: CREAR .gitignore
################################################################################

print_info "Paso 5: Creando .gitignore..."

if [ ! -f ".gitignore" ]; then
    cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/

# Node.js
node_modules/
npm-debug.log
yarn-error.log

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# LocalStack
.localstack/
localstack_logs/

# AWS
.aws/
*.pem
*.key

# Temporal
*.tmp
.temp/
EOF
    print_success ".gitignore creado"
else
    print_warning ".gitignore ya existe"
fi

echo ""

################################################################################
# PASO 6: AGREGAR ARCHIVOS AL STAGING
################################################################################

print_info "Paso 6: Agregando archivos al staging..."

git add README.md
git add Images/
git add .gitignore
git add *.sh 2>/dev/null || true

STAGED_FILES=$(git diff --cached --name-only | wc -l)
print_success "${STAGED_FILES} archivos agregados al staging"

git diff --cached --name-only | sed 's/^/  ✓ /'

echo ""

################################################################################
# PASO 7: CREAR COMMIT
################################################################################

print_info "Paso 7: Creando commit..."

COMMIT_MESSAGE="docs: AWS VPC Infrastructure deployment with LocalStack

- Provisioned VPC with 2 subnets (public/private)
- Configured Internet Gateway and routing
- Implemented Security Groups for web traffic
- Validated with Nginx deployment
- Includes complete AWS CLI commands and architecture diagrams

Project: AWS-with-floci-training
"

git commit -m "$COMMIT_MESSAGE"

print_success "Commit creado exitosamente"
echo ""

################################################################################
# PASO 8: AGREGAR REMOTE ORIGIN
################################################################################

print_info "Paso 8: Configurando remote origin..."

# Verificar si remote ya existe
if git remote get-url origin &>/dev/null; then
    CURRENT_URL=$(git remote get-url origin)
    print_warning "Remote origin ya existe: ${CURRENT_URL}"
    
    if [ "$CURRENT_URL" != "$REPO_URL" ]; then
        print_info "Actualizando remote origin..."
        git remote set-url origin "$REPO_URL"
    fi
else
    git remote add origin "$REPO_URL"
    print_success "Remote origin agregado"
fi

echo ""

################################################################################
# PASO 9: VERIFICAR RAMA PRINCIPAL
################################################################################

print_info "Paso 9: Verificando rama principal..."

CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
print_info "Rama actual: ${CURRENT_BRANCH}"

if [ "$CURRENT_BRANCH" != "main" ] && [ "$CURRENT_BRANCH" != "master" ]; then
    print_warning "Rama no es main/master"
    read -p "¿Deseas renombrar a 'main'? (s/n): " RENAME
    if [ "$RENAME" = "s" ]; then
        git branch -M main
        print_success "Rama renombrada a 'main'"
    fi
fi

echo ""

################################################################################
# PASO 10: PUSH A GITHUB
################################################################################

print_info "Paso 10: Preparando push a GitHub..."

echo ""
print_warning "IMPORTANTE: Antes de continuar, asegúrate de:"
echo "  1. Haber creado el repositorio en GitHub: https://github.com/new"
echo "  2. El repositorio debe llamarse: ${REPOSITORY_NAME}"
echo "  3. Estar autenticado en GitHub (usa Personal Access Token)"
echo ""

read -p "¿Continuar con el push? (s/n): " CONFIRM

if [ "$CONFIRM" != "s" ]; then
    print_warning "Push cancelado"
    echo ""
    print_info "Para hacer push más tarde, ejecuta:"
    echo "  git push -u origin main"
    exit 0
fi

echo ""
print_info "Realizando push..."

# Intentar push
if git push -u origin main 2>&1; then
    print_success "¡Push completado exitosamente!"
    echo ""
    print_success "Tu repositorio está disponible en:"
    echo "  ${REPO_URL}"
    echo ""
else
    print_error "El push falló. Posibles causas:"
    echo ""
    echo "  1. Repositorio no existe en GitHub"
    echo "     → Crea uno en https://github.com/new"
    echo ""
    echo "  2. Falta autenticación"
    echo "     → Usa GitHub CLI: gh auth login"
    echo "     → O configura SSH key"
    echo ""
    echo "  3. Rama remota existe"
    echo "     → Intenta: git push -f origin main"
    echo ""
    exit 1
fi

################################################################################
# PASO 11: VERIFICACIÓN FINAL
################################################################################

print_info "Paso 11: Verificación final..."

git log --oneline -1
print_success "¡Proyecto subido a GitHub exitosamente!"

echo ""
echo "═══════════════════════════════════════════════════════════"
print_success "RESUMEN"
echo "═══════════════════════════════════════════════════════════"
echo ""
print_success "✅ Repositorio inicializado"
print_success "✅ Archivos agregados al staging"
print_success "✅ Commit creado"
print_success "✅ Push a GitHub completado"
echo ""
echo "Próximos pasos:"
echo "  1. Visita tu repositorio: ${REPO_URL}"
echo "  2. Agrega una descripción y topics (aws, networking, infrastructure)"
echo "  3. Habilita GitHub Pages en Settings para hospedaje web"
echo ""
echo "═══════════════════════════════════════════════════════════"
echo ""
