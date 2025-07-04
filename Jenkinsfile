pipeline {
    agent none

    environment {
        BRANCH = "${env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'develop'}"
    }

    stages {
        stage('Get Code & Config') {
            agent { label 'agente-secundario' }
            steps {
                sh 'echo Ejecutando como: $(whoami) en $(hostname)'
                checkout scm

                script {
                    def configBranch = BRANCH.contains('master') ? 'production' : 'staging'
                    echo " Descargando configuración desde rama: ${configBranch}"
                    sh """
                        rm -f samconfig.toml
                        git clone --branch ${configBranch} https://github.com/andresh2004/todo-list-aws-config.git config-repo
                        cp config-repo/samconfig.toml .
                        rm -rf config-repo
                        echo ' samconfig.toml descargado correctamente:'
                        cat samconfig.toml
                    """
                }
            }
        }

        stage('Análisis Estático') {
            when {
                expression { BRANCH.contains('develop') }
            }
            agent { label 'agente-secundario' }
            steps {
                sh 'echo Ejecutando análisis estático de código...'
                // Aquí podrías integrar pylint, flake8, etc.
            }
        }

        stage('Despliegue con SAM') {
            when {
                expression { BRANCH.contains('master') }
            }
            agent { label 'agente-secundario' }
            steps {
                sh 'echo  Desplegando aplicación en producción con SAM...'
                sh 'sam deploy --config-env default --config-file samconfig.toml'
            }
        }

        stage('Pruebas API REST') {
            when {
                expression { BRANCH.contains('develop') }
            }
            agent { label 'agente-secundario' }
            steps {
                sh 'echo  Ejecutando pruebas sobre la API REST...'
                //  Usar curl, Postman CLI, etc.
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado para rama: ${BRANCH}"
        }
    }
}

