pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ganeshchandran/javahometech-jenkin-demo.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube') { // You can override the credential to be used
                sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Nexus Artifact Upload') {
            steps {
	       script {
		 pom = readMavenPom file: "pom.xml";
		 filesByGlob = findFiles(glob: "target/*.${pom.packaging}");      
		 echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
		 artifactPath = filesByGlob[0].path;
		 artifactExists = fileExists artifactPath;
		 nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: 'localhost:8081',
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: 'first-pipeline',
                            credentialsId: 'nexus-credentials',
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
	       }
            }
			}
	stage('Docker build and push') {
            steps {
		script {
			docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential')
			{
			dockerImage = docker.build "registry.hub.docker.com/ganeshchandran/jenkin-pipeline:$BUILD_NUMBER"
			dockerImage.push()
			dockerImage.push('latest')
			}
			sh "docker rmi registry.hub.docker.com/ganeshchandran/jenkin-pipeline:$BUILD_NUMBER"
			sh "docker rmi registry.hub.docker.com/ganeshchandran/jenkin-pipeline"
		}
            }
        }
	    
	stage('Deploy to GKE') {
            steps {
                step([
                $class: 'KubernetesEngineBuilder',
                projectId: 'inlaid-micron-268506',
                clusterName: 'cluster-1',
                location: 'us-central1-c',
                manifestPattern: 'jenkins-deployment.yaml',
                credentialsId: 'inlaid-micron-268506',
                verifyDeployments: true])
            }
        }
	    
	    
	
        //stage('Email Notification') {
        //    steps {
        //        mail bcc: '', body: 'Jenkins Sample Email', cc: '', from: '', replyTo: '', subject: 'Jenkins Build Success', to: 'ganeshchandran@live.in'
        //    }
        //}
 }
 }
