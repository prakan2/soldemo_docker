pipeline {
    agent any
    environment {
        //JURL = 'http://artifactory-unified.soleng-us.jfrog.team/'
        JURL = 'http://nagag-jpd1.devopsacc.team/'
        //RT_URL = 'http://artifactory-unified.soleng-us.jfrog.team/artifactory'
        RT_URL = 'http://nagag-jpd1.devopsacc.team/artifactory'
        TOKEN = credentials('nagag-jpd1')
        ARTIFACTORY_LOCAL_DEV_REPO = 'soldocker_demo_dev'
        ARTIFACTORY_DOCKER_REGISTRY = 'nagag-jpd1.devopsacc.team/soldocker_demo_dev'
        DOCKER_REPOSITORY = 'soldocker_demo_dev'
        IMAGE_NAME = 'sol_docker_demo'
        IMAGE_VERSION = '1.0.0'
        SERVER_ID = 'k8s'
        BUILD_NAME = "SolDemo_docker_maven_new"
        PATH="${PATH}:/var/jenkins_home/bin" 
        MY_BUILD_URL="http://localhost:8888/job/SolDemo_dev"
    }
    tools {
          maven "maven-3.6.3"
      //  maven 'MAVEN_TOOL'
        //jfrog 'proscli'
    }
 
    stages {
//      stage ('Install JFrgo CLI') {
//             steps {
//                 rtServer (
//                     id: 'nagag-jpd1',
//                     url: 'http://nagag-jpd1.devopsacc.team/artifactory',
//                     credentialsId: 'nagag-jpd1'
//                 )
//             }
//         }
        stage ('Config JFrgo CLI') {
            steps {
                  echo 'Done'
//                 sh 'curl -fL https://install-cli.jfrog.io | sh'
//                 sh 'jf c add ${SERVER_ID} --interactive=false --overwrite=true --access-token=${TOKEN} --url=${JURL}'
//                 sh 'jf config use ${SERVER_ID}'
            }
        }
        stage ('Ping to Artifactory') {
            steps {
                 echo 'Ok'
          //       sh 'jf rt ping'
            }
        }
        stage ('Config Maven'){
            steps {
                dir('complete'){
                    sh 'jf mvnc --repo-resolve-releases=soldocker_demo_virtual --repo-resolve-snapshots=soldocker_demo_virtual --repo-deploy-releases=soldocker_demo_virtual --repo-deploy-snapshots=soldocker_demo_virtual'
                }
            }
        }
        stage('Compile') {
            steps {
                echo 'Compiling'
                dir('complete') {
                    //sh 'jf mvnc'
                    sh 'jf mvn clean test-compile -Dcheckstyle.skip -DskipTests'
                }
            }
        }
        stage('Package') {
            steps {
                dir('complete') {
                //Before creating the docker image, we need to create the .jar file
                   // sh 'jf mvnc'
                    sh 'jf mvn package spring-boot:repackage -DskipTests -Dcheckstyle.skip'
                  //  sh "./mvnw package"
                    echo 'Create the Docker image'
                   // sh "docker build -t build_promotion ."
                    script {
                        docker.build(ARTIFACTORY_DOCKER_REGISTRY+'/'+IMAGE_NAME+':'+IMAGE_VERSION, '--build-arg JAR_FILE=target/*.jar ../.')
                    }
                }
            }
        }
        
        stage ('Push image to Artifactory') {
            steps {
                sh 'export DOCKER_OPTS+=" --insecure-registry nagag-jpd1.devopsacc.team"'
                sh 'docker login -u nishu -p nishuJFROG_01 nagag-jpd1.devopsacc.team'
                sh 'docker push ${ARTIFACTORY_DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}'
              //  sh 'jf rt docker-push ${ARTIFACTORY_DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION} ${DOCKER_REPOSITORY} --build-name="${BUILD_NAME}" --build-number=${BUILD_ID} --url ${RT_URL} --access-token ${TOKEN}'
            }
        }
      
        stage ('Publish build info') {
            steps {
                // Collect environment variables for the build
                sh 'jf rt bce "${BUILD_NAME}" ${BUILD_ID}'
                //Collect VCS details from git and add them to the build
                sh 'jf rt bag "${BUILD_NAME}" ${BUILD_ID}'
                //Publish build info
                sh 'jf rt bp "${BUILD_NAME}" ${BUILD_ID} --build-url=${MY_BUILD_URL}/${BUILD_ID} --insecure-tls=true'
                //Promote the build
                sh 'jf rt bpr --status=Development --props="status=Development" "${BUILD_NAME}" ${BUILD_ID} ${ARTIFACTORY_LOCAL_DEV_REPO}'
            }
        }
    }
}
