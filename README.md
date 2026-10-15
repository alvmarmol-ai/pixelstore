PixelStore — Ecosistema Full-Stack Contenerizado y Optimizado

Bienvenido al repositorio oficial del proyecto PixelStore, desarrollado como solución para la actividad evaluada de Ingeniería de ImáProject Readmegenes Eficientes: Refactorización Multi-stage y Optimización de Capas en Docker.

Este proyecto implementa un entorno de producción real, ligero, seguro y reproducible compuesto por tres servicios principales orquestados mediante Docker Compose.

🏗 Arquitectura del Ecosistema

El sistema se divide en tres contenedores independientes que se comunican a través de una red interna de Docker:

Frontend (Next.js 16+ con App Router):

Construido mediante un proceso multi-stage utilizando node:20-alpine.

Separa la etapa de instalación de dependencias (deps), compilación (builder) y ejecución ligera (runner), excluyendo código fuente innecesario y reduciendo drásticamente el tamaño de la imagen.

Expuesto al host en el puerto 3000.

Backend (Laravel con PHP 8.2-FPM y Nginx):

Construido sobre una imagen base php:8.2-fpm-alpine.

Incorpora extensiones críticas (pdo_mysql, mbstring, openssl, tokenizer, xml, ctype, json).

Utiliza una etapa independiente con composer:2 para empaquetar las dependencias (vendor/) sin arrastrar herramientas de desarrollo pesadas a producción.

Cuenta con un servidor Nginx interno actuando como proxy inverso para exponer la API eficientemente por el puerto 8000.

Base de Datos (MySQL 8.0):

Instancia aislada de mysql:8.0.

El puerto interno 3306 está protegido y mapeado de forma segura (ej. al puerto 3307 en desarrollo para evitar conflictos locales).

Utiliza volúmenes persistentes (mysql_data) para garantizar que la información no se pierda al reiniciar los contenedores.

🚀 Instrucciones para Ejecutar el Ecosistema

Sigue estos pasos para levantar todo el proyecto con un solo comando:

1. Clonar o configurar el repositorio

Asegúrate de estar en la raíz del proyecto donde se encuentra el archivo docker-compose.yml.

2. Configurar las variables de entorno

Copia el archivo de ejemplo .env.example y renómbralo a .env:

cp .env.example .env


(Puedes ajustar las credenciales de la base de datos y puertos según tus necesidades).

3. Levantar los contenedores con Docker Compose

Ejecuta el siguiente comando para construir las imágenes optimizadas y poner en marcha los servicios en segundo plano:

docker compose up --build -d


4. Ejecutar las migraciones de la base de datos

Una vez que los servicios estén activos, inicializa las tablas de Laravel ejecutando:

MSYS_NO_PATHCONV=1 docker compose exec backend php artisan migrate


5. Acceder a los servicios

Frontend (Next.js): Abre tu navegador en http://localhost:3000

Backend API (Laravel): Accesible internamente por el proxy Nginx en el puerto 8000.

🛠 Explicación de las Decisiones Técnicas en los Dockerfiles

1. Frontend (frontend/Dockerfile)

Multi-stage builds: Se utiliza una primera etapa para instalar dependencias con npm ci aprovechando la caché estricta de capas. Una segunda etapa compila la aplicación ejecutando npm run build. La etapa final de producción (runner) parte de una imagen limpia basada en Alpine, copiando únicamente los artefactos estrictamente necesarios (.next/standalone, .next/static, y archivos públicos), lo que reduce el tamaño final en más de un 70%.

Seguridad y Usuario: Se configura un usuario no privilegiado (nextjs) para mitigar riesgos de vulnerabilidades por ejecución como root.

2. Backend (backend/Dockerfile)

Separación de dependencias (Composer + PHP-FPM): Se utiliza una etapa de compilación utilizando la imagen oficial de Composer para empaquetar los vendors de producción (composer install --no-dev --optimize-autoloader).

Librerías del Sistema y Extensiones PHP: Se compilan e instalan de forma limpia utilizando gestores de paquetes ligeros de Alpine (apk add), limpiando cachés temporales en la misma capa para evitar inflar el tamaño de la imagen final.

Proxy Inverso con Nginx: Se integra Nginx directamente en el contenedor del backend para gestionar las peticiones HTTP y traducirlas eficientemente al protocolo FastCGI de PHP-FPM.

version: '3.8'

