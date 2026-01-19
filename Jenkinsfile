pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TMDB_API_KEY = credentials('TMDB_API_KEY')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Tech-Vengers/Netflix-clone.git'
            }
        }

        stage('SonarQube Analysis') {
    tools {
        nodejs 'nodejs-16'
    }
    steps {
        withSonarQubeEnv('sonar-server') {
            sh '''
              node -v
              $SCANNER_HOME/bin/sonar-scanner \
              -Dsonar.projectName=Netflix \
              -Dsonar.projectKey=Netflix
            '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqubetoken'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                                odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build --build-arg TMDB_API_KEY=$TMDB_API_KEY -t netflix .'
                        sh 'docker tag netflix Techvengers7788/netflix:latest'
                        sh 'docker push Techvengers7788/netflix:latest'
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh 'trivy image Techvengers7788/netflix:latest > trivyimage.txt'
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                  docker rm -f netflix || true
                  docker run -d --name netflix \
                    -p 8081:80 \
                    -e TMDB_API_KEY=$TMDB_API_KEY \
                    Techvengers7788/netflix:latest
                '''
            }
        }
    }
}
