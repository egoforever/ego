pipeline {
    agent any

    environment {
        STACK_NAME = "my_stack"
        COMPOSE_FILE = "docker-compose.yml"
        DB_CONTAINER = "mysql"         
        DB_USER = "root"
        DB_PASS = "password"
        DB_NAME = "website_db"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/egoforever/ego.git'
            }
        }

        stage('Deploy to Swarm') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker stack deploy -c ${COMPOSE_FILE} ${STACK_NAME}
                    """
                }
            }
        }

        stage('Check DB Column Type') {
            steps {
                echo 'проверка users.created_at тип TIMESTAMP...'

                sh """
                CONTAINER_ID=\$(docker ps -qf "name=${STACK_NAME}_mysql")
                docker exec \$CONTAINER_ID mysql -u${DB_USER} -p${DB_PASS} -D ${DB_NAME} -e "
                SELECT COLUMN_NAME, DATA_TYPE 
                FROM INFORMATION_SCHEMA.COLUMNS 
                WHERE TABLE_NAME='users' AND COLUMN_NAME='created_at';
                " > column_check.txt

                cat column_check.txt

                
                if ! grep -qi 'timestamp' column_check.txt; then
                    echo 'ошибка поле created_at не TIMESTAMP!'
                    exit 1
                else
                    echo 'поле created_at имеет тип TIMESTAMP.'
                fi
                """
            }
        }
    }

    post {
        success {
            echo 'пайплайн успешно'
        }
        failure {
            echo 'пайплайн ошибка'
        }
    }
}
