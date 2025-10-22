pipeline {
    agent any

    environment {
        SONARQUBE = credentials('SonarQube')
        GITHUB_CREDS = credentials('GitHub')
        DOCKERHUB_CREDS = credentials('DockerHub')
        TWILIO = credentials('Twilio')
    }

    parameters {
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Backend Docker Tag')
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Frontend Docker Tag')
    }

    stages {

        // 🧾 1️⃣ Checkout Code
        stage('Checkout Code') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    echo "📦 Pulling source code from GitHub..."
                    git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/rehanazam123/Jenkins.git'
                    echo "✅ Code checkout complete!"

                    // WhatsApp notify
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Checkout Code", start, "✅ Completed")
                }
            }
        }

        // 🔍 2️⃣ SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    echo "🚀 Running SonarQube Analysis..."
                    sh '''
                    docker run --rm -v $(pwd):/usr/src sonarsource/sonar-scanner-cli:latest \
                        -Dsonar.projectKey=WounderLust \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://192.168.18.145:9000 \
                        -Dsonar.login=${SONARQUBE}
                    '''
                    echo "✅ SonarQube Analysis Completed!"
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("SonarQube Analysis", start, "✅ Completed")
                }
            }
        }

        // 🐳 3️⃣ Build Docker Images
        stage('Docker: Build Images') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    echo "🔨 Building Backend & Frontend Docker Images..."
                    dir('backend') {
                        sh "docker build -t rehanazam/wounderlust-backend:${params.BACKEND_DOCKER_TAG} ."
                    }
                    dir('frontend') {
                        sh "docker build -t rehanazam/wounderlust-frontend:${params.FRONTEND_DOCKER_TAG} ."
                    }
                    echo "✅ Docker Images Built Successfully!"
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Docker: Build Images", start, "✅ Completed")
                }
            }
        }

        // 🧠 4️⃣ Trivy Security Scan
        stage('Trivy: Scan Built Images') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    sh '''
                    mkdir -p trivy-reports
                    if [ ! -f html.tpl ]; then
                        wget -q https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl
                    fi
                    docker run --rm -v $(pwd):/app -w /app aquasec/trivy image \
                        rehanazam/wounderlust-backend:${BACKEND_DOCKER_TAG} \
                        --severity LOW,MEDIUM,HIGH,CRITICAL \
                        --format template \
                        --template "@/app/html.tpl" \
                        -o trivy-reports/backend_report.html || true
                    docker run --rm -v $(pwd):/app -w /app aquasec/trivy image \
                        rehanazam/wounderlust-frontend:${FRONTEND_DOCKER_TAG} \
                        --severity LOW,MEDIUM,HIGH,CRITICAL \
                        --format template \
                        --template "@/app/html.tpl" \
                        -o trivy-reports/frontend_report.html || true
                    '''
                    archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Trivy Scan", start, "✅ Completed")
                }
            }
        }

        // 📤 5️⃣ Docker Push
        stage('Docker: Push to Hub') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        echo "🚀 Logging into Docker Hub..."
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push rehanazam/wounderlust-backend:${BACKEND_DOCKER_TAG}
                        docker push rehanazam/wounderlust-frontend:${FRONTEND_DOCKER_TAG}
                        '''
                    }
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Docker Push", start, "✅ Completed")
                }
            }
        }

        // 🧱 6️⃣ Deploy
        stage('Docker: Compose Up') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    sh '''
                    docker-compose down || true
                    docker-compose up -d
                    '''
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Docker Compose Up", start, "✅ Completed")
                }
            }
        }

        // 🧹 7️⃣ Cleanup
        stage('Cleanup Old Images') {
            steps {
                script {
                    def start = System.currentTimeMillis()
                    sh 'docker image prune -a -f'
                    def sendWhatsApp = { stageName, startMillis, status ->
                        def durationSec = ((System.currentTimeMillis() - startMillis)/1000).toInteger()
                        def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                        def message = "${status} Stage ${stageName}!\n⏱ Duration: ${durationSec} sec\n🕒 Completed at: ${timestamp}"
                        withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                            sh """
                            curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                                --data-urlencode "To=whatsapp:+923284084920" \\
                                --data-urlencode "From=whatsapp:+14155238886" \\
                                --data-urlencode "Body=${message}" \\
                                -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                            """
                        }
                    }
                    sendWhatsApp("Cleanup Old Images", start, "✅ Completed")
                }
            }
        }

    }

    post {
        failure {
            script {
                def failedStage = currentBuild.rawBuild.getExecution().getCurrentHeads()[0].getDisplayName()
                def timestamp = new Date().format("HH:mm:ss", TimeZone.getTimeZone('Asia/Karachi'))
                def message = "💀 Pipeline Failed at stage ${failedStage}!\n🕒 Time: ${timestamp}"
                withCredentials([usernamePassword(credentialsId: 'Twilio', usernameVariable: 'TWILIO_USID', passwordVariable: 'TWILIO_AUTH_TOKEN')]) {
                    sh """
                    curl -X POST https://api.twilio.com/2010-04-01/Accounts/${TWILIO_USID}/Messages.json \\
                        --data-urlencode "To=whatsapp:+923284084920" \\
                        --data-urlencode "From=whatsapp:+14155238886" \\
                        --data-urlencode "Body=${message}" \\
                        -u "${TWILIO_USID}:${TWILIO_AUTH_TOKEN}"
                    """
                }
            }
        }
    }
}
