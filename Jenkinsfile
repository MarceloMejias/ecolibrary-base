pipeline {
    agent any

    environment {
        // Aseguramos que uv pueda instalarse y ejecutarse
        PATH = "$HOME/.cargo/bin:$PATH"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Setup Environment') {
            steps {
                // Instalar uv en el agente de Jenkins (que es un contenedor)
                sh 'curl -LsSf https://astral.sh/uv/install.sh | sh'
            }
        }

        stage('Test & Coverage') {
            steps {
                script {
                    // 1. Tests API
                    dir('api') {
                        sh '~/.cargo/bin/uv sync'
                        // Genera reporte XML para SonarQube
                        sh '~/.cargo/bin/uv run pytest --cov=. --cov-report=xml'
                    }
                    // 2. Tests WEB
                    dir('web') {
                        sh '~/.cargo/bin/uv sync'
                        sh '~/.cargo/bin/uv run pytest --cov=. --cov-report=xml'
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    // Analizamos API
                    dir('api') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                    // Analizamos Web
                    dir('web') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    // REQUISITO CRÍTICO: Detener si falla 
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    // Lógica de Ramas 
                    if (env.BRANCH_NAME ==~ /feature\/.*/) {
                        echo "Desplegando entorno de DESARROLLO (Dev)"
                        // Usa el docker-compose normal (Puertos 8001/8002)
                        sh 'docker-compose up -d --build'
                    } 
                    else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME ==~ /release\/.*/) {
                        echo "Desplegando entorno de PRODUCCIÓN (Prod)"
                        // Usa el override de producción (Puertos 9001/9002)
                        // -p prod: Crea contenedores con prefijo 'prod' para que coexistan
                        sh 'docker-compose -f docker-compose.yml -f docker-compose.prod.yml -p prod up -d --build'
                    }
                    else {
                        echo "Rama no configurada para despliegue automático"
                    }
                }
            }
        }
    }
}