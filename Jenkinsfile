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
                git url: 'https://github.com/Jezwin/aem-guides-wknd.git', branch: 'main'
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
                    deployToAEM('http://localhost:4502', 'admin', 'admin')
                }
            }
        }
        stage('Deploy to Dev Publish') {
            steps {
                script {
                    deployToAEM('http://localhost:4503', 'admin', 'admin')
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

def deployToAEM(String aemUrl, String username, String password) {
    sh """
        curl -u ${username}:${password} -F file=@target/your-package.zip -F name=your-package -F force=true -F install=true ${aemUrl}/crx/packmgr/service.jsp
    """
}
