node('kustomize') {
    stage ('pull code') {
        sh "git clone https://github.com/edwin/infrastructure-as-a-code-using-kustomize source "
    }
    stage ('build') {
        dir("source") {
            sh "kustomize build overlays/sit | kubectl apply  -n dev-apps -f -"
        }
    }
}