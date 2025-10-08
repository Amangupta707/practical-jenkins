pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Amangupta707/practical-jenkins.git'
            }
        }

        stage('Install') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }

        stage('Install Trivy') {
            steps {
                echo 'Installing Trivy vulnerability scanner...'
                sh '''
                    set -e
                    if ! command -v trivy >/dev/null 2>&1; then
                        echo "Trivy not found, installing..."
                        apt-get update -y
                        apt-get install -y wget
                        wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.65.0_Linux-64bit.deb
                        dpkg -i trivy_0.65.0_Linux-64bit.deb
                    else
                        echo "Trivy is already installed."
                    fi
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy vulnerability scan...'
                sh '''
                    mkdir -p trivy-reports
                    trivy fs --format json -o trivy-reports/trivy-report.json .
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-reports/trivy-report.json', fingerprint: true
                }
            }
        }
    }
}
