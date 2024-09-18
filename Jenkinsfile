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
                bat 'mvn clean install'
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
        // Use 'findFiles' to locate ZIP files in the packagePath
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
