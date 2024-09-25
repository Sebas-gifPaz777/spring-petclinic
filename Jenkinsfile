pipeline {
    agent none  // No asignar agente a nivel de pipeline global

    stages {
        stage('Prepare') {
            agent { label 'build-node' }  // Agente solo para la etapa de preparación
            steps {
                script {
                    // Descargar el código desde el repositorio
                    git 'https://github.com/Sebas-gifPaz777/spring-petclinic.git'
                    stash name: 'source-code'
                }
            }
        }

        stage('Build and Test') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'build-node' }  // Agente solo para esta sub-etapa
                    steps {
                        unstash 'source-code'
                        // Ejecutar pruebas unitarias con Gradle
                        sh './gradlew test'
                        stash name: 'test-results'
                    }
                }

                stage('Generate Javadoc') {
                    agent { label 'build-node' }  // Agente solo para esta sub-etapa
                    steps {
                        unstash 'source-code'
                        // Generar Javadoc en paralelo
                        sh './gradlew javadoc'
                        stash name: 'javadoc'
                    }
                }
            }
        }

        stage('Deploy and Integration Tests') {
            agent { label 'deploy-node' }  // Agente para la etapa de despliegue
            steps {
                unstash 'source-code'
                // Construir la imagen Docker y ejecutar el contenedor
                sh 'docker build -t petclinic-app .'
                sh 'docker run -d -p 8080:8080 --name petclinic-app petclinic-app'
                
                // Ejecutar pruebas de integración
                sh './gradlew integrationTest'

                // Detener y limpiar el contenedor
                sh 'docker stop petclinic-app'
                sh 'docker rm petclinic-app'
            }
        }
    }

    post {
        always {
            // Publicar resultados
            junit 'build/test-results/**/*.xml'
            archiveArtifacts artifacts: 'build/docs/javadoc/**/*', allowEmptyArchive: true
        }
    }
}
