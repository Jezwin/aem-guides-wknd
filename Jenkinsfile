pipeline {
    agent any
    tools {
        jdk 'JAVA'  // Ensure this matches your Jenkins configuration
        maven 'Maven'  // Ensure this matches your Jenkins configuration
    }
    environment {
        AEM_CREDENTIALS = credentials('aem-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Jezwin/aem-guides-wknd.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean install -Pclassic'
            }
        }
        stage('Deploy to Dev Author') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:4502')
                }
            }
        }
        stage('Deploy to Dev Publish') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:4503')
                }
            }
        }
        stage('Approval for Stage Deployment') {
            steps {
                input message: 'Deploy to Stage?', ok: 'Deploy'
            }
        }
        stage('Deploy to Stage Author') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:5502')
                }
            }
        }
        stage('Deploy to Stage Publish') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:5503')
                }
            }
        }
        stage('Approval for Production Deployment') {
            steps {
                input message: 'Deploy to PROD Author and Preview?', ok: 'Deploy'
            }
        }
        stage('Deploy to Prod Author') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:6502')
                }
            }
        }
        stage('Deploy to Prod Preview') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:6503')
                }
            }
        }
        stage('Approval for Live Deployment') {
            steps {
                input message: 'Deploy to PROD Publish?', ok: 'Deploy'
            }
        }
        stage('Deploy to Prod Publish') {
            steps {
                script {
                    deployPackagesToAEM('http://172.29.73.131:6504')
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed.'
        }
    }
}

def deployPackagesToAEM(String aemUrl) {
    def modules = ['all', 'ui.apps', 'ui.content', 'ui.config']
    modules.each { module ->
        def packagePath = "${module}/target/"
        def zipFiles = findFiles(glob: "${packagePath}*.zip")
        if (zipFiles.length > 0) {
            zipFiles.each { file ->
                bat """
                curl -u ${AEM_CREDENTIALS_USR}:${AEM_CREDENTIALS_PSW} ^
                    -F "file=@${file.path}" ^
                    -F "name=${file.name}" ^
                    -F "force=true" ^
                    -F "install=true" ^
                    ${aemUrl}/crx/packmgr/service.jsp
                """
            }
        } else {
            echo "No ZIP files found in ${packagePath}"
        }
    }
}
