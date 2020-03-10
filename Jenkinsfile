pipeline {
  environment {
    registry = "rradi/testgreenimage"
    registryCredential = 'docker'
    dockerImage = ''
  }
  agent any
  stages {

    stage('1: Say Hello - Build init ') {
      steps {
        sh 'echo "Hello World "'
        sh '''
                  echo "Multi-line works too!"
                  ls -lrtha
                '''
      }
    }

    stage('2: Git Clone') {
      steps {
        git 'https://github.com/rradi/UdacityDevOpsCapstone.git'
      }
    }

    stage('3: Lint HTML') {
      steps {
        sh 'tidy -q -e green/ws/index.html'
      }
    }

    stage('4: Docker Image') {
      steps {
        script {
		dockerImage = docker.build registry + ":$BUILD_NUMBER"	

        }
      }
    }
    stage('5:Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
          dockerImage.push()
          }
        }
      }
    }
    stage('6:Remove Unused docker image') {
      steps{
         sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
     stage('7: Context') {
      steps {
        withAWS(credentials: 'aws-static', region: 'us-west-2') {
	  sh '''
	  aws eks --region us-west-2 update-kubeconfig --name app
	  '''
        }

      }
    }
  stage('8: Apply kube') {
      steps {
        withAWS(credentials: 'aws-static', region: 'us-west-2') {
          echo 'Success'
          sh 'kubectl config use-context arn:aws:eks:us-west-2:523856192541:cluster/app'
          sh 'kubectl apply -f green/green-controller.json'
        }

      }
    }
  }
}
