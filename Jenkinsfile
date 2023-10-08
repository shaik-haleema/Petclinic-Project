pipeline {
    agent any
    
    /* Particular Version Recommended to Use in Project */
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }

    stages {
        
        /* Clone the code from https://github.com/chinnayyachintha/Ekart/tree/main*/
        stage('SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/chinnayyachintha/Petclinic-Project.git']])
            }
        }
        
        /* Compile source code and skip Test cases why? because in this project we will get error */
        stage('COMPILE') {
            steps {
                sh "mvn clean compile "
            }
        }
    }
}
