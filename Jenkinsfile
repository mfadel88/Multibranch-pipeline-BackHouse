

pipeline {
    agent { label 'slave1' }
    parameters{
        choice(name: 'ENV', choices: {'dev', 'test', 'prod','release'})
    }
    environment {
        dockerhub=credentials('Docker_Hub')
    }
    stages {
        stage('Build & Deploy') {
     steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh """
                        docker login -u $dockerhub_USR -p $dockerhub_PSW
                        docker build -t mfadel8/app:$BUILD_NUMBER .
                        docker push mfadel8/app:$BUILD_NUMBER
                        echo ${BUILD_NUMBER} > ../build
                        """
                } else if (env.BRANCH_NAME == 'prod' || env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'test') {
                withCredentials([file(credentialsId: 'config', variable: 'cfg')]){
                        sh """
                        if [-f build]; then
                            export BUILD_NUMBER=\$(cat ../build)
                        else
                            export BUILD_NUMBER=0
                        fi
                        mv Deployment/deploy.yaml Deployment/deploy
                        cat Deployment/deploy | envsubst > Deployment/deploy.yaml
                        rm -f Deployment/deploy
                        kubectl apply --kubeconfig=${cfg} -f Deployment/service.yaml
                        kubectl apply --kubeconfig=${cfg} -f Deployment/deploy.yaml
                        """
                 }
                } 

                }
            }
        }
    }

    post {
        success {
            echo 'This will run only if successful'
        }
    }
}
