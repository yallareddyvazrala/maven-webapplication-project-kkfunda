
//qa jenkins pipeline
pipeline
{
	
   agent any
   tools
   {
      maven "maven-3.9.14"
   }
   stages
   {
           stage('git checkout')
           {
              steps
              {
                 
                 git branch: 'qa', url: 'https://github.com/kkdevopsb8/maven-webapplication-project-kkfunda.git'
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
--upload-file /var/lib/jenkins/workspace/MBPL-JIO-OM_qa/target/maven-web-application.war \
"http://13.232.26.179:8080/manager/text/deploy?path=/maven-web-application&update=true"
          
        """
              }
           }
           stage('airtel-uat')
           {
              steps
              {
                 build job: 'airtel-uat'  //This down stream job
              }
           }

   }  //stages ending
} // pipeline ending

