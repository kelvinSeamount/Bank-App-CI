pipeline {
    agent any
    tools{
        maven "maven3"
    }
    environment {
        SCANNER_HOME = tool "sonar-scanner"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/kelvinSeamount/Bank-App-CI.git'
            }
        }
        
         stage('Compile') {
            steps {
                dir('Bank-CI-main') {
                    sh "mvn compile"
                }
            }
        }
        
         stage('Testing') {
            steps {
                dir('Bank-CI-main') {
                    sh "mvn test"
                }
            }
        }
        
         stage('Trivy Fs Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
         stage('Sonar Analysis') {
            steps {
                dir('Bank-CI-main') {
                    withSonarQubeEnv('sonar') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=gcbank -Dsonar.projectName=gcbank \
                               -Dsonar.java.binaries=target '''
                    }
                }
                echo "SonarQube analysis completed. Check results at: http://3.127.39.213:9000/dashboard?id=gcbank"
            }
        }
        
          // Quality Gate Check - Skipped for now due to timeout issues
          // You can view the quality gate results directly in SonarQube dashboard
          // stage('Quality Gate Check') {
          //   steps {
          //       timeout(time: 5, unit: 'MINUTES') {
          //            waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
          //       }
          //   }
          // }
        
          stage('Build') {
            steps {
                dir('Bank-CI-main') {
                    sh "mvn package"
                }
            }
        }
        
          stage('Publish to Nexus') {
            steps {
                dir('Bank-CI-main') {
                    withMaven(globalMavenSettingsConfig: 'mekadevops', maven: 'maven3', traceability: true) {
                        sh "mvn deploy -DskipTests"
                    }
                }
            }
        }
        
          stage('Docker Image Build & Tag') {
            steps {
                dir('Bank-CI-main'){
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker build -t castromeka/gcbank:${IMAGE_TAG} ."
                        }
                    }
                }
            }
        }
        
        stage('Scan Docker Image') {
            steps {
                sh "trivy image --format table -o image-report.html castromeka/gcbank:${IMAGE_TAG}"
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push castromeka/gcbank:${IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Complete') {
            steps {
                echo "======================================"
                echo "Pipeline completed successfully!"
                echo "Docker Image: castromeka/gcbank:${IMAGE_TAG}"
                echo "SonarQube Dashboard: http://3.127.39.213:9000/dashboard?id=gcbank"
                echo "======================================"
            }
        }
    }
}