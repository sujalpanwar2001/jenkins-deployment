pipeline {
    agent any

    environment {
        CI = true
        ARTIFACTORY_ACCESS_TOKEN = credentials('artifactory-access-token')
        GITLAB_CREDENTIALS_ID = 'gitlab-access-token1'
        PEM_KEY = credentials('sujal_aws')
    }

    tools {
        maven "maven"
        jdk "jdk17"
    }

    stages {
        // Code Checkout Stage
        stage('CODE CHECKOUT') {
            steps {
                script {
                    // Securely access GitLab credentials
                    withCredentials([usernamePassword(credentialsId: env.GITLAB_CREDENTIALS_ID,
                                                    passwordVariable: 'GITLAB_ACCESS_TOKEN',
                                                    usernameVariable: 'GITLAB_USERNAME')]) {
                        git credentialsId: env.GITLAB_CREDENTIALS_ID,
                           url: 'https://git.nagarro.com/GITG00641/Java/sujal-panwar.git',
                           branch: 'master'
                    }
                }
            }
        }

        // Build & SonarQube Analysis Stage
        stage('BUILD && SonarQube Analysis') {
            steps {
                sh """
                    cd DEVOPS/Jenkins/dummyproject/
                    mvn clean install sonar:sonar \
                    -Dsonar.projectKey=dummy-project-2-pipeline \
                    -Dsonar.projectName='dummy-project-2-pipeline' \
                    -Dsonar.host.url=http://192.168.29.163:9000 \
                    -Dsonar.token=sqp_4912a042f242ac830f84b72c969d4237ba10da06
                """
            }
        }

        // Test and Publish JUnit Results Stage
        stage('Test and Publish JUnit Results') {
            steps {
                sh """
                    cd DEVOPS/Jenkins/dummyproject
                    mvn test
                    junit '**/target/**/*.xml'
                """
            }
        }

        // Upload to Artifactory Stage
        stage('Upload to Artifactory') {
            steps {
                sh 'jf rt upload --url http://192.168.29.163:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} /var/lib/jenkins/workspace/CI-MyProject-Maven-freestyle-Gitlab/DEVOPS/Jenkins/dummyproject/target/dummyproject-1.0-SNAPSHOT.war dummyproject/'
            }
        }

        // Login to AWS ECR Stage
        stage('Logging into AWS ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 876724398547.dkr.ecr.us-east-1.amazonaws.com'
                }
            }
        }

        // Build Docker Image Stage
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        cd DEVOPS/Jenkins/dummyproject/
                        docker build -t dummyproject:build .
                    """
                }
            }
        }

        // Tag & Push Image to AWS ECR Stage
        stage('Tagging the image & pushing it to AWS ECR') {
            steps {
                script {
                    sh """
                        docker tag dummyproject:build 876724398547.dkr.ecr.us-east-1.amazonaws.com/jenkins-pipeline:build
                        docker push 876724398547.dkr.ecr.us-east-1.amazonaws.com/jenkins-pipeline:build
                    """
                }
            }
        }

       

        // Run on EC2 Instance Stage
        stage('Run on EC2 instance') {
            steps {
                script {
                    sh """
                        ssh -i ${PEM_KEY} -o StrictHostKeyChecking=no ec2-user@54.156.170.193 \
                          'docker stop project1 || true && \
                           docker rm project1 || true && \
                           aws configure set aws_access_key_id *********** && \
                           aws configure set aws_secret_access_key ************* && \
                           aws configure set default.region us-east-1 && \
                           aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 876724398547.dkr.ecr.us-east-1.amazonaws.com  &&  \
                           docker pull 876724398547.dkr.ecr.us-east-1.amazonaws.com/jenkins-pipeline:build && \
                           docker run -d -p 8088:8080 --name project1 876724398547.dkr.ecr.us-east-1.amazonaws.com/jenkins-pipeline:build'
                    """
                }
            }
        }


    }
}
