pipeline {
    agent any

    environment {
        // Путь к Helm в рабочем пространстве Jenkins, чтобы гарантировать его доступность
        HELM_PATH = "${WORKSPACE}/helm/helm"
        CHART_REPO = "https://github.com/p4v31/nginx-chart"
        CHART_DIR = "${WORKSPACE}/nginx-chart"
        RELEASE_NAME = "my-release"
        NAMESPACE = "devops-tools"
    }

    stages {
        stage('Preparation') {
            steps {
                checkout scm
            }
        }

        stage('Install Helm') {
            steps {
                script {
                    // Установка Helm, если он еще не установлен
                    sh """
                    if [ ! -f ${HELM_PATH} ]; then
                        mkdir -p ${WORKSPACE}/helm
                        curl -fsSL https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz -o helm.tar.gz
                        tar -zxvf helm.tar.gz -C ${WORKSPACE}/helm
                        mv ${WORKSPACE}/helm/linux-amd64/helm ${WORKSPACE}/helm/helm
                    fi
                    """
                }
            }
        }

        stage('Clone Helm Chart Repo') {
            steps {
                script {
                    // Клонирование репозитория Helm чарта
                    sh "rm -rf ${CHART_DIR}"
                    sh "git clone ${CHART_REPO} ${CHART_DIR}"
                }
            }
        }

        stage('Delete Existing Release') {
            steps {
                script {
                    // Удаление существующего релиза
                    sh """
                    ${HELM_PATH} uninstall ${RELEASE_NAME} --namespace ${NAMESPACE} || echo "Release not found. Skipping delete."
                    """
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    // Использование Helm для развертывания с указанием NodePort
                    sh """
                    ${HELM_PATH} version
                    ${HELM_PATH} upgrade --install ${RELEASE_NAME} ${CHART_DIR} \\
                        --namespace ${NAMESPACE} \\
                        --set service.type=NodePort \\
                        --set service.nodePort=32080
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Развертывание завершено успешно.'
        }
        failure {
            echo 'Ошибка при развертывании.'
        }
    }
}
