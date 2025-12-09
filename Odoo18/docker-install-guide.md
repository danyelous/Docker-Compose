# 🐳 Odoo 18 Docker Setup

# I created it with the help of AI

A complete Docker Compose setup for Odoo 18 Community Edition with PostgreSQL database and pgAdmin for database management. This setup allows for easy customization, persistence, and development of Odoo modules.
📋 Features

    ✅ Odoo 18 Community Edition - Latest stable version

    ✅ PostgreSQL 15 - Production-ready database

    ✅ pgAdmin 4 - Web-based database administration

    ✅ Custom Modules Support - Mount your own modules

    ✅ Data Persistence - All data survives container restarts

    ✅ Health Checks - Automatic service monitoring

    ✅ Port Mapping - Customizable ports (starts from 8100)

    ✅ Easy Configuration - Simple environment variables

🚀 Quick Start
Prerequisites

    Docker Engine 20.10+

    Docker Compose 2.0+

    Git (optional)

Installation

    Clone or create the project structure:

bash

# Create project directory
mkdir odoo18-docker
cd odoo18-docker

# Create required directories
mkdir -p config pgadmin custom-addons

    Create the docker-compose.yml file:

yaml

version: '3.7'
services:
  postgres:
    image: postgres:15
    container_name: odoo_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: odoo18
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo_password
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --lc-collate=C --lc-ctype=C"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U odoo"]
      interval: 10s
      timeout: 5s
      retries: 10

  odoo:
    image: odoo:18.0
    container_name: odoo_app
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    ports:
      - "8100:8069"
      - "8107:8072"
    environment:
      HOST: postgres
      USER: odoo
      PASSWORD: odoo_password
      POSTGRES_DB: odoo18
    volumes:
      - odoo_data:/var/lib/odoo
      - ./custom-addons:/mnt/extra-addons
      - ./config/odoo.conf:/etc/odoo/odoo.conf:ro

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: odoo_pgadmin
    restart: unless-stopped
    depends_on:
      - postgres
    ports:
      - "8108:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin_password
    volumes:
      - pgadmin_data:/var/lib/pgadmin

volumes:
  postgres_data:
  odoo_data:
  pgadmin_data:

    Create Odoo configuration file:

bash

cat > config/odoo.conf << 'EOF'
[options]
admin_passwd = admin
db_host = postgres
db_port = 5432
db_user = odoo
db_password = odoo_password
db_name = odoo18
addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons
data_dir = /var/lib/odoo
EOF

    Start the services:

bash

# Start all containers
docker-compose up -d

# Wait for PostgreSQL to initialize
sleep 30

# Initialize Odoo database (ONE-TIME OPERATION)
docker-compose run --rm odoo odoo -d odoo18 -i base --stop-after-init

# Restart Odoo
docker-compose restart odoo

    Access the applications:

    🌐 Odoo: http://localhost:8100

    🗄️ pgAdmin: http://localhost:8108

🔧 Configuration
Default Credentials
Service	Username/Email	Password	Database
Odoo	admin	admin	odoo18
pgAdmin	admin@example.com	admin_password	N/A
PostgreSQL	odoo	odoo_password	odoo18
Port Configuration
Service	Container Port	Host Port	Description
Odoo	8069	8100	Main web interface
Odoo Longpolling	8072	8107	Real-time features
pgAdmin	80	8108	Database administration
Customizing Ports

Edit the ports section in docker-compose.yml:
yaml

ports:
  - "YOUR_PORT:8069"  # Change YOUR_PORT to desired port

📁 Project Structure
text

odoo18-docker/
├── docker-compose.yml          # Main Docker configuration
├── config/
│   └── odoo.conf              # Odoo configuration
├── pgadmin/
│   └── servers.json           # pgAdmin server config (optional)
└── custom-addons/             # Your custom Odoo modules
    └── my_module/
        ├── __manifest__.py
        ├── models/
        └── views/

🛠️ Custom Module Development

    Create a new module:

bash

mkdir -p custom-addons/my_module/{models,views,security}

    Create __manifest__.py:

python

{
    'name': 'My Custom Module',
    'version': '1.0',
    'summary': 'Custom module for Odoo 18',
    'category': 'Tools',
    'author': 'Your Name',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/my_view.xml',
    ],
    'installable': True,
    'application': True,
}

    Install in Odoo:

        Go to Apps → Update Apps List

        Search for your module

        Click Install

📊 Database Management
Using pgAdmin

    Access http://localhost:8108

    Login with: admin@example.com / admin_password

    Right-click "Servers" → Register → Server

    Connection details:

        Name: Odoo PostgreSQL

        Host: postgres

        Port: 5432

        Username: odoo

        Password: odoo_password

Backup Database
bash

# Backup to file
docker-compose exec postgres pg_dump -U odoo odoo18 > backup_$(date +%Y%m%d).sql

# Backup using Docker volume
docker run --rm -v odoo18-docker_postgres_data:/source -v $(pwd):/backup alpine tar czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz -C /source .

Restore Database
bash

