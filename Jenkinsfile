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
        script {
            def msg = ""
            
            if (env.BRANCH_NAME == 'development') {
                msg = "✅ DEV SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nBranch: ${env.BRANCH_NAME}\nFull CI/CD executed\n${env.BUILD_URL}"
            } 
            else if (env.BRANCH_NAME == 'main') {
                msg = "✅ MAIN SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nBranch: ${env.BRANCH_NAME}\nBuild + Sonar only\n${env.BUILD_URL}"
            } 
            else {
                msg = "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nBranch: ${env.BRANCH_NAME}\n${env.BUILD_URL}"
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
            def msg = "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nBranch: ${env.BRANCH_NAME}\n${env.BUILD_URL}"

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

