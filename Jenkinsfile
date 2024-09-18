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
                // No credentialsId needed for public repositories
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
        def packagePath = "${module}${File.separator}target${File.separator}"
        def dir = new File(packagePath)
        if (dir.exists()) {
            dir.eachFile { file ->
                if (file.name.endsWith('.zip')) {
                    // Ensure paths are correctly quoted and escaped for Windows
                    bat """
                    curl -u ${AEM_CREDENTIALS_USR}:${AEM_CREDENTIALS_PSW} ^
                        -F "file=@${file.absolutePath}" ^
                        -F "name=${file.name}" ^
                        -F "force=true" ^
                        -F "install=true" ^
                        ${aemUrl}/crx/packmgr/service.jsp
                    """
                }
            }
        } else {
            echo "No target directory found for module ${module}"
        }
    }
}
