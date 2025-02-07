pipeline {
    agent any
    environment {
        JIRA_SITE = 'https://bethsaidach-1738694022756.atlassian.net'
        JIRA_CREDENTIALS = credentials('jenkins-credentials-local') // Utilizamos las credenciales guardadas de forma segura
        //JIRA_USER = "${JIRA_CREDENTIALS_USR}"
        JIRA_API_TOKEN = "${JIRA_CREDENTIALS_PSW}"
        JIRA_ISSUE_KEY = 'PLPROJECT1'  // Clave del proyecto en Jira
        JIRA_ISSUE_TYPE = 'Bug' // Tipo de issue, personaliza según sea necesario
        // Aquí concatenamos y codificamos las credenciales en Base64 de manera más segura
        JIRA_AUTH = credentials('jenkins-credentials-local') // No usamos string interpolation insegura


        JIRA_USER = "bethsaidach@outlook.com"
        JIRA_URL = "https://bethsaidach-1738694022756.atlassian.net/rest/api/3/issue"
    }

    tools {
        maven 'MAVEN_HOME'
        jdk 'JAVA_HOME'
    }

    stages {
        stage('Build') {
            steps {
                echo "Building..."
                // Descomenta si quieres añadir un comentario en Jira al inicio de la build
                // jiraAddComment comment: 'Build iniciada en Jenkins', idOrKey: 'PLPROJECT1', site: 'bethsaidach-1738694022756.atlassian.net'
            }
        }

        stage('Compilar proyecto') {
            steps {
                bat 'mvn compile' // Compila el proyecto
            }
        }

        stage('Ejecutar Pruebas') {
            steps {
                bat 'mvn test' // Ejecuta las pruebas
            }
        }

        stage('Create Jira Issue') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'JIRA_API_TOKEN', variable: 'JIRA_AUTH_PSW')]) {
                        def authHeader = "Basic " + "${JIRA_USER}:${JIRA_AUTH_PSW}".bytes.encodeBase64().toString()
                        def jsonPayload = """{
                            "fields": {
                                "project": { "key": "TEST" },
                                "summary": "Prueba desde Jenkins",
                                "description": "Creando un issue desde Jenkins",
                                "issuetype": { "name": "Bug" }
                            }
                        }"""

                        def response = sh(
                            script: """curl -X POST -H "Authorization: ${authHeader}" -H "Content-Type: application/json" -H "Accept: application/json" --data '${jsonPayload}' ${JIRA_URL}""",
                            returnStdout: true
                        ).trim()

                        echo "JIRA Response: ${response}"
                    }
                }
            }
        }

    }

    post {
        success {
            echo 'Successfully created Jira issue!' // Mensaje de éxito
        }
        failure {
            echo 'Build failed and Jira issue was not created!' // Mensaje de fallo
        }
    }
}
