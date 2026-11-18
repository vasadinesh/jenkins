// pipeline {
//     agent any

//     stages {
//         stage('Build') {
//             steps {
//                 echo 'Building...'
//                 // Add build steps here
//             }
//         }
//         stage('Test') {
//             steps {
//                 echo 'Testing...'
//                 // Add test steps here
//             }
//         }
//         stage('Deploy') {
//             steps {
//                 echo 'Deploying...'
//                 // Add deploy steps here
//             }
//         }
//     }

//     post {
//         always {
//             echo 'This will always run after the stages.'
//         }
//         success {
//             echo 'This will run only if the pipeline succeeds.'
//         }
//         failure {
//             echo 'This will run only if the pipeline fails.'
//         }
//     }
// }



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

                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Check Required Files') {
            steps {
                sh '''
                    set -e

                    if [ ! -f grafana-prometheus-updated.sh ]; then
                        echo "Installation script not found!"
                        exit 1
                    fi

                    echo "Installation script found successfully"
                '''
            }
        }

        stage('Set Permissions') {
            steps {
                sh '''
                    chmod +x grafana-prometheus-updated.sh
                    ls -l grafana-prometheus-updated.sh
                '''
            }
        }

        stage('Validate Shell Script') {
            steps {
                sh '''
                    bash -n grafana-prometheus-updated.sh
                    echo "Shell script syntax is valid"
                '''
            }
        }

        stage('Install Monitoring Stack') {
            steps {
                echo 'Installing Prometheus, Grafana, Alertmanager and Node Exporter'

                sh '''
                    sudo bash grafana-prometheus-updated.sh
                '''
            }
        }

        stage('Verify Prometheus') {
            steps {
                sh '''
                    systemctl is-active prometheus
                '''
            }
        }

        stage('Verify Grafana') {
            steps {
                sh '''
                    systemctl is-active grafana-server
                '''
            }
        }

        stage('Verify Alertmanager') {
            steps {
                sh '''
                    systemctl is-active alertmanager
                '''
            }
        }

        stage('Verify Node Exporter') {
            steps {
                sh '''
                    systemctl is-active node_exporter
                '''
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
