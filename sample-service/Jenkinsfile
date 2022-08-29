/*6
This pipeline is used to sync git state to deployed state.
Any app.version file update will trigger this pipeline to sync state.
*/

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// CALCULATED VARIABLES
LABEL_NAME  = 'agent-' + new Date().getTime()
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

pipeline {
    agent {
        kubernetes {
            label "$LABEL_NAME"
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins-sa
  containers:
  - name: builder
    image: mayankrkhunt/my-public-repo:jenkins-builder-01
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
    environment {
      GIT_SSH_COMMAND = "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no"
      TAG = "${env.BUILD_NUMBER}"
      registryCredential  = 'Docker'
    }
    stages {
      stage('GitClone') {
        steps {
          container('builder') {
            script {
              git url: 'https://github.com/mayankrkhunt/devops-assessment.git'
            }
          }
        }
      }
      stage('Build') {
        steps {
          container('builder') {
            script {
              docker.withRegistry( '', registryCredential ) {
                sh '''
                  echo $TAG
                  cd devops-assessment/sample-service && docker build -t mayankrkhunt/sample-app:latest .
                  docker push mayankrkhunt/sample-app:latest
                '''
              }
            }
          }
        }
      }
      stage('Deploy') {
        steps {
          container('builder') {
            sh '''
              cd devops-assessment/manifest/
              old=$(cat app-deployment.yaml|grep "sample-app"|cut -d ":" -f3)
              sed -i "s/$old/sample-app-$TAG/g" app-deployment.yml
              kubectl apply -f app-deployment.yaml
            '''
          }
        }
      }
    }
}
