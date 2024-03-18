pipeline {
    agent {
        kubernetes {
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: gradle
                    image: gradle
                    command:
                    - sleep
                    args:
                    - 99d
                    volumeMounts:
                    - name: shared-storage
                      mountPath: /mnt        
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - sleep
                    args:
                    - 9999999
                    volumeMounts:
                    - name: shared-storage
                      mountPath: /mnt
                    - name: kaniko-secret
                      mountPath: /kaniko/.docker
                  restartPolicy: Never
                  volumes:
                  - name: shared-storage
                    persistentVolumeClaim:
                      claimName: jenkins-pv-claim
                  - name: kaniko-secret
                    secret:
                        secretName: dockercred
                        items:
                        - key: .dockerconfigjson
                          path: config.json
            """
        }
    }
    
    stages {
        stage('Checkout code and prepare environment') {
            steps {
                git branch: env.BRANCH_NAME, url: 'https://github.com/peterluy-UML/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
                container('gradle') {
                    sh '''
                    cd Chapter08/sample1
                    chmod +x gradlew
                    ./gradlew test
                    '''
                }
            }
        }
        stage('Run pipeline against a gradle project - main branch') {
            when {
                branch 'main'
            }
            steps {
                container('gradle') {
                    sh '''
                    cd Chapter08/sample1
                    ./gradlew jacocoTestCoverageVerification
                    '''
                }
            }
        }
        stage('Run pipeline against a gradle project - feature branch') {
            when {
                branch 'feature'
            }
            steps {
                container('gradle') {
                    sh '''
                    cd Chapter08/sample1
                    ./gradlew checkstyleMain
                    '''
                }
            }
        }
        stage('Run pipeline against a gradle project - playground branch') {
            when {
                branch 'playground'
            }
            steps {
                container('gradle') {
                    sh '''
                    cd Chapter08/sample1
                    ./gradlew checkstyleMain
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                if ( env.BRANCH_NAME == 'main' ) {
                    container('gradle') {
                        sh '''
                        echo 'Create jar file'
                        cd Chapter08/sample1
                        sed -i 's/minimum = 0.2/minimum = 0.1/' build.gradle
                        sed -i '/checkstyle {/,/}/d' build.gradle 
                        sed -i '/checkstyle/d' build.gradle 
                        cat build.gradle
                        chmod +x gradlew
                        ./gradlew build
                        mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
                        '''
                    }
                    container('kaniko') {
                        sh '''
                        echo 'Create container'
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination peterluyuml/calculator:1.0
                        '''
                    }
                } else if ( env.BRANCH_NAME == 'feature' ) {
                    container('gradle') {
                        sh '''
                        echo 'Create jar file'
                        cd Chapter08/sample1
                        sed -i 's/minimum = 0.2/minimum = 0.1/' build.gradle
                        sed -i '/checkstyle {/,/}/d' build.gradle 
                        sed -i '/checkstyle/d' build.gradle 
                        cat build.gradle
                        chmod +x gradlew
                        ./gradlew build
                        mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
                        '''
                    }
                    container('kaniko') {
                        sh '''
                        echo 'Create container'
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination peterluyuml/calculator-feature:0.1
                        '''
                    }
                }
            }
        }
    }
}
