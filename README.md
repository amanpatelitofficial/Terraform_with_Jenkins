# Terraform_with_Jenkins

## How to manage Terraform through Jenkins

### Step a) Update the credentials of GitHub in Jenkin's credentials.

![image](https://github.com/user-attachments/assets/28a12651-8a79-40c4-b6a4-f7b6303d6980)

### Step b) Update the Access key of AWS in Jenkin's credentials by Secret Text

![image](https://github.com/user-attachments/assets/b55565da-b86f-46cd-b3d2-b4f78c19c34c)

### Step c) Update the Secret key of AWS in Jenkin's credentials by Secret Text

![image](https://github.com/user-attachments/assets/57a9b932-fa8a-4bad-b8cb-b38b109b728b)

### Step d) Now create the terraform Pipeline in Parameterized form

![image](https://github.com/user-attachments/assets/f1a5c823-9dbe-4911-8c7c-fe0ca5d9fd44)

### Step e) Jenkins Pileline

![image](https://github.com/user-attachments/assets/88a213b0-cf62-4816-a1be-cf9569d85590)

````bash

pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    parameters {
        string(name: 'action', defaultValue: 'apply', description: 'terraform action: apply or destroy')
    }

    stages {
        stage('Checkout from Git') {                        
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/Aseemakram19/starbucks-kubernetes.git'
            }
        }

        stage('terraform version') {
            steps {
                sh 'terraform --version'
            }
        }

        stage('terraform init') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform init \
                    -backend-config="access_key=$AWS_ACCESS_KEY_ID" \
                    -backend-config="secret_key=$AWS_SECRET_ACCESS_KEY"
                    '''
                }
            }
        }

        stage('terraform validate') {
            steps {
                dir('terraform') {
                    sh 'terraform validate'
                }
            }
        }

        stage('terraform plan') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform plan \
                    -var="access_key=$AWS_ACCESS_KEY_ID" \
                    -var="secret_key=$AWS_SECRET_ACCESS_KEY"
                    '''
                }
            }
        }

        stage('terraform apply/destroy') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform ${action} --auto-approve \
                    -var="access_key=$AWS_ACCESS_KEY_ID" \
                    -var="secret_key=$AWS_SECRET_ACCESS_KEY"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ terraform execution completed successfully!'
        }
        failure {
            echo '❌ terraform execution failed! Check the logs.'
        }
    }
}

````

