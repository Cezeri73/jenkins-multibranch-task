pipeline {
    agent any
    
    // Jenkins Tools'da tanımlayacağımız Node ismini buraya yazıyoruz
    tools {
        nodejs 'node17' 
    }

    environment {
        // Branch ismine göre dinamik port ve image adı belirliyoruz
        // Main ise 3000, değilse (dev) 3001
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Kodun Jenkins'e çekilmesi
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Node modüllerini yükle
                sh 'npm install'
            }
        }

        stage('Docker Build') {
            steps {
                // Image'ı v1.0 tag'i ile build et
                sh "docker build -t ${IMAGE_NAME}:v1.0 ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // SADECE BU ENV'E AIT container'ı temizle (Advanced Task gereksinimi)
                    // Diğer branch'in container'ı (örneğin dev çalışırken main) silinmez.
                    sh """
                    docker ps -q --filter "name=${IMAGE_NAME}" | xargs -r docker stop
                    docker ps -aq --filter "name=${IMAGE_NAME}" | xargs -r docker rm
                    """
                    
                    // Yeni container'ı belirlenen portla çalıştır
                    // Sol taraf (APP_PORT) tarayıcıdan gireceğimiz port, sağ taraf (3000) uygulamanın iç portu
                    sh "docker run -d --name ${IMAGE_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}:v1.0"
                    
                    echo "Deployment Başarılı! URL: http://localhost:${APP_PORT}"
                }
            }
        }
    }
}
