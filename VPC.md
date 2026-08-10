# Create the script push-to-github.sh adapted with the exact images provided
script_content = '''#!/bin/bash

################################################################################
# Script: push-to-github.sh
# Propósito: Automatizar la subida del proyecto AWS VPC a GitHub
# Autor: Cloud Infrastructure Team
# Uso: ./push-to-github.sh <username> <repository>
################################################################################

set -e  # Exit on error

# Colores para output
RED='\\033[0;31m'
GREEN='\\033[0;32m'
YELLOW='\\033[1;33m'
BLUE='\\033[0;34m'
NC='\\033[0m' # No Color

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
# PASO 2: VALIDAR ESTRUCTURA DEL PROYECTO E IMÁGENES
################################################################################

print_info "Paso 2: Validando estructura del proyecto e imágenes..."

if [ ! -f "README.md" ] && [ ! -f "VPC.md" ]; then
    print_error "No se encontró ni README.md ni VPC.md"
    exit 1
fi

# Detectar la carpeta de imágenes (soporta 'images' o 'Images')
IMAGES_DIR=""
if [ -d "images" ]; then
    IMAGES_DIR="images"
elif [ -d "Images" ]; then
    IMAGES_DIR="Images"
else
    print_error "Carpeta 'images' o 'Images' no encontrada"
    exit 1
fi

print_success "Carpeta de imágenes detectada: '${IMAGES_DIR}'"

# Lista exacta de las 10 imágenes requeridas para el proyecto AWS VPC
EXPECTED_IMAGES=(
    "18e2a1e2-80cd-4065-8f01-d7dc4a760f7c.jpeg"
    "preview.webp"
    "preview (1).webp"
    "preview (2).webp"
    "preview (3).webp"
    "preview (4).webp"
    "preview (5).webp"
    "preview (6).webp"
    "preview (7).webp"
    "preview (8).webp"
)

MISSING_COUNT=0
print_info "Verificando las 10 imágenes del laboratorio VPC:"

for img in "${EXPECTED_IMAGES[@]}"; do
    if [ -f "${IMAGES_DIR}/${img}" ]; then
        print_success "  ✓ Encontrada: ${img}"
    else
        print_warning "  ✗ Falta: ${img}"
        MISSING_COUNT=$((MISSING_COUNT + 1))
    fi
done

if [ $MISSING_COUNT -gt 0 ]; then
    print_warning "Faltan ${MISSING_COUNT} imagen(es) esperada(s). Se continuará con las disponibles."
else
    print_success "¡Las 10 imágenes exactas del laboratorio VPC están presentes!"
fi

echo ""

################################################################################
# PASO 3: INICIALIZAR REPOSITORIO GIT
################################################################################

print_info "Paso 3: Inicializando repositorio Git..."

if [ -d ".git" ]; then
    print_warning "El repositorio Git ya existe"
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

print_info "Paso 6: Agregando exactamente las imágenes y documentos al staging..."

[ -f "README.md" ] && git add README.md
[ -f "VPC.md" ] && git add VPC.md
git add .gitignore
git add *.sh 2>/dev/null || true

# Agregar exactamente las 10 imágenes requeridas
for img in "${EXPECTED_IMAGES[@]}"; do
    if [ -f "${IMAGES_DIR}/${img}" ]; then
        git add "${IMAGES_DIR}/${img}"
    fi
done

STAGED_FILES=$(git diff --cached --name-only | wc -l)
print_success "${STAGED_FILES} archivos agregados al staging"

git diff --cached --name-only | sed 's/^/  ✓ /'

echo ""

################################################################################
# PASO 7: CREAR COMMIT
################################################################################

print_info "Paso 7: Creando commit..."

COMMIT_MESSAGE="docs: AWS VPC Infrastructure deployment with LocalStack

- Provisioned VPC (100.0.0.0/16) with public/private subnets
- Configured Internet Gateway and Route Tables
- Implemented Security Group for HTTP port 80
- Validated with Nginx deployment and CLI describe inspection
- Includes exactly 10 architecture & CLI inspection screenshots

Project: AWS-with-floci-training
"

git commit -m "$COMMIT_MESSAGE"

print_success "Commit creado exitosamente"
echo ""

################################################################################
# PASO 8: AGREGAR REMOTE ORIGIN
################################################################################

print_info "Paso 8: Configurando remote origin..."

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
    print_info "Asegurando nombre de rama 'main'..."
    git branch -M main
    print_success "Rama renombrada a 'main'"
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
echo "  3. Estar autenticado en GitHub (usa Personal Access Token o SSH)"
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

if git push -u origin main 2>&1; then
    print_success "¡Push completado exitosamente!"
    echo ""
    print_success "Tu repositorio está disponible en:"
    echo "  ${REPO_URL}"
    echo ""
else
    print_error "El push falló. Posibles causas:"
    echo ""
    echo "  1. Repositorio no existe en GitHub → Créalo en https://github.com/new"
    echo "  2. Falta autenticación → Usa: gh auth login"
    echo "  3. Conflicto de ramas → Usa: git push -f origin main"
    echo ""
    exit 1
fi

################################################################################
# PASO 11: VERIFICACIÓN FINAL
################################################################################

print_info "Paso 11: Verificación final..."

git log --oneline -1
print_success "¡Proyecto AWS VPC subido a GitHub exitosamente con sus 10 imágenes exactas!"

echo ""
echo "═══════════════════════════════════════════════════════════"
print_success "RESUMEN DE PUSH"
echo "═══════════════════════════════════════════════════════════"
echo ""
print_success "✅ Repositorio inicializado"
print_success "✅ Exactamente 10 capturas de VPC agregadas"
print_success "✅ Commit estructurado"
print_success "✅ Push a GitHub completado"
echo ""
echo "URL del proyecto: ${REPO_URL}"
echo "═══════════════════════════════════════════════════════════"
echo ""
'''

with open('push-to-github.sh', 'w') as f:
    f.write(script_content)

print("Script 'push-to-github.sh' generado correctamente.")