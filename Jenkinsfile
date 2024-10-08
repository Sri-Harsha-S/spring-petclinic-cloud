
pipeline {
    agent any

    environment {
        AWS_REGION = "eu-west-3"
        AWS_ACCESS_KEY_ID = credentials('ACCESS_KEY_AWS')
        AWS_SECRET_ACCESS_KEY = credentials('SECRET_KEY_AWS')
        ECR_REGISTRY = "565393062704.dkr.ecr.eu-west-3.amazonaws.com"
    }
    stages {
        stage("Git Checkout") {
            steps {
                git(
                    url: "https://github.com/Sri-Harsha-S/spring-petclinic-cloud",
                    branch: "master",
                    credentialsId: 'GITHUB_TOKEN'
                )
            }
        }
        stage("Setup Env Variable") {
            steps {
                script {
                    env.REPOSITORY_PREFIX = "${ECR_REGISTRY}/petclinic"
                    echo "Repository Prefix set to: ${env.REPOSITORY_PREFIX}"
                }
            }
        }
        stage("Login to ECR") {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }
        stage("Build and Tag Images") {
            steps {
                withMaven() {
                    sh '''
                        docker --version

                        # Build images
                        mvn -X spring-boot:build-image -Pk8s -DREPOSITORY_PREFIX=$ECR_REGISTRY/petclinic

                        # Tag the images with simpler names
                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-api-gateway:latest $REPOSITORY_PREFIX/api-gateway:latest
                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-api-gateway:latest $REPOSITORY_PREFIX/api-gateway:1.0.$BUILD_NUMBER

                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-customers-service:latest $REPOSITORY_PREFIX/customers-service:latest
                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-customers-service:latest $REPOSITORY_PREFIX/customers-service:1.0.$BUILD_NUMBER

                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-vets-service:latest $REPOSITORY_PREFIX/vets-service:latest
                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-vets-service:latest $REPOSITORY_PREFIX/vets-service:1.0.$BUILD_NUMBER

                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-visits-service:latest $REPOSITORY_PREFIX/visits-service:latest
                        docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-visits-service:latest $REPOSITORY_PREFIX/visits-service:1.0.$BUILD_NUMBER
                    '''
                }
            }
        }
        // New stage to push images
        stage("Push Images to ECR") {
            steps {
                script {
                    sh '''
                        # Push the images to ECR
                        docker push $REPOSITORY_PREFIX/api-gateway:latest
                        docker push $REPOSITORY_PREFIX/api-gateway:1.0.$BUILD_NUMBER

                        docker push $REPOSITORY_PREFIX/customers-service:latest
                        docker push $REPOSITORY_PREFIX/customers-service:1.0.$BUILD_NUMBER

                        docker push $REPOSITORY_PREFIX/vets-service:latest
                        docker push $REPOSITORY_PREFIX/vets-service:1.0.$BUILD_NUMBER

                        docker push $REPOSITORY_PREFIX/visits-service:latest
                        docker push $REPOSITORY_PREFIX/visits-service:1.0.$BUILD_NUMBER
                    '''
                }
            }
        }
        stage('Trigger ManifestUpdate') {
            steps {
                script {
                    echo "Triggering the Update Manifest Job"
                    build job: 'helm-chart-CI', parameters: [string(name: 'IMAGEVERSION', value: "1.0.${env.BUILD_NUMBER}")]
                }
            }
        }
    }
}
// pipeline {
//     agent any

//     stages {
//         stage('Checkout') {
//             steps {
//                 script {
//                     // Define your Git repository URL
//                     def gitRepo = 'https://github.com/Sri-Harsha-S/spring-petclinic-cloud.git'

//                     // Define your credentials ID (the ID of the Git token stored in Jenkins)
//                     def credentialsId = 'GITHUB_TOKEN'

//                     // Checkout the repository
//                     try {
//                         checkout([$class: 'GitSCM', 
//                             branches: [[name: '*/master']], // Change this to your default branch if different
//                             userRemoteConfigs: [[
//                                 url: gitRepo, 
//                                 credentialsId: credentialsId
//                             ]]
//                         ])
//                         // Print success message
//                         echo 'Successfully checked out'
//                     } catch (Exception e) {
//                         echo "Checkout failed: ${e.getMessage()}"
//                         currentBuild.result = 'FAILURE'
//                     }
//                 }
//             }
//         }
//     }
// }

// pipeline {
//     agent any

//     environment {
//         AWS_REGION = "eu-west-3"
//         AWS_ACCESS_KEY_ID = credentials('ACCESS_KEY_AWS')
//         AWS_SECRET_ACCESS_KEY = credentials('SECRET_KEY_AWS')
//         ECR_REGISTRY = "565393062704.dkr.ecr.eu-west-3.amazonaws.com"
//     }
//     stages {
//         stage("Git Checkout") {
//             steps {
//                 git(
//                     url: "https://github.com/Sri-Harsha-S/spring-petclinic-cloud",
//                     branch: "master",
//                     credentialsId: 'GITHUB_TOKEN' // Removed the $ sign for credentialsId
//                 )
//             }
//         }
//         stage("Setup Env Variable") {
//             steps {
//                 script {
//                     env.REPOSITORY_PREFIX = "${ECR_REGISTRY}/petclinic"
//                     echo "Repository Prefix set to: ${env.REPOSITORY_PREFIX}"
//                 }
//             }
//         }
//         stage("Login to ECR") {
//             steps {
//                 script {
//                     sh '''
//                         aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
//                     '''
//                 }
//             }
//         }
//         stage("Build and Tag Images") {
//             steps {
//                 withMaven() {
//                     sh '''
//                         docker --version

//                         # Build images
//                         mvn -X spring-boot:build-image -Pk8s -DREPOSITORY_PREFIX=$ECR_REGISTRY/petclinic

//                         # Tag the images with simpler names
//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-api-gateway:latest $REPOSITORY_PREFIX/api-gateway:latest
//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-api-gateway:latest $REPOSITORY_PREFIX/api-gateway:1.0.$BUILD_NUMBER

//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-customers-service:latest $REPOSITORY_PREFIX/customers-service:latest
//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-customers-service:latest $REPOSITORY_PREFIX/customers-service:1.0.$BUILD_NUMBER

//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-vets-service:latest $REPOSITORY_PREFIX/vets-service:latest
//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-vets-service:latest $REPOSITORY_PREFIX/vets-service:1.0.$BUILD_NUMBER

//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-visits-service:latest $REPOSITORY_PREFIX/visits-service:latest
//                         docker tag $ECR_REGISTRY/petclinic/spring-petclinic-cloud-visits-service:latest $REPOSITORY_PREFIX/visits-service:1.0.$BUILD_NUMBER
//                     '''
//                 }
//             }
//         }
//     }
// }
