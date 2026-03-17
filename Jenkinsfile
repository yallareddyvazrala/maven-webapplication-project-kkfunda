//This is scripted-way pipeline 
node
{

  def mavenHome = tool name: "maven-3.9.14"

  stage('checkout')
  {
    git branch: 'dev', url: 'https://github.com/kkdevopsb8/maven-webapplication-project-kkfunda.git'
  }

  stage('compile')
  {
    sh "${mavenHome}/bin/mvn compile"
  }

  stage('BUILD')
  {
    sh "${mavenHome}/bin/mvn clean package"
  }

 /* stage('SQ REPORT')
  {
    sh "${mavenHome}/bin/mvn sonar:sonar"
  } */

  stage('Deploy to Nexus')
  {
    sh "${mavenHome}/bin/mvn deploy"
  }



  stage('Deploy to Tomcat') {
    if (currentBuild.currentResult == 'SUCCESS') {
        withCredentials([usernamePassword(
            credentialsId: 'tomcat-cread-1',
            usernameVariable: 'TOMCAT_USER',
            passwordVariable: 'TOMCAT_PASS'
        )]) {
            sh '''
            curl -u "$TOMCAT_USER:$TOMCAT_PASS" \
            --upload-file /var/lib/jenkins/workspace/jio-dev-pipeline/target/maven-web-application.war \
            "http://43.205.230.133:8080/manager/text/deploy?path=/maven-web-application&update=true"
            '''
        }
    } else {
        echo "Build failed or unstable. Skipping deployment."
    }
  }

} //node ending
