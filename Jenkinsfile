pipeline {
    agent any
    tools {
        jdk 'JAVA' // Ensure this matches the configured JDK in Jenkins tools
        maven 'Maven' // Ensure this matches the configured Maven in Jenkins tools
    }
    environment {
        AEM_CREDENTIALS = credentials('aem-credentials') // Stores the credential ID securely
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
                        curl -u ${AEM_CREDENTIALS_USR}:${AEM_CREDENTIALS_PSW} -F file=@${file.absolutePath} -F name=${file.name} -F force=true -F install=true ${aemUrl}/crx/packmgr/service.jsp
                    """
                }
            }
        } else {
            echo "No target directory found for module ${module}"
        }
    }
}
