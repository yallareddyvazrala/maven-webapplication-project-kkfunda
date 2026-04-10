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
                git branch: "${env.BRANCH_NAME}",
                    url: 'https://github.com/yallareddyvazrala/maven-webapplication-project-kkfunda.git'
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
            script {
                def msg = ""

                if (env.BRANCH_NAME == 'development') {
                    msg = """✅ DEV SUCCESS
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
Flow: Full CI/CD
URL: ${env.BUILD_URL}"""
                } 
                else if (env.BRANCH_NAME == 'main') {
                    msg = """✅ MAIN SUCCESS
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
Flow: Build + Sonar
URL: ${env.BUILD_URL}"""
                } 
                else {
                    msg = """✅ SUCCESS
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
URL: ${env.BUILD_URL}"""
                }

                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "good",
                    message: msg
                )
            }
        }

        failure {
            script {
                def msg = """❌ FAILED
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
URL: ${env.BUILD_URL}"""

                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "danger",
                    message: msg
                )
            }
        }

        always {
            echo "Pipeline completed for branch: ${env.BRANCH_NAME}"
        }
    }
}
