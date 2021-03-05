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
                sh '/usr/local/bin/tkgi login -a api.tkgi.platformdemosm.com -u admin -p ${TKGI_ADMIN_PASS} --ca-cert /tmp/ssh/root_ca_cert.pem'
                sh '/usr/local/bin/tkgi get-credentials demo-cluster'
                sh '/usr/local/bin/kubectl delete deployment spring-music-deploy'
                sh '/usr/local/bin/kubectl apply -f spring-music-deploy.yml'
                sh '/usr/local/bin/kubectl set image deployments/spring-music-deploy spring-music-app=savmas22/tkgi-spring-music-pipeline:${BUILD_NUMBER}'
            }
        }
        stage('Cleanup') { 
            steps {
                echo 'Remove local images and tags...'
                sh 'docker image rm tkgi-spring-music-pipeline:1.0'
                sh 'docker image rm savmas22/tkgi-spring-music-pipeline:${BUILD_NUMBER}'
                sh 'docker image rm savmas22/tkgi-spring-music-pipeline:latest'
            }
        }
    }
}
