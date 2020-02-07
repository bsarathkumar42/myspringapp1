pipeline {
    agent any
        tools {
            maven "maven-3.6.3"
       /* choice(
            name: 'sarath',
            choices: ['DEV', 'QA', 'UAT', 'PROD'],
            description: 'Passing the parameters')
         */ 
    }
    
    environment {

        //NEXUS_VERSION = "nexus3"
        //NEXUS_PROTOCOL = "http"
        //NEXUS_URL = "13.233.121.236:8081"
        //NEXUS_REPOSITORY = "sarath142"
        //NEXUS_CREDENTIAL_ID = "nexus_cred"
		registry = "bsarathkumar42/myspringapp1"
        registryCredential = 'dockerhub'
        dockerImage = ''
		project = 'myspringapp1'
		appName = 'myspringapp1'
		serviceName = "${appName}-backend"  
		imageVersion = 'development'
		namespace = 'development'
		namespace1 = 'production'
		imageTag = "gcr.io/${project}/${appName}:${imageVersion}.${env.BUILD_NUMBER}"
		//scannerHome = tool 'SonarQubeScanner'
		  
    }
   /* stages {
    stage('Environment') {
      steps {
        echo " The environment is ${params.Env}"
      }
    }
  }*/
    stages {
        stage("clone code") {
            steps {
                script {
                    
                     git credentialsId: 'Github_credentials', url: 'https://github.com/bsarathkumar42/myspringapp1.git'
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                             
                    sh "mvn clean package -DskipTests=true"
                }
            }
        }
	/*	stage('Sonarqube') {
			steps {
				withSonarQubeEnv('sonar') {
						sh " mvn sonar:sonar -Dsonar.host.url=http://52.66.250.13:9000 -Dsonar.login=11b8a2aced56d047f183092eb7268c9ae67c73e4 "
          			    timeout(time: 02, unit: 'MINUTES') {
							//waitForQualityGate abortPipeline: true
            }
        }
    }
}
		  
       		
		stage("publish to nexus") {
            steps {
                script {
                    
                    sh  "mvn deploy"

         
        }
    }
}
	*/	
	
}
}