# Restore from SQL dump
cat backup.sql | docker-compose exec -T postgres psql -U odoo odoo18

# Restore Docker volume
docker run --rm -v odoo18-docker_postgres_data:/target -v $(pwd):/backup alpine tar xzf /backup/postgres_backup.tar.gz -C /target

🚨 Troubleshooting
Common Issues

    Odoo shows 500 error:

bash

# Check Odoo logs
docker-compose logs odoo

# Re-initialize database
docker-compose run --rm odoo odoo -d odoo18 -i base --stop-after-init
docker-compose restart odoo

    PostgreSQL won't start:

bash

# Remove corrupted volume
docker-compose down -v
docker-compose up -d

    Port already in use:

bash

# Check what's using the port
sudo lsof -i :8100

# Change port in docker-compose.yml

    Can't access pgAdmin:

bash

# Check pgAdmin logs
docker-compose logs pgadmin

# Ensure PostgreSQL is running
docker-compose ps

Useful Commands
bash

# View logs
docker-compose logs -f odoo
docker-compose logs -f postgres

# Enter container shell
docker-compose exec odoo bash
docker-compose exec postgres bash

# Restart specific service
docker-compose restart odoo

# Check service status
docker-compose ps

# Update images
docker-compose pull

# Stop and clean everything
docker-compose down -v

🔒 Security Considerations
For Production Use:

    Change all default passwords in docker-compose.yml and config/odoo.conf

    Use SSL/TLS for database connections

    Set up firewall rules to restrict access

    Enable Odoo's built-in security features

    Regularly update Docker images

    Implement backup strategy

Password Management:

Create a .env file for sensitive data:
bash

# .env file
POSTGRES_PASSWORD=your_secure_password
ODOO_ADMIN_PASSWORD=your_secure_odoo_password
PGADMIN_PASSWORD=your_secure_pgadmin_password

Update docker-compose.yml to use environment variables:
yaml

environment:
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

📈 Monitoring & Maintenance
Health Checks

Services include health checks that automatically monitor:

    PostgreSQL database connectivity

    Odoo web service availability

Logs Location

    Odoo logs: docker-compose logs odoo

    PostgreSQL logs: docker-compose logs postgres

    pgAdmin logs: docker-compose logs pgadmin

Performance Tuning

For better performance, consider adding to config/odoo.conf:
ini

; Increase workers for multi-core systems
workers = 4

; Adjust memory limits
limit_memory_hard = 2684354560
limit_memory_soft = 2147483648

; Database connection pooling
db_maxconn = 64

🤝 Contributing

    Fork the repository

    Create a feature branch

    Commit your changes

    Push to the branch

    Create a Pull Request

📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
🙏 Acknowledgments

    Odoo Community for the amazing ERP platform

    PostgreSQL for the robust database

    pgAdmin for database management

    Docker for containerization technology

📞 Support

For issues and questions:

    Check the Troubleshooting section

    Search existing GitHub issues

    Create a new issue with detailed information




Opción 1: Conectar Manualmente en pgAdmin

    En pgAdmin (http://192.168.0.224:8108):

        Usuario: admin@example.com

        Contraseña: admin_password

    Crear Nueva Conexión:

        Click derecho en "Servers" → Register → Server

    Parámetros de Conexión:

🔗 Conexión Manual a PostgreSQL desde pgAdmin

Esta guía explica cómo configurar manualmente la conexión a la base de datos de Odoo desde pgAdmin cuando se usa Docker Compose.
📌 Información de Conexión

Importante: Cuando usas Docker Compose, los servicios se comunican entre sí usando sus nombres de servicio, no localhost.
Credenciales de Acceso a pgAdmin

    URL: http://localhost:8108 (o http://TU_IP:8108)

    Email: admin@example.com

    Password: admin_password

Datos de la Base de Datos Odoo
Parámetro	Valor	Nota
Servidor	postgres	Nombre del servicio en docker-compose.yml
Puerto	5432	Puerto interno de PostgreSQL
Base de datos	odoo18	Base de datos principal de Odoo
Usuario	odoo	Usuario específico de Odoo
Contraseña	odoo_password	Contraseña del usuario odoo
🚀 Pasos para Configurar la Conexión
Paso 1: Acceder a pgAdmin

    Abre tu navegador y ve a: http://localhost:8108

    Inicia sesión con:

        Email: admin@example.com

        Password: admin_password

Paso 2: Registrar Nuevo Servidor

    En el panel izquierdo, haz clic derecho en "Servers"

    Selecciona "Register" → "Server..."

Paso 3: Configurar Conexión (Pestaña General)
General Tab:
Campo	Valor a Ingresar
Name	Odoo PostgreSQL (o el nombre que prefieras)
Connection Tab:
Campo	Valor a Ingresar	Explicación
Host name/address	postgres	⚠️ NO usar localhost
Port	5432	Puerto por defecto de PostgreSQL
Maintenance database	postgres	Base de datos por defecto
Username	odoo	Usuario configurado en docker-compose.yml
Password	odoo_password	Contraseña del usuario odoo
Save password?	✅ Marcar	Para no tener que ingresarla cada vez
