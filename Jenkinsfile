pipeline {
	agent any

stages{

  stage('Generate Local Tag') {
    steps {
        script {
            def shortHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            def timestamp = new Date().format('yyyyMMddHHmmss')
            env.LOCAL_TAG = "v${timestamp}-${shortHash}"

            echo "Generated Local Tag: ${env.LOCAL_TAG}"
        }
    }
    }



    stage('Docker Build & Push') {
        steps {
            withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD_Formation13', variable: 'DOCKER_HUB_PASSWORD')]) {
                echo "print tag : $LOCAL_TAG"
                sh 'docker login -u ybm2i -p $DOCKER_HUB_PASSWORD'
                sh 'docker build -t ybm2i/website-jenkins-yb:$LOCAL_TAG .'
                sh 'docker push ybm2i/website-jenkins-yb:$LOCAL_TAG'
            }
        }
    }

 

    stage('Kubernetes Deployment') {
        steps {
            withKubeConfig([credentialsId: 'kubeconfigYB']) {
                script {
                    sh "sed -i 's#replace#ybm2i/website-jenkins-yb:$LOCAL_TAG#g' k8s_deployment_service.yaml"
                    sh 'kubectl apply -f k8s_deployment_service.yaml'
                }
            }
        }
    }
    stage('Force Rollout Restart') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfigachraf']) {
            script {
                // This forces Kubernetes to restart the deployment and pull the latest image
                sh 'kubectl rollout restart deployment devachraf'
            }
        }
    }
}

}
}





