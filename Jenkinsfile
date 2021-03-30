pipeline {
    agent any 
    stages {
        stage('Build Image') { 
            steps {
                echo 'Build docker image in local docker library...'
                sh './gradlew bootBuildImage'
            }
        }
        stage('Upload Image to Registry') { 
            steps {
                echo 'Push docker image to docker hub...'
                sh 'docker login harbor.tanzu.platformdemosm.com -u admin -p harbor_admin'
                sh 'docker tag tkg-spring-music-pipeline:1.0 harbor.tanzu.platformdemosm.com/library/tkg-spring-music:${BUILD_NUMBER}'
                sh 'docker tag tkg-spring-music-pipeline:1.0 harbor.tanzu.platformdemosm.com/library/tkg-spring-music:latest'
                sh 'docker push harbor.tanzu.platformdemosm.com/library/tkg-spring-music:${BUILD_NUMBER}'
            }
        }
        stage('Deploy to TKG Cluster') { 
            steps {
                echo 'Deploy application to TKG...'
                sh 'export KUBECONFIG=/var/lib/jenkins/kubeconfig'
                sh 'sudo kubectl config use-context sm-tkg-demo-cluster-admin@sm-tkg-demo-cluster --kubeconfig /var/lib/jenkins/kubeconfig'
                sh 'kubectl delete deployment spring-music-deploy'
                sh 'kubectl delete service spring-music-ci-service'
                sh 'kubectl apply -f spring-music-app-deployment.yml'
            }
        }
        stage('Cleanup') { 
            steps {
                echo 'Remove local images and tags...'
                sh 'docker image rm tkg-spring-music-pipeline:1.0'
                sh 'docker image rm savmas22/tkg-spring-music-pipeline:${BUILD_NUMBER}'
                sh 'docker image rm savmas22/tkg-spring-music-pipeline:latest'
            }
        }
    }
}
