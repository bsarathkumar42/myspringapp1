pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "Maven 3.6.3"
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "15.206.84.125:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "sarath142"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_cred"
		registry = "gustavoapolinario/docker-test"
        registryCredential = 'dockerhub'
        dockerImage = ''
		project = 'my-project'
		appName = 'my-first-microservice'
		serviceName = "${appName}-backend"  
		imageVersion = 'development'
		namespace = 'development'
		namespace1 = 'production'
		imageTag = "gcr.io/${project}/${appName}:${imageVersion}.${env.BUILD_NUMBER}"
		  
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    
                    git credentialsId: 'Github_credentials', url: 'https://github.com/bsarathkumar42/myspringapp.git'
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
		stage('Sonarqube') {
			environment {
				scannerHome = tool 'SonarQubeScanner'
			}    steps {
				withSonarQubeEnv('sonarqube') {
					sh "${scannerHome}/bin/sonar-scanner"
				}        timeout(time: 10, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
       		
		stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    nexusBySarath = findFiles(sarath: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${nexusBySarath[0].name} ${nexusBySarath[0].path} ${nexusBySarath[0].directory} ${nexusBySarath[0].length} ${nexusBySarath[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = nexusBySarath[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
		

		stage('Building image') {
			steps{
				script {
					dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
	
	
		stage('Deploy Image') {
			steps{
				script {
					docker.withRegistry( '', registryCredential ) {
					dockerImage.push()
          }
        }
      }
    }
	
		stage('Remove Unused docker image') {
			steps{
				sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

	
		stage('Deploy Application') {
			switch (namespace) {
              
				case "development":
                   
					sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
					sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/development/*.yaml")
					sh("kubectl --namespace=${namespace} apply -f k8s/development/deployment.yaml")
					sh("kubectl --namespace=${namespace} apply -f k8s/development/service.yaml")
                    sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                   break
           
                case "production":
        
                   sh("kubectl get ns ${namespace1} || kubectl create ns ${namespace1}")
                   sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/production/*.yaml")
                   sh("kubectl --namespace=${namespace1} apply -f k8s/production/deployment.yaml")
                   sh("kubectl --namespace=${namespace1} apply -f k8s/production/service.yaml")
                   sh("echo http://`kubectl --namespace=${namespace1} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                  break
       
              //default:
                   //sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
                   //sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/development/*.yaml")
                   //sh("kubectl --namespace=${namespace} apply -f k8s/development/deployment.yaml")
                   //sh("kubectl --namespace=${namespace} apply -f k8s/development/service.yaml")
                   //sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                   //break
  }

}	
	
