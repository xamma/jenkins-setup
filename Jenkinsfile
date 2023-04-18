pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          serviceAccountName: jenkins-admin
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          - name: python
            image: python:3.11
            command:
            - cat
            tty: true
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }

  stages {
    stage('Checkout') {
      steps {
        // Checkout repository
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/master']],
          userRemoteConfigs: [[url: 'https://github.com/xamma/jenkins-setup.git']]
        ])
      }
    }

    stage('Test') {

        steps {
            container('python') {
                // install dependencies, switch to directory
                dir('src') {
                    sh 'pip install -r requirements.txt'
                }

                // test code and fail pipeline if unsuccessful
                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    dir('src/main') {
                        sh 'pytest'
                    }
                }
            }
        }
    }

    stage('Build and Push Docker Image') {
      steps {
          container('docker') {
            // Build docker image
            sh 'docker build -t my-jenkins-docker .'
            sh 'docker tag my-jenkins-docker:latest xamma/my-jenkins-docker:latest'

            // Push image to Container-Registry (Dockerhub)
            withCredentials([usernamePassword(credentialsId: '27d39497-23a4-46cc-8f08-15c07f078563', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'docker login -u $USERNAME -p $PASSWORD'  // use CR-login data from credentials
                // sh 'docker login -u $USERNAME -p $PASSWORD ghcr.io'  // Container-Registry-Anmeldedaten aus dem Credential verwenden
                sh 'docker push xamma/my-jenkins-docker:latest'  // push docker image to registry
            }
          }
      }
    }

    stage('Deploy to Kubernetes') {
    //   when {
    //     expression {
    //       return currentBuild.result == 'SUCCESS'
    //     }
    //   }
      steps {
        // apply k8s manifests to update application
        sh 'kubectl apply -f k8s-manifests/'
      }
      // Only run when tests where successfull
    }
  }
}
