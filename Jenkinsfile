def COLOR_MAP = [
  'SUCCESS': 'good',
  'FAILURE':'danger',
]

pipeline {
    agent any
    
    environment {
        // Maven and JDK tools defined in Jenkins
        MAVEN_HOME = tool 'apache-maven-3.9.8'
        JAVA_HOME = tool 'jdk 21'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"

        // Docker environment
        NETWORK_NAME = 'gestion_eventos_network'
        PG_CONTAINER = 'pg_container'
        APP_IMAGE = 'gestion_eventos_app_image'
        APP_CONTAINER = 'gestion_eventos_app'
        POSTGRES_USER = 'postgres'
        POSTGRES_PASSWORD = 'admin'
        POSTGRES_DB = 'gestion_eventos'
        SPRING_DATASOURCE_URL = "jdbc:postgresql://pg_container:5432/gestion_eventos"
        SPRING_DATASOURCE_USERNAME = 'postgres'
        SPRING_DATASOURCE_PASSWORD = 'admin'
        SPRING_DATASOURCE_DRIVER_CLASS_NAME = 'org.postgresql.Driver'
        SPRING_JPA_DATABASE_PLATFORM = 'org.hibernate.dialect.PostgreSQLDialect'
        SERVER_PORT = '8082'
        SPRING_APPLICATION_NAME = 'G2-GestionEventos'
        //PROJECT_PATH = 'C:\\Users\\mmf27\\Documents\\Repositorios\\TD_Kibernum_Devops_Grupo2AmericanSummer-GestionEventos'

        // Define el SonarQube Server a utilizar
        SONARQUBE_URL = 'http://localhost:9000'
        scannerHome = tool 'SonarQubeScanner'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clonar el repositorio
                git url: 'https://github.com/cristianjonhson/TD_Kibernum_Devops_Grupo2AmericanSummer-GestionEventos.git', branch: 'feature/PT2-45-JM'
            }
        }

        stage('Build Project with Maven') {
            steps {
                // Construir el proyecto con Maven
                sh 'mvn -f pom.xml clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-10.6.0.92116') { // Asegúrate de que el nombre coincide con la configuración de Jenkins
                    sh '${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=Grupo2AmericanSummer-GestionEventos \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.java.binaries=target/classes'
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                // Ejecutar pruebas con Maven
                sh 'mvn -f pom.xml test'
            }
            post {
                always {
                    // Publicar resultados de pruebas
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Create Docker Network') {
            steps {
                script {
                    sh '''
                    docker network create ${NETWORK_NAME} || echo "Network already exists"
                    '''
                }
            }
        }

        stage('Start PostgreSQL Container') {
            steps {
                script {
                    sh '''
                    docker run -d --name ${PG_CONTAINER} --network ${NETWORK_NAME} \
                        -e POSTGRES_USER=${POSTGRES_USER} \
                        -e POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
                        -e POSTGRES_DB=${POSTGRES_DB} \
                        -p 5432:5432 postgres:latest
                    '''
                }
            }
        }

        stage('Build Application Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t ${APP_IMAGE} .
                    '''
                }
            }
        }

        stage('Start Application Container') {
            steps {
                script {
                    sh '''
                    docker run -d --name ${APP_CONTAINER} --network ${NETWORK_NAME} \
                        -e "SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL}" \
                        -e "SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME}" \
                        -e "SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}" \
                        -e "SPRING_DATASOURCE_DRIVER_CLASS_NAME=${SPRING_DATASOURCE_DRIVER_CLASS_NAME}" \
                        -e "SPRING_JPA_DATABASE_PLATFORM=${SPRING_JPA_DATABASE_PLATFORM}" \
                        -e "SERVER_PORT=${SERVER_PORT}" \
                        -e "SPRING_APPLICATION_NAME=${SPRING_APPLICATION_NAME}" \
                        -p ${SERVER_PORT}:${SERVER_PORT} ${APP_IMAGE}
                    '''
                }
            }
        }

        stage('Inspect Docker Network') {
            steps {
                script {
                    sh '''
                    docker network inspect ${NETWORK_NAME}
                    '''
                }
            }
        }

        /*stage('Run SQL Script') {
            steps {
                script {
                    sh '''
                    docker exec ${PG_CONTAINER} bash -c 'psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} -f /script.sql'
                    '''
                }
            }
        }*/
        
        stage('Ejecutar comandos SQL') {
            steps {
                script {
                    // Ejecutar comandos SQL dentro del contenedor pg_container
                    sh '''
                        # Conectarse al contenedor y ejecutar comandos SQL
                        docker exec -i ${PG_CONTAINER} psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} <<EOF
                        CREATE DATABASE gestion_eventos WITH ENCODING 'UTF8';

                        -- Tabla de usuarios
                        CREATE TABLE usuarios (
                            id SERIAL PRIMARY KEY,
                            nombre VARCHAR(100) NOT NULL,
                            apellido VARCHAR(100) NOT NULL,
                            email VARCHAR(100) NOT NULL UNIQUE,
                            contrasena VARCHAR(100) NOT NULL,
                            fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                            rol VARCHAR(50) DEFAULT 'usuario'
                        );

                        -- Tabla de categoris de eventos
                        CREATE TABLE categorias (
                            id SERIAL PRIMARY KEY,
                            nombre VARCHAR(100) NOT NULL UNIQUE
                        );

                        -- Tabla de ciudades
                        CREATE TABLE ciudades (
                            id SERIAL PRIMARY KEY,
                            nombre VARCHAR(100) NOT NULL UNIQUE
                        );

                        -- Tabla de eventos
                        CREATE TABLE eventos (
                            id SERIAL PRIMARY KEY,
                            titulo VARCHAR(100) NOT NULL,
                            descripcion TEXT,
                            fecha_inicio TIMESTAMP NOT NULL,
                            fecha_fin TIMESTAMP NOT NULL,
                            ubicacion VARCHAR(255),
                            organizador_id INT NOT NULL REFERENCES usuarios(id),
                            categoria_id INT NOT NULL REFERENCES categorias(id),
                            ciudad_id INT NOT NULL REFERENCES ciudades(id),
                            valor DECIMAL(10, 2),
                            imagen_html TEXT,
                            fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                        );

                        -- Tabla de inscripciones a los eventos
                        CREATE TABLE inscripciones (
                            id SERIAL PRIMARY KEY,
                            usuario_id INT NOT NULL REFERENCES usuarios(id),
                            evento_id INT NOT NULL REFERENCES eventos(id),
                            fecha_inscripcion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                            UNIQUE(usuario_id, evento_id)
                        );

                        -- Insertar datos de prueba en la tabla de usuarios
                        INSERT INTO usuarios (nombre, apellido, email, contrasena, rol)
                        VALUES 
                        ('Admin', 'Admin', 'admin@eventos.com', 'admin123', 'admin'),
                        ('Juan', 'Perez', 'juan.perez@example.com', 'password123', 'ROLE_usuario'),
                        ('Maria', 'Garcia', 'maria.garcia@example.com', 'password123', 'usuario');

                        -- Insertar datos de prueba en la tabla de categorias
                        INSERT INTO categorias (nombre)
                        VALUES 
                        ('Verano Familiar'),
                        ('Programacion'),
                        ('Musica'),
                        ('Deportes');

                        -- Insertar datos de prueba en la tabla de ciudades
                        INSERT INTO ciudades (nombre)
                        VALUES 
                        ('Playa Central'),
                        ('Parque de la Ciudad'),
                        ('Sala de Conferencias'),
                        ('Polideportivo Municipal');

                        -- Insertar datos de prueba en la tabla de eventos
                        INSERT INTO eventos (titulo, descripcion, fecha_inicio, fecha_fin, ubicacion, organizador_id, categoria_id, ciudad_id, valor, imagen_html)
                        VALUES 
                        ('Festival de Verano Familiar', 'Un dia lleno de actividades familiares en la playa.', '2024-08-01 10:00:00', '2024-08-01 18:00:00', 'Playa Central', 1, 1, 1, 0.00, '<img src="festival_verano.jpg" />'),
                        ('Conferencia de Programacion', 'Una conferencia sobre las ultimas tendencias en programacion.', '2024-08-05 09:00:00', '2024-08-05 17:00:00', 'Sala de Conferencias', 1, 2, 3, 0.00, '<img src="conf_programacion.jpg" />');

                        -- Insertar datos de prueba en la tabla de inscripciones
                        INSERT INTO inscripciones (usuario_id, evento_id)
                        VALUES 
                        (2, 1),
                        (3, 2);

                        -- Consultas para verificar los datos insertados en las tablas
                        SELECT * FROM usuarios;
                        SELECT * FROM categorias;
                        SELECT * FROM ciudades;
                        SELECT * FROM eventos;
                        SELECT * FROM inscripciones;

EOF
                    '''
                }
            }
        }
    }

    post {
        /*always {
            echo 'Cleaning up Docker containers and workspace...'
            script {
                sh '''
                docker stop ${PG_CONTAINER} ${APP_CONTAINER}
                docker rm ${PG_CONTAINER} ${APP_CONTAINER}
                docker network rm ${NETWORK_NAME}
                '''
            }
            // Limpieza de archivos temporales
            cleanWs()
        }*/
        always{
            echo 'Slack Notification desde Jenkinsfile'
            slackSend channel: 'jenkins',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}: Job ${env.JOB_NAME} (jenkinsfile) - build ${env.BUILD_NUMBER}\n More Info at: ${env.BUILD_URL}"
        }
    }
}
