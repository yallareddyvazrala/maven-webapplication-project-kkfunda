
//dev jenkins pipeline
pipeline
{
    
   agent any
   tools
   {
      maven "maven-3.9.7"
   }
   stages
   {
           stage('git checkout')
           {
              steps
              {
                 
                 git branch: 'dev', url: 'https://github.com/kkdevopsb7/maven-webapplication-project-kkfunda.git'
              }
           }
           stage('compile')
           {
              steps
              {
                 sh "mvn compile"
              }
           }
           stage('Build')
           {
             steps
             {
               sh "mvn clean package"
             }
           }
         stage('SQ REPORT')
           {
             steps
             {
                sh "mvn sonar:sonar"
             }
           }   
           stage('Deploy to nexus')
           {
              steps
              {
                sh "mvn clean deploy"
              }
           }
           stage('Deploy to tomcat')
           {
              steps
              {
                 sh """

      curl -u kk:password \
--upload-file /var/lib/jenkins/workspace/jio-Declarative-PL-dev/target/maven-web-application.war \
"http://13.232.234.199:8080/manager/text/deploy?path=/maven-web-application&update=true"
          
        """
              }
           }
           stage('airtel-qa')
           {
              steps
              {
                 build job: 'airtel-qa'  //This down stream job
              }
           }

   }  //stages ending

   } //pipeline ending





