pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = ("AKIAYRCOPKCSAJVN4GQB")
        AWS_SECRET_ACCESS_KEY = ("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")
    }
    stages {
        stage("Build") {
            environment {
                DB_HOST = ("laravel.cwb1jzpmjmgj.ap-south-1.rds.amazonaws.com")
                DB_DATABASE = ("laravel")
                DB_USERNAME = ("admin")
                DB_PASSWORD = ("12345678")
            }
            steps {
                sh 'php --version'
                sh 'composer install'
                sh 'composer --version'
                sh 'cp .env.example .env'
                sh 'echo DB_HOST=${DB_HOST} >> .env'
                sh 'echo DB_USERNAME=${DB_USERNAME} >> .env'
                sh 'echo DB_DATABASE=${DB_DATABASE} >> .env'
                sh 'echo DB_PASSWORD=${DB_PASSWORD} >> .env'
                sh 'php artisan key:generate'
                sh 'cp .env .env.testing'
                sh 'php artisan migrate'
            }
        }
        stage("Unit test") {
            steps {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            steps {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            steps {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            steps {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            steps {
                sh "docker build -t danielgara/laravel8cdpart2 --no-cache ."
            }
        }
        stage("Docker push") {
            environment {
                DOCKER_USERNAME = ("vrpawar86")
                DOCKER_PASSWORD = ("1211122111Vi")
            }
            steps {
                sh "docker login --username ${DOCKER_USERNAME} --password ${DOCKER_PASSWORD}"
                sh "docker push vrpawar86/laravel8cdpart2"
            }
        }
        stage("Deploy to staging") {
            steps {
                sh "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
                sh "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}"
                sh "ssh-agent sh -c 'ssh-add /etc/ansible/pem/key.pem && ansible-playbook /etc/ansible/playbook/playbook-staging-run.yml'"
            }
        }
        stage("Acceptance test curl") {
            steps {
                sleep 20
                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
            }
        }
        stage("Acceptance test codeception") {
            steps {
                sh "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
                sh "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}"
                sh "ssh-agent sh -c 'ssh-add /etc/ansible/pem/key.pem && ansible-playbook /etc/ansible/playbook/playbook-staging-acceptance.yml'"
            }
            post {
                always {
                    sh "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
                    sh "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}"
                    sh "ssh-agent sh -c 'ssh-add /etc/ansible/pem/key.pem && ansible-playbook /etc/ansible/playbook/playbook-staging-stop.yml'"
                }
            }
        }
        stage("Release") {
            steps {
                sh "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
                sh "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}"
                sh "ssh-agent sh -c 'ssh-add /etc/ansible/pem/key.pem && ansible-playbook /etc/ansible/playbook/playbook-production-run.yml'"
            }
        }
        stage("Smoke test") {
            steps {
                sleep 20
                sh "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
                sh "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}"
                sh "ssh-agent sh -c 'ssh-add /etc/ansible/pem/key.pem && ansible-playbook /etc/ansible/playbook/playbook-production-acceptance.yml'"
            }
        }
    }
}
