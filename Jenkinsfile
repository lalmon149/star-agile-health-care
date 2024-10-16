pipeline { 
    agent any
    tools {
        maven 'M2_HOME'
    }     
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'This stage is to clone the repo from GitHub'
                git branch: 'master', url: 'https://github.com/lalmon149/star-agile-health-care.git'
            }
        }
        
        stage('Create Package') {
            steps {
                echo 'This stage will compile, test, package my application'
                sh 'mvn package'
            }
        }
        
        stage('Generate Test Report') {
            steps {
                echo 'This stage generates Test report using TestNG'
                publishHTML([
                    allowMissing: false, 
                    alwaysLinkToLastBuild: false, 
                    keepAll: false, 
                    reportDir: '/var/lib/jenkins/workspace/Healthcare/target/surefire-reports', 
                    reportFiles: 'index.html', 
                    reportName: 'HTML Report', 
                    reportTitles: '', 
                    useWrapperFileDirectly: true
                ])
            }
        }
        
        stage('Create Docker Image') {
            steps {
                echo 'This stage will create a Docker image'
                sh 'docker build -t ajit0211/healthcare:1.0 .' 
            }
        }
        
      stage('Login to Dockerhub') {
    steps {
        echo 'This stage will log into Dockerhub' 
        withCredentials([usernamePassword(credentialsId: 'Dockerlogin', passwordVariable: 'docker_pass', usernameVariable: 'docker_login')]) {
            sh 'echo $docker_pass | docker login -u $docker_login --password-stdin'
                            }
                      }
                  }

        
        stage('Docker Push Image') {
            steps {
                echo 'This stage will push my new image to Dockerhub'
                sh 'docker push ajit0211/healthcare:1.0'
            }
        }
    }
}
