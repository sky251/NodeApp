pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS' // Assumes NodeJS is configured in Jenkins tools
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
        SONARQUBE_SERVER = 'SonarQube'  // Name configured for SonarQube in Jenkins
        NEXUS_URL = 'localhost:8081' // Nexus URL
        NEXUS_REPO = 'http://localhost:8081/repository/hello_world/'
        NEXUS_CREDENTIALS_ID = '8cdbbc8b-9d5c-3ef9-af6f-8d055ac07375' // Jenkins credentials ID for Nexus
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'  // Assumes build command is defined in package.json
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner -Dsonar.projectKey=my-nodejs-project'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }

        stage('Deploy to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [
                    [artifactId: 'my-app', classifier: '', file: 'dist/my-app.zip', type: 'zip']
                ],
                credentialsId: "${NEXUS_CREDENTIALS_ID}",
                groupId: 'com.example',
                nexusUrl: "${NEXUS_URL}",
                repository: "${NEXUS_REPO}",
                version: '1.0.0'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
