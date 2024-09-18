pipeline {
    agent any
    tools {
        jdk 'JDK11'
        maven 'Maven3'
    }
    environment {
        AEM_USER = credentials('aem-credentials').username
        AEM_PASSWORD = credentials('aem-credentials').password
    }
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'git-credentials', url: 'https://github.com/Jezwin/aem-guides-wknd.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -PautoInstallPackage'
            }
        }
        stage('Deploy to Dev Author') {
            steps {
                script {
                    deployPackagesToAEM('http://localhost:4502')
                }
            }
        }
        stage('Deploy to Dev Publish') {
            steps {
                script {
                    deployPackagesToAEM('http://localhost:4503')
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
        def dir = new File(packagePath)
        if (dir.exists()) {
            dir.eachFile { file ->
                if (file.name.endsWith('.zip')) {
                    sh """
                        curl -u ${AEM_USER}:${AEM_PASSWORD} -F file=@${file.absolutePath} -F name=${file.name} -F force=true -F install=true http://172.29.73.131/crx/packmgr/service.jsp
                    """
                }
            }
        } else {
            echo "No target directory found for module ${module}"
        }
    }
}
