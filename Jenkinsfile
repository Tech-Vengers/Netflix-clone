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
    tools {
        nodejs 'nodejs-16'
    }
    steps {
        sh '''
          node -v
          npm -v
          npm install
        '''
            }
        }

      stage('OWASP FS Scan') {
    environment {
        NVD_API_KEY = credentials('NVD_API_KEY')
    }
    steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
            dependencyCheck(
                odcInstallation: 'DP-Check',
                additionalArguments: [
                    '--scan .',
                    '--disableYarnAudit',
                    '--disableNodeAudit',
                    '--nvdApiKey', env.NVD_API_KEY,
                    '--nvdApiDelay', '16000',
                    '--noupdate',
                    '--failOnCVSS', '11'
                ].join(' ')
            )
        }
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
                        sh 'docker tag netflix techvengers7788/netflix:latest'
                        sh 'docker push techvengers7788/netflix:latest'
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh 'trivy image techvengers7788/netflix:latest > trivyimage.txt'
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                  docker rm -f netflix || true
                  docker run -d --name netflix \
                    -p 8081:80 \
                    -e TMDB_API_KEY=$TMDB_API_KEY \
                    techvengers7788/netflix:latest
                '''
            }
        }
    }
}
