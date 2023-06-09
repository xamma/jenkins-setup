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
          - name: kubectl
            image: google/cloud-sdk:latest
            command:
            - cat
            tty: true
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - cat
            tty: true
            volumeMounts:
              - name: kaniko-secret
                mountPath: /kaniko/.docker
          volumes:
          - name: kaniko-secret
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
        echo 'Checking out code Repository.'
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
            echo 'Testing code integration.'
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
          echo 'Building docker image and pushing to Dockerhub.'
          container('kaniko') {
            sh '/kaniko/executor --context `pwd` --destination xamma/my-jenkins-docker:latest'
            }
        }
    }

    stage('Deploy to Kubernetes') {
      steps {
          echo 'Deploying to Kubernetes.'
          // apply k8s manifests to update application
          container('kubectl') {
            sh 'kubectl apply -f k8s-manifests/'
        }
      }
    }
  }
}