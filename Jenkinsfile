pipeline {
    agent any

    environment {
        PATH = "$HOME/.cargo/bin:$PATH"
        SCANNER_HOME = tool 'sonar-scanner'
        // URLs de tus otros repositorios
        REPO_API_URL = 'file:///local_repos/ecolibrary-api'
        REPO_WEB_URL = 'file:///local_repos/ecolibrary-web'
    }

    stages {
        stage('Checkout Sub-Repositories') {
            steps {
                script {
                    // Limpiamos workspace anterior para evitar mezclas
                    cleanWs()
                    
                    // 1. Clonar el Repo BASE (donde estamos) ya lo hace Jenkins automático
                    // pero necesitamos clonar los hijos en carpetas específicas.

                    // Descargar API
                    dir('api') {
                        echo "Clonando API desde rama: ${env.BRANCH_NAME}"
                        // Intentamos bajar la misma rama (feature/x), si falla, bajamos main/dev
                        try {
                            git branch: env.BRANCH_NAME, url: env.REPO_API_URL
                        } catch (Exception e) {
                            echo "Rama ${env.BRANCH_NAME} no existe en API, usando 'main'"
                            git branch: 'main', url: env.REPO_API_URL
                        }
                    }

                    // Descargar WEB
                    dir('web') {
                        echo "Clonando WEB desde rama: ${env.BRANCH_NAME}"
                        try {
                            git branch: env.BRANCH_NAME, url: env.REPO_WEB_URL
                        } catch (Exception e) {
                            echo "Rama ${env.BRANCH_NAME} no existe en WEB, usando 'main'"
                            git branch: 'main', url: env.REPO_WEB_URL
                        }
                    }
                }
            }
        }

        stage('Setup & Test') {
            steps {
                script {
                    sh 'curl -LsSf https://astral.sh/uv/install.sh | sh'
                    
                    // Tests API
                    dir('api') {
                        sh '~/.local/bin/uv sync'
                        sh '~/.local/bin/uv run pytest --cov=. --cov-report=xml'
                    }
                    // Tests WEB
                    dir('web') {
                        sh '~/.local/bin/uv sync'
                        sh '~/.local/bin/uv run pytest --cov=. --cov-report=xml'
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    dir('api') { sh "${SCANNER_HOME}/bin/sonar-scanner" }
                    dir('web') { sh "${SCANNER_HOME}/bin/sonar-scanner" }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

stage('Build & Deploy') {
            steps {
                script {
                    // Lógica de Ramas Actualizada
                    // AHORA: Si es 'dev' O es una 'feature/...', va al entorno de desarrollo
                    if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME ==~ /feature\/.*/) {
                        echo "--- DESPLIEGUE ENTORNO DE DESARROLLO (Dev) ---"
                        echo "Rama: ${env.BRANCH_NAME}"
                        // Despliega en puertos 8001 (API) y 8002 (Web)
                        sh 'docker-compose up -d --build'
                    } 
                    // Si es 'main' o 'release/...', va a producción
                    else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME ==~ /release\/.*/) {
                        echo "--- DESPLIEGUE ENTORNO DE PRODUCCIÓN (Prod) ---"
                        echo "Rama: ${env.BRANCH_NAME}"
                        // Despliega en puertos 9001 (API) y 9002 (Web)
                        sh 'docker-compose -f docker-compose.yml -f docker-compose.prod.yml -p prod up -d --build'
                    }
                    else {
                        echo "Rama ${env.BRANCH_NAME} solo pasó pruebas (CI), no se despliega."
                    }
                }
            }
        }
    }
}