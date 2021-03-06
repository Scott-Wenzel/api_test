pipeline {
    agent {
        kubernetes {
            label 'api_test'  // all your pods will be named with this prefix, followed by a unique id
            idleMinutes 1  // how long the pod will live after no jobs have run on it
            yamlFile 'build-pod.yaml'  // path to the pod definition relative to the root of our project 
        }
    }
    environment {
        DOCKER_REPO_CREDS = credentials('68527666-ec6d-4c9a-afb2-a9e7e9e78431')
        KUBECONFIG = credentials('kubeconfig')
        DOCKERTAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh "docker login -u ${env.DOCKER_REPO_CREDS_USR} -p ${env.DOCKER_REPO_CREDS_PSW}"  // Login   
                    sh "docker build -t jwenzel/api-test:${env.DOCKERTAG} ."  // when we run docker in this step, we're running it via a shell on the docker build-pod container,
                    sh "docker push jwenzel/api-test:${env.DOCKERTAG}"        // which is just connecting to the host docker deaemon
                }
            }
        }
        stage('Deploy via Helm') {
            steps {
                container('helm') {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh "helm upgrade --install --force api-test --namespace=api-test --set image.repository=jwenzel/api-test --set image.tag=${env.DOCKERTAG} ./helm"
                    }                    
                }
            }
       }
    }
}


