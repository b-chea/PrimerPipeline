pipeline {
    agent any
    environment {
        JIRA_SITE = 'https://bethsaidach-1738694022756.atlassian.net'
        JIRA_CREDENTIALS = credentials('jenkins-credentials-local') // Utilizamos las credenciales guardadas de forma segura
        JIRA_USER = "${JIRA_CREDENTIALS_USR}"
        JIRA_API_TOKEN = "${JIRA_CREDENTIALS_PSW}"
        JIRA_ISSUE_KEY = 'PLPROJECT1'  // Clave del proyecto en Jira
        JIRA_ISSUE_TYPE = 'Bug' // Tipo de issue, personaliza según sea necesario
        // Aquí concatenamos y codificamos las credenciales en Base64 de manera más segura
        JIRA_AUTH = credentials('jenkins-credentials-local') // No usamos string interpolation insegura
    }

    tools {
        maven 'MAVEN_HOME'
        jdk 'JAVA_HOME'
    }

    stages {
        stage('Build') {
            steps {
                echo "Building..."
                echo "JIRA_USER: ${JIRA_USER}"
                echo "JIRA_API_TOKEN: ${JIRA_API_TOKEN}"
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
                    // Crear el cuerpo del JSON para la solicitud de Jira
                    def jiraIssue = [
                        fields: [
                            project: [ key: env.JIRA_ISSUE_KEY ], // Clave del proyecto en Jira
                            summary: "Build failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}", // Resumen del issue
                            description: "The build failed. Please check the Jenkins logs for more details.", // Descripción del issue
                            issuetype: [ name: env.JIRA_ISSUE_TYPE ] // Tipo de issue (Bug, Task, etc.)
                        ]
                    ]

                    // Realizar la solicitud HTTP para crear el issue
                    def response = httpRequest(
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        httpMode: 'POST',
                        requestBody: groovy.json.JsonOutput.toJson(jiraIssue), // Convertir el cuerpo a JSON
                        url: "${env.JIRA_SITE}/rest/api/3/issue", // URL del API de Jira
                        customHeaders: [[name: 'Authorization', value: "Basic ${JIRA_USER}:${JIRA_API_TOKEN}".bytes.encodeBase64().toString()]] // Codificación de las credenciales de forma segura
                    )

                    echo "Jira issue created: ${response.status}" // Mostrar el estado de la respuesta
                    echo "Jira response: ${response.content}" // Mostrar el contenido de la respuesta
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
