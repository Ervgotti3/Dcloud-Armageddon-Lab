pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(name: 'STACK', choices: ['Tokyo', 'Sao_Paulo', 'Global'], description: 'Which Terraform stack to run')
        choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Terraform action')
        booleanParam(name: 'ENABLE_SAOPAULO_ACCEPT', defaultValue: false, description: 'Used only for Tokyo stack')
        booleanParam(name: 'CONFIRM_DESTROY', defaultValue: false, description: 'Must be true to allow destroy')
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        TF_IN_AUTOMATION   = 'true'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                  terraform version
                  aws --version
                  aws sts get-caller-identity
                '''
            }
        }

        stage('Pre-Check Safety Rules') {
            steps {
                script {
                    if (params.ACTION == 'destroy' && !params.CONFIRM_DESTROY) {
                        error("Destroy blocked: set CONFIRM_DESTROY=true before running destroy.")
                    }

                    if (params.ACTION == 'destroy') {
                        echo "Recommended destroy order for Lab 3: Global -> Sao_Paulo -> Tokyo"
                    }

                    if (params.ACTION == 'apply' && params.STACK == 'Global') {
                        echo "Reminder: Global should normally be applied after Tokyo and Sao_Paulo are already created."
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    if (params.STACK == 'Tokyo') {
                        sh 'terraform -chdir=Lab_3/Tokyo init -input=false'
                    }

                    if (params.STACK == 'Sao_Paulo') {
                        sh 'terraform -chdir=Lab_3/Sao_Paulo init -input=false'
                    }

                    if (params.STACK == 'Global') {
                        sh 'terraform -chdir=Lab_3/Global init -input=false'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            when {
                expression { params.ACTION != 'destroy' }
            }
            steps {
                script {
                    if (params.STACK == 'Tokyo') {
                        sh 'terraform -chdir=Lab_3/Tokyo validate'
                    }

                    if (params.STACK == 'Sao_Paulo') {
                        sh 'terraform -chdir=Lab_3/Sao_Paulo validate'
                    }

                    if (params.STACK == 'Global') {
                        sh 'terraform -chdir=Lab_3/Global validate'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    if (params.STACK == 'Tokyo') {
                        if (params.ACTION == 'destroy') {
                            sh '''
                              terraform -chdir=Lab_3/Tokyo plan \
                                -destroy \
                                -input=false \
                                -var="enable_saopaulo_accept=${ENABLE_SAOPAULO_ACCEPT}" \
                                -out=tfplan
                            '''
                        } else {
                            sh '''
                              terraform -chdir=Lab_3/Tokyo plan \
                                -input=false \
                                -var="enable_saopaulo_accept=${ENABLE_SAOPAULO_ACCEPT}" \
                                -out=tfplan
                            '''
                        }
                    }

                    if (params.STACK == 'Sao_Paulo') {
                        if (params.ACTION == 'destroy') {
                            sh '''
                              terraform -chdir=Lab_3/Sao_Paulo plan \
                                -destroy \
                                -input=false \
                                -out=tfplan
                            '''
                        } else {
                            sh '''
                              terraform -chdir=Lab_3/Sao_Paulo plan \
                                -input=false \
                                -out=tfplan
                            '''
                        }
                    }

                    if (params.STACK == 'Global') {
                        if (params.ACTION == 'destroy') {
                            sh '''
                              terraform -chdir=Lab_3/Global plan \
                                -destroy \
                                -input=false \
                                -out=tfplan
                            '''
                        } else {
                            sh '''
                              terraform -chdir=Lab_3/Global plan \
                                -input=false \
                                -out=tfplan
                            '''
                        }
                    }
                }
            }
        }

        stage('Manual Approval') {
            when {
                expression { params.ACTION == 'apply' || params.ACTION == 'destroy' }
            }
            steps {
                input message: "Proceed with ${params.ACTION} for ${params.STACK}?", ok: "Continue"
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                script {
                    if (params.STACK == 'Tokyo') {
                        sh 'terraform -chdir=Lab_3/Tokyo apply -input=false -auto-approve tfplan'
                    }

                    if (params.STACK == 'Sao_Paulo') {
                        sh 'terraform -chdir=Lab_3/Sao_Paulo apply -input=false -auto-approve tfplan'
                    }

                    if (params.STACK == 'Global') {
                        sh 'terraform -chdir=Lab_3/Global apply -input=false -auto-approve tfplan'
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                script {
                    if (params.STACK == 'Tokyo') {
                        echo 'Destroying Tokyo. This should normally be the LAST destroy step.'
                        sh 'terraform -chdir=Lab_3/Tokyo apply -input=false -auto-approve tfplan'
                    }

                    if (params.STACK == 'Sao_Paulo') {
                        echo 'Destroying Sao_Paulo. Global should normally be destroyed first.'
                        sh 'terraform -chdir=Lab_3/Sao_Paulo apply -input=false -auto-approve tfplan'
                    }

                    if (params.STACK == 'Global') {
                        echo 'Destroying Global first.'
                        sh 'terraform -chdir=Lab_3/Global apply -input=false -auto-approve tfplan'
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "SUCCESS: ${params.STACK} ${params.ACTION}"
        }
        failure {
            echo "FAILED: Check Jenkins console output carefully."
        }
    }
}