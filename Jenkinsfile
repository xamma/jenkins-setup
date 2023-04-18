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
          - name: python
            image: python:3.11
            command:
            - cat
            tty: true
          - name: kaniko
            image: gcr.io/kaniko-project/executor:latest
            args: ["--dockerfile=Dockerfile", "--context=dir/with/dockerfile", "--destination=<your-dockerhub-username>/<your-dockerhub-repo>:<tag>"]
            env:
            - name: DOCKER_CONFIG
              value: "/kaniko/.docker"
            volumeMounts:
            - mountPath: /kaniko/.docker
              name: docker-config
              readOnly: true
            - mountPath: /kaniko/ssl/certs
              name: kaniko-certs
              readOnly: true
          volumes:
          - name: docker-config
            secret:
              secretName: jenkins-docker-config
          - name: kaniko-certs
            secret:
              secretName: kaniko-certs  
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
          container('kaniko') {
            // Build and Push image to Container-Registry (Dockerhub)
            withCredentials([usernamePassword(credentialsId: '27d39497-23a4-46cc-8f08-15c07f078563', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | /kaniko/executor --dockerfile=Dockerfile --context=. --destination=xamma/my-jenkins-docker:latest --dockerconfig $DOCKER_CONFIG'
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
