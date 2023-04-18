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
            image: gcr.io/kaniko-project/executor:debug
            command:
            - cat
            volumeMounts:
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          volumes:
          - name: kanico-secret
            secret:
              secretName: dockercred
              items:
                - key: .dockerconfigjson
                  path: config.json
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
            sh '/kaniko/executor --context `pwd` --destination xamma/my-jenkins-docker:latest'
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
    }
  }
}
