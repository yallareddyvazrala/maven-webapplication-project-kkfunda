pipeline {
    agent any 

    triggers {
        githubPush()
    }

    tools {
        maven 'maven-3.9.10'
    }

    environment {
        SLACK_CHANNEL = 'tataaig-app'
    }

    stages {

        stage('Git Clone') {
            steps {
                git 'https://github.com/yallareddyvazrala/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Build') {
            when {
                expression {
                    env.BRANCH_NAME == 'development' || env.BRANCH_NAME == 'main'
                }
            }
            steps {
                sh "mvn clean package"
            }
        }

        stage('SonarQube') {
            when {
                expression {
                    env.BRANCH_NAME == 'development' || env.BRANCH_NAME == 'main'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.token=${SONAR_TOKEN}"
                }
            }
        }

        stage('Artifactory') {
            when {
                expression {
                    env.BRANCH_NAME == 'development'
                }
            }
            steps {
                sh "mvn deploy"
            }
        }

        stage('Deploy WAR File') {
            when {
                expression {
                    env.BRANCH_NAME == 'development'
                }
            }
            steps {
                sshagent(['ec-user']) {
                    sh """
                    scp -o StrictHostKeyChecking=no \
                    target/maven-web-application.war \
                    ec2-user@65.0.173.83:/opt/apache-tomcat-9.0.117/webapps/
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: "good",
                message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.BRANCH_NAME}) - ${env.BUILD_URL}"
            )
        }

        failure {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: "danger",
                message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.BRANCH_NAME}) - ${env.BUILD_URL}"
            )
        }

        always {
            echo "Pipeline completed for branch: ${env.BRANCH_NAME}"
        }
    }
}

