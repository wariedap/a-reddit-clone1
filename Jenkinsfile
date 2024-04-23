pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "java-registration-app"
        RELEASE = "1.0.0"
        DOCKER_USER = "osanyap"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
       JENKINS_API_TOKEN = '1105bf46774471ace702dcc34b938f8977'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/wariedap/a-reddit-clone1.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI -Dsonar.projectKey=Reddit.Clone.CI"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
         }
         stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                     }
                 }
             }
         }
     /*   stage("Trivy Image Scan") {
    steps {
        script {
            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy osanyap/java-registration-app:latest --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
        }
    }
}
*/

        stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }
	    stage("Trigger CD Pipeline") {
            steps {
                script {
                sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://ec2-100-26-177-139.compute-1.amazonaws.com:8080/job/Reddit-Clone-CD/buildWithParameters?token=gitops-token'"

                }
            }
         }
     }
}
