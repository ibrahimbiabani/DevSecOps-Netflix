pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ibrahimbiabani/DevSecOps-Netflix.git'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
-Dsonar.projectName=Netflix \
-Dsonar.projectKey=Netflix'''
                }
            }
        }

        stage('quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('app') {
                    sh "npm install"
                }
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        // 1) TMDB API KEY GOES HERE
                        sh "docker build --build-arg TMDB_V3_API_KEY=75729bd5a677c0750d99b578d52f66c1 -t netflix ."

                        sh "docker tag netflix ibrahimbiabani/netflix:latest"
                        sh "docker push ibrahimbiabani/netflix:latest"
                    }
                }
            }
        }

        stage('TRIVY IMAGE SCAN') {
            steps {
                sh "trivy image ibrahimbiabani/netflix:latest > trivyimage.txt"
            }
        }

        stage('Deploy to container') {
            steps {
                sh "docker run -d --name netflix -p 8081:80 ibrahimbiabani/netflix:latest"
            }
        }
    }
}
