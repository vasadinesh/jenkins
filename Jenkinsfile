
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROJECT_NAME = 'Prometheus-Grafana-Monitoring'
    }

    stages {

        stage('Start Pipeline') {
            steps {
                echo "Starting ${PROJECT_NAME}"
            }
        }

        stage('Git Checkout') {
            steps {
                echo 'Downloading source code from GitHub'
                checkout scm

                sh '''
                    echo "Workspace:"
                    pwd
                    echo "Repository files:"
                    ls -la
                '''
            }
        }

        stage('Check Required Files') {
            steps {
                script {
                    def filePath = sh(
                        script: "find . -type f -name 'grafana-prometheus-updated.sh' -print -quit",
                        returnStdout: true
                    ).trim()

                    if (!filePath) {
                        error "Installation script not found in GitHub repository!"
                    }

                    env.INSTALL_SCRIPT = filePath

                    echo "Installation script found: ${env.INSTALL_SCRIPT}"
                }
            }
        }

        stage('Set Permissions') {
            steps {
                sh '''
                    chmod +x "$INSTALL_SCRIPT"
                    ls -l "$INSTALL_SCRIPT"
                '''
            }
        }

        stage('Validate Shell Script') {
            steps {
                sh '''
                    bash -n "$INSTALL_SCRIPT"
                    echo "Shell script syntax is valid"
                '''
            }
        }

        stage('Install Monitoring Stack') {
            steps {
                echo 'Installing Prometheus, Grafana, Alertmanager and Node Exporter'

                sh '''
                    sudo -n bash "$INSTALL_SCRIPT"
                '''
            }
        }

        stage('Verify Prometheus') {
            steps {
                sh 'systemctl is-active prometheus'
            }
        }

        stage('Verify Grafana') {
            steps {
                sh 'systemctl is-active grafana-server'
            }
        }

        stage('Verify Alertmanager') {
            steps {
                sh 'systemctl is-active alertmanager'
            }
        }

        stage('Verify Node Exporter') {
            steps {
                sh 'systemctl is-active node_exporter'
            }
        }

        stage('Final Verification') {
            steps {
                sh '''
                    echo "Checking monitoring services..."

                    for service in prometheus grafana-server alertmanager node_exporter
                    do
                        if systemctl is-active --quiet "$service"; then
                            echo "$service is RUNNING"
                        else
                            echo "$service is NOT RUNNING"
                            exit 1
                        fi
                    done

                    echo "All monitoring services are running!"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Monitoring stack deployed successfully!'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}
