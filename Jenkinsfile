def PROJECT_NAME = "sample-api"
def gitUrl = "https://github.com/BYCJY/${PROJECT_NAME}.git"
def gitOpsUrl = "https://github.com/BYCJY/sample-gitops.git"
def opsBranch = "main"
/////////////////////////////
pipeline {
     environment {
         PATH = "$PATH:/usr/local/bin/"  //maven, skaffold, argocd,jq path
       }
    agent any
    stages {
        stage('Build') {
            steps {
                checkout scm: [
                        $class: "GitSCM",
                        userRemoteConfigs: [[url: "${gitUrl}",
                        credentialsId: "git-credential" ]],     //credential 이름이 jenkins에 등록된 이름과 동일해야 함
                        branches: [[name: "refs/tags/${TAG}"]]],
                    poll: false
                script{
                    docker.withRegistry("","imageRegistry-credential"){   //credential 이름이 jenkins에 등록된 이름과 동일해야 함, jenkins에 docker deploy 권한 필요
                        sh "skaffold build -p dev -t ${TAG}"
                    }
                    // mac local 일때만 사용 linux 환경에서는 docker.withRegistry 사용
                    //sh "skaffold build -p dev -t ${TAG}"
                }
            }
        }
        stage('workspace clear'){
            steps {
                cleanWs()
            }
        }
        stage('GitOps update') {
            steps {
                    print "======kustomization.yaml tag update====="
                    withCredentials([
                        gitUsernamePassword(credentialsId: 'git-credential', gitToolName: 'Default')
                    ]) {
                        sh """
                        git clone ${gitOpsUrl}
                        cd ./sample-gitops/sample-api/rolling-update-no-istio
                        kustomize edit set image oscka/sample-api:${TAG}
                        # 로컬외에는 주석 제거한다
                        git config --global user.email "admin@demo.com"
                        git config --global user.name "admin"
                        git add .
                        git commit -am 'update image tag ${TAG}'
                        git remote set-url --push origin ${gitOpsUrl}
                        git push origin ${opsBranch}
                        """
                    }
                    print "git push finished !!!"
                }                
                
            
        }
    }
}
