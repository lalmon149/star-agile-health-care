pipeline { 
    agent any
    tools {
        maven 'M2_HOME'
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
        stage('AWS-Login') {
      steps {
       withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Awsaccess', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
           }
        }  
    }       
          
stage('Terraform Operations for test workspace') {
      steps {
        script {
          sh '''
            terraform workspace select test || terraform workspace new test
            terraform init
            terraform plan
            terraform destroy -auto-approve
          '''
        }
      }
    }
    stage('Terraform destroy & apply for test workspace') {
      steps {
        sh 'terraform apply -auto-approve'
      }
    }
    stage('get kubeconfig') {
      steps {
        sh 'aws eks update-kubeconfig --region us-east-1 --name test-cluster'
        sh 'kubectl get nodes'
      }
    }
    stage('Deploying the application') {
      steps {
        sh 'kubectl apply -f app-deploy.yml'
        sh 'kubectl get svc'
      }
    }
    stage('Terraform Operations for Production workspace') {
      when {
        expression {
          return currentBuild.currentResult == 'SUCCESS'
        }
      }
      steps {
        script {
          sh '''
            terraform workspace select prod || terraform workspace new prod
            terraform init
            terraform plan
            terraform destroy -auto-approve
          '''
        }
      }
    }
    stage('Terraform destroy & apply for production workspace') {
      steps {
        sh 'terraform apply -auto-approve'
      }
    }
    stage('get kubeconfig for production') {
      steps {
        sh 'aws eks update-kubeconfig --region us-east-1 --name prod-cluster'
        sh 'kubectl get nodes'
      }
    }
    stage('Deploying the application to production') {
      steps {
        sh 'kubectl apply -f app-deploy.yml'
        sh 'kubectl get svc'
      }
    }
  }
}
        
    

