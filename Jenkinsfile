node{

   def tomcatWeb = 'C:\\Program Files\\Apache Software Foundation\\Tomcat 10.0\\webapps'
   def tomcatBin = 'C:\\Program Files\\Apache Software Foundation\\Tomcat 10.0\\bin'
   def tomcatStatus = ''
   stage('SCM Checkout'){
     git 'https://github.com/tavleen68/TopgearProject'
   }
   stage('Compile-Package-create-war-file'){
      // Get maven home path
      def mvnHome =  tool name: 'MAVEN_HOME', type: 'maven'   
      bat "${mvnHome}/bin/mvn package"
      }
/*   stage ('Stop Tomcat Server') {
               bat ''' @ECHO OFF
               wmic process list brief | find /i "tomcat" > NUL
               IF ERRORLEVEL 1 (
                    echo  Stopped
               ) ELSE (
               echo running
                  "${tomcatBin}\\shutdown.bat"
                  sleep(time:10,unit:"SECONDS") 
               )
'''
   }*/
   stage('Deploy to Tomcat'){
     bat "copy target\\SampleMavenWebproject.war \"${tomcatWeb}\\SampleMavenWebproject.war\""
   }
      stage ('Start Tomcat Server') {
         sleep(time:5,unit:"SECONDS") 
         bat "${tomcatBin}\\startup.bat"
         sleep(time:100,unit:"SECONDS")
   }
   stage('Build Docker Imager'){
   sh 'docker build -t tavleen/myweb:0.0.1 .'
 }
 
 stage('Push to Docker Hub'){
 
	 withCredentials([string(credentialsId: 'github-pwd', variable: 'dockerHubPwd')]) {
        sh "docker login -u tavleen -p ${dockerHubPwd}"
     }
	 sh 'docker push tavleen/myweb:0.0.1'
 }
 stage('Remove Previous Container'){
	try{
		def dockerRm = 'docker rm -f myweb'
		sshagent(['docker-dev']) {
			sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.17.196 ${dockerRm}"
		}
	}catch(error){
		//  do nothing if there is an exception
	}
 }
 stage('Deploy to Dev Environment'){
   def dockerRun = 'docker run -d -p 8080:8080 --name myweb tavleen/myweb:0.0.1 '
   sshagent(['docker-dev']) {
    sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.17.196 ${dockerRun}"
   }

 }
}
