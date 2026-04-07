pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy (e.g. v1.0.8)')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_SESSION_TOKEN = credentials('aws-session-token')
        AWS_DEFAULT_REGION = 'us-east-1'
        VAULT_PASS = credentials('ansible-vault-password')
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Ansible Dependencies') {
            steps {
                sh '/usr/bin/pip3 install boto3 botocore --break-system-packages'
                sh 'ansible-galaxy collection install amazon.aws'
            }
        }

        stage('Prepare Vault Password') {
            steps {
                sh 'echo $VAULT_PASS > ~/.vault_pass && chmod 600 ~/.vault_pass'
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh "IMAGE_TAG=${params.IMAGE_TAG} ansible-playbook provision.yml --vault-password-file ~/.vault_pass"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 30
                    PUBLIC_IP=$(aws ec2 describe-instances \
                        --filters "Name=tag:Name,Values=prod-runcalc-*" "Name=instance-state-name,Values=running" \
                        --query "Reservations[-1].Instances[0].PublicIpAddress" \
                        --output text)
                    echo "Deployed to: $PUBLIC_IP"
                    curl -f http://$PUBLIC_IP || exit 1
                '''
            }
        }
    }
}
