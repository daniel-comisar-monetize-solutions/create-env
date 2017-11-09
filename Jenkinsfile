pipeline {
    agent any

    parameters {
        string(
            name: 'ENV_NAME'
        )
    }

    stages {
        stage("validate env name") {
            steps {
                script {
                    if (!params.ENV_NAME) {
                        echo "ENV_NAME cannot be empty"
                        sh "exit 1"
                    }
                }
            }
        }

        stage("provision web server") {
            steps {
                script {
                    instance_id = sh(returnStdout: true, script: "aws ec2 describe-instances --filters '[{\"Name\":\"tag:Name\",\"Values\":[\"${params.ENV_NAME}\"]},{\"Name\":\"instance-state-name\",\"Values\":[\"running\"]}]' | jq .Reservations[0].Instances[0].InstanceId").trim()
                    if (instance_id == "null") {
                        instance_id = sh(returnStdout: true, script: "aws ec2 run-instances --image-id ami-6e1a0117 --instance-type t2.micro --tag-specifications 'ResourceType=instance,Tags=[{Key=\"Name\",Value=\"${params.ENV_NAME}\"}]' | jq .Instances[0].InstanceId").trim()
                        sh("aws ec2 wait instance-running --instance-id ${instance_id}")
                    }
                }
            }
        }

        stage("fork database") {
            steps {
                script {
                    snapshot = sh(returnStdout: true, script: "date '+inpharmics-monetize-%Y-%m-%d-%H-%M'").trim()
                    sh("aws rds create-db-snapshot --db-instance-identifier inpharmics-monetize --db-snapshot-identifier ${snapshot}")
                    sh("aws rds wait db-snapshot-completed --db-snapshot-identifier ${snapshot}")
                    sh("aws rds restore-db-instance-from-db-snapshot --db-instance-identifier ${params.ENV_NAME} --db-snapshot-identifier ${snapshot}")
                    sh("aws rds wait db-instance-available --db-instance-identifier ${params.ENV_NAME}")
                }
            }
        }

        /*stage("deploy") {
            steps {
            }
        }*/
    }
}
