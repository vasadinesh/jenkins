
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

        // STAGE 1: Start Pipeline
        stage('Start Pipeline') {
            steps {
                echo "Starting ${PROJECT_NAME}"
            }
        }

        // STAGE 2: Git Checkout
        stage('Git Checkout') {
            steps {
                echo 'Downloading code from GitHub'

                deleteDir()
                checkout scm

                sh '''
                    echo "========== WORKSPACE =========="
                    pwd

                    echo "========== REPOSITORY FILES =========="
                    ls -la

                    echo "========== ALL FILES =========="
                    find . -maxdepth 5 -type f -not -path './.git/*'
                '''
            }
        }

        // STAGE 3: Find Installation Script
        stage('Check Required Files') {
            steps {
                script {
                    def filePath = sh(
                        script: """
                            find . -type f \
                            -not -path './.git/*' \
                            -iname 'grafana-prometheus-updated.sh' \
                            -print -quit
                        """,
                        returnStdout: true
                    ).trim()

                    if (filePath == '') {
                        error """
                        INSTALLATION SCRIPT NOT FOUND!

                        Please add grafana-prometheus-updated.sh
                        to your GitHub repository.
                        """
                    }

                    env.INSTALL_SCRIPT = filePath

                    echo "Installation script found: ${filePath}"
                }
            }
        }

        // STAGE 4: Set Permissions
        stage('Set Permissions') {
            steps {
                sh '''
                    set -e

                    chmod +x "$INSTALL_SCRIPT"

                    echo "File permissions updated:"
                    ls -l "$INSTALL_SCRIPT"
                '''
            }
        }

        // STAGE 5: Validate Shell Script
        stage('Validate Shell Script') {
            steps {
                sh '''
                    set -e

                    bash -n "$INSTALL_SCRIPT"

                    echo "Shell script syntax validation successful"
                '''
            }
        }

        // STAGE 6: Check Sudo Access
        stage('Check Sudo Access') {
            steps {
                sh '''
                    set -e

                    sudo -n true

                    echo "Sudo access is available"
                '''
            }
        }

        // STAGE 7: Install Monitoring Stack
        stage('Install Monitoring Stack') {
            steps {
                echo 'Installing Prometheus, Grafana, Alertmanager and Node Exporter'

                sh '''
                    set -e

                    sudo -n bash "$INSTALL_SCRIPT"
                '''
            }
        }

        // STAGE 8: Verify Prometheus
        stage('Verify Prometheus') {
            steps {
                sh '''
                    systemctl is-active prometheus
                '''
            }
        }

        // STAGE 9: Verify Grafana
        stage('Verify Grafana') {
            steps {
                sh '''
                    systemctl is-active grafana-server
                '''
            }
        }

        // STAGE 10: Verify Alertmanager
        stage('Verify Alertmanager') {
            steps {
                sh '''
                    systemctl is-active alertmanager
                '''
            }
        }

        // STAGE 11: Verify Node Exporter
        stage('Verify Node Exporter') {
            steps {
                sh '''
                    systemctl is-active node_exporter
                '''
            }
        }

        // STAGE 12: Final Verification
        stage('Final Verification') {
            steps {
                sh '''
                    set -e

                    echo "========== SERVICE STATUS =========="

                    for service in prometheus grafana-server alertmanager node_exporter
                    do
                        if systemctl is-active --quiet "$service"; then
                            echo "$service: RUNNING"
                        else
                            echo "$service: FAILED"
                            exit 1
                        fi
                    done

                    echo "===================================="
                    echo "ALL SERVICES ARE RUNNING SUCCESSFULLY"
                '''
            }
        }
    }

    post {

        success {
            echo 'SUCCESS: Monitoring stack deployed successfully!'
        }

        failure {
            echo 'FAILED: Please check Jenkins Console Output.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}
