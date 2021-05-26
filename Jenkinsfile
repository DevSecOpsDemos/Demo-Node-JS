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
  - name: shell
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
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
        
        stage("Build ") {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh ''' #!/busybox/sh
                    ls -al
                    /kaniko/executor -f `pwd`/Dockerfile --context="dir:///home/jenkins/agent/workspace/${JOB_NAME}/" --destination="sachinpgade/devsecopsdemo:2.0.0"
                    '''
                }
            }
        }
        
        stage("Docker Image Scan using Clair") {
            steps {
                container('shell') {
                    sh '''
                    apt-get update && apt-get install -y curl
                    curl -L https://github.com/optiopay/klar/releases/download/v1.5/klar-1.5-linux-amd64 -o /usr/local/bin/klar && chmod +x /usr/local/bin/klar
                    CLAIR_ADDR=http://35.225.200.111:6060 CLAIR_OUTPUT=High CLAIR_THRESHOLD=10 klar sachinpgade/devsecopsdemo:2.0.0
                    '''
                }
            }
        }
        
        stage("Push images to Nexus") {
            steps {
                sh 'echo In Progress'
            }
        }
        
        stage('Deploy to QA') {
            steps{
                sh 'echo In Progress'      
            }
        }
        
        stage('Publish artifacts to Stage') {
            steps{
                sh 'echo In Progress'      
            }
        }
        
        stage('Deploy to Stage') {
            steps{
                container('kubectl') {
                    sh 'ls -al'
                    step([$class: 'KubernetesEngineBuilder', projectId: 'devopsindiasummit', clusterName: 'demo-devsecops', location: 'us-central1-c', manifestPattern: 'deploy', credentialsId: 'devopsindiasummit', verifyDeployments: true])
                }       
            }
        }
        
    }    
}
