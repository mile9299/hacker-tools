pipeline {
    agent any
    
    environment {
        HACKING_TOOLS_REPO = 'https://github.com/mile9299/hacker-tools.git'
        DOCKER_PORT = 3000 // Default Docker port
        SPECTRAL_DSN = credentials('SPECTRAL_DSN')
    }
    // Added
    tools {
        nodejs 'NodeJS 18.0.0'
    }
/// Added
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: HACKING_TOOLS_REPO]]])
                }
            }
        }
        stage('Test with Snyk') {
            steps {
                script {
                    snykSecurity failOnIssues: false, severity: 'critical', snykInstallation: 'snyk-manual', snykTokenId: 'SNYK'
                }
           }
        }
       // stage('install Spectral') {
         //     steps {
           //     sh "curl -L 'https://get.spectralops.io/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
             // }
        //}
       // stage('scan for issues') {
         //     steps {
           //     sh "$HOME/.spectral/spectral scan --ok --engines secrets,iac,oss --include-tags base,audit,iac"
             // }
       // }
         stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    def dockerImage = docker.build('hackingtools', '-f Dockerfile .')
                    echo 'Docker image built successfully!'
                }
            }
        }
        stage('Falcon Cloud Security') {
              steps {
                withCredentials([usernameColonPassword(credentialsId: 'CRWD', variable: '')]) {
                crowdStrikeSecurity imageName: 'hackingtools', imageTag: 'latest', enforce: true, timeout: 60
            }
          }
        }
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying application...'
                    sh 'docker stop hackingtools || true'
                    sh 'docker rm spooky || true'
                    def containerId = sh(script: "docker run -d -P --name hackingtools hackingtools", returnStdout: true).trim()
                    def dockerHostPort = sh(script: "docker port ${containerId} ${DOCKER_PORT} | cut -d ':' -f 2", returnStdout: true).trim()
                    echo "Hacking Tools are running on http://localhost:${dockerHostPort}"
                }
            }
        }
        
    }

    post {
        success {
            echo 'Build, test, and deployment successful!'
        }
        failure {
            echo 'Build, test, or deployment failed!'
        }
    }
}
