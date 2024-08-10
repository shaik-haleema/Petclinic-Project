pipeline {
    agent any
    
    environment {
        // Define environment variables as needed
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKER_REPO = 'docker.io/chinnayya339'
        DOCKER_PASS = credentials('docker-pass')
        APP_NAME = 'jpetstore'
        DOCKER_IMAGE = "${DOCKER_REPO}/${APP_NAME}"
        DOCKER_TAG = "${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
    }

    stages{
        
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        
        stage('checkout'){
            steps{
                git 'https://github.com/chinnayyachintha/jpetstore-6.git'
            }
        }
        stage('Trivy Scan - Code') {
            steps {
                script {
                    sh "trivy fs --exit-code 0 --severity HIGH,CRITICAL ."
                }
            }
        }
        stage('build'){
            steps{
                sh 'mvn -o package'
                // sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                // scan your dependencies in your project
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Login', toolName: 'docker'){
                        // Build the Docker image
                        sh "docker build -t ${DOCKER_TAG} ."
                    }
                }
            }
        } 
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Login', toolName: 'docker'){
                        // Push the Docker image to the repository
                        sh "docker push ${DOCKER_TAG}" 
                     }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh " trivy image ${DOCKER_TAG} "
            }
        }
        
        stage('Run Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Login', toolName: 'docker'){
                        // Run the Docker container from the previously built and pushed image
                        sh "docker-compose -f  docker-compose.yaml up -d"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up by removing the local Docker image
            script {
                sh "docker rmi -f ${DOCKER_TAG}"
            }
       }
    }
}
