def VALUESFILE = ""
def INSTALL = ""
pipeline {
//    agent { dockerfile { filename 'Dockerfile.jenkins' } }
    agent any

    environment {
        KUBECONFIG = 'kubeconfig'
        HELM_CHARTS_GIT_REPO = 'https://github.com/infracloudio/app-mono-helmcharts.git'
        HELM_CHARTS_REPO = 'app-mono-helmcharts'
        HELM_CHARTS_BRANCH = 'master'
        GIT = credentials('github-credentials')
        K8S_SERVER = credentials('k8s-server')
        K8S_TILLER_TOKEN = credentials('k8s-tiller-token')
        K8S_CA_BASE64 = credentials('k8s-ca-base84')
    }

    stages {
        stage('Create kubeconfig') {
            steps {
                createKubeconfig()
            }
        }
        stage('Checkout scm') {
            steps {
                checkout scm
            }
        }

        stage('Find app name to deploy') {
            steps {
                script {
                    VALUESFILE = sh(returnStdout: true, script:'git show --name-only --pretty=""')
                    env.APP = VALUESFILE.split('/')[0] // TODO: change index and add error handling
                }
                echo "application to deploy:${APP}"
            }
        }

        stage('Get helm-charts') {
            steps {
                getHelmCharts()
            }
        }

        stage('Deploy helm chart') {
            steps {
                script {
                    INSTALL = sh (
                        script: "helm status ${APP}",
                        returnStatus: true
                    )
                }
                echo "To install? ${INSTALL}"
                helmInstall(INSTALL)
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
    }
}

def createKubeconfig() {
    sh '''
    kubectl config set-cluster k8s-cluster --server=$K8S_SERVER --certificate-authority=$K8S_CA_BASE64
    sed -i 's/certificate-authority/certificate-authority-data/g' $KUBECONFIG
    kubectl config set-credentials tiller --token=$K8S_TILLER_TOKEN
    kubectl config set-context tiller --cluster=k8s-cluster --user=tiller
    kubectl config use-context tiller
    kubectl get ns
    '''
}

def getHelmCharts() {
    sh '''
    git clone https://${GIT_USR}:${GIT_PSW}@${HELM_CHARTS_GIT_REPO}
    cd {HELM_CHARTS_REPO}
    git checkout ${HELM_CHARTS_BRANCH}
    '''
    }

def helmInstall(install) {
    sh 'echo deploying application:$APP'
    sh 'helm init --client-only --home /tmp'
    if (install) {
        sh 'helm install --home /tmp --name $APP -f $APP/values.yaml ${HELM_CHARTS_REPO}/charts/$APP/'
    } else {
        sh 'helm upgrade --home /tmp $APP -f $APP/values.yaml ${HELM_CHARTS_REPO}/charts/$APP/'
    }
}
