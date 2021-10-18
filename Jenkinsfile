pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION='us-east-2'
        //THE_BUTLER_SAYS_SO = credentials('Jenkins-aws-creds')
        BUILDNAME='${JOB_NAME}-${BUILD_NUMBER}'
        BUCKETNAME='artefactrepository' //S3
        APPLICATIONNAME='projeto-python-aws'  // ElasticBeanstalk
        ENVIRONMENTNAME='projeto-python-aws-env' //ElastcBeanstalk

        FLASK_ENV = 'testing'
        FLASK_APP = 'application.py'
        DEBUG = true
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage ("Install Dependencies") {
            steps {
                sh "virtualenv venv"
                sh "source venv/bin/activate"
                sh "pip install --upgrade pip"
                sh "pip install -r requirements.txt"
                sh "deactivate"
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                source bin/activate
                nosetests app --with-xunit
                deactivate
                '''
                junit "nosetests.xml"
            }
        }
        
        stage('Empacotando') {
            steps {
                echo 'Compactando arquivo em ZIP'
                sh"""zip -r ${BUILDNAME}.zip * -x '*venv*'"""
            }
        }

        stage('Archivament S3') {
            steps {
                echo 'Deploying on S3....'
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Jenkins-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                        aws s3 ls s3://${BUCKETNAME}

                        aws s3 cp ${BUILDNAME}.zip s3://${BUCKETNAME} --region ${AWS_DEFAULT_REGION}
                    """
                }
            }
        }

        stage('create elasticbeanstalk'){
            steps{
                echo ''
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Jenkins-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws elasticbeanstalk create-application-version --application-name "${APPLICATIONNAME}" --version-label "${BUILDNAME}" --description "Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}" --source-bundle S3Bucket=${BUCKETNAME},S3Key=${BUILDNAME}.zip --region ${AWS_DEFAULT_REGION}
                    """
                }
            }
        }

        stage('update environment elasticbeanstalk'){
            steps{
                echo ''
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Jenkins-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                        aws elasticbeanstalk update-environment --environment-name "${ENVIRONMENTNAME}" --version-label "${BUILDNAME}" --region ${AWS_DEFAULT_REGION}
                    """
                }
            }
        }
        stage('remove zip e venv'){
            steps{
                sh """rm -rf ${BUILDNAME}.zip"""
                echo 'removendo o venv'
                sh 'rm -rf venv'
            }
        }

    }
}
