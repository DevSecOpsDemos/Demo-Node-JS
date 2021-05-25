pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
        items:
        - key: .dockerconfigjson
          path: config.json
"""
        }
    }
    
    stages {
        stage("Checkout code") {
            steps {
                git credentialsId: 'git_id', url: 'https://github.com/DevSecOpsDemos/Demo-Node-JS.git'
            }
        }
        stage("Push Docker image") {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh ''' #!/busybox/sh
                    ls -al
                    /kaniko/executor -f `pwd`/Dockerfile --context="dir:///home/jenkins/agent/workspace/${JOB_NAME}/" --destination="sachinpgade/devsecopsdemo:2.0.0"
                    '''
                }
            }
        }
        stage('Deploy to GKE') {
            steps{
                container('kubectl') {
                    sh 'ls -al'
                    step([$class: 'KubernetesEngineBuilder', projectId: 'devopsindiasummit', clusterName: 'demo-devsecops', location: 'us-central1-c', manifestPattern: 'deploy', credentialsId: 'devopsindiasummit', verifyDeployments: true])
                }       
            }
        }
    }    
}
