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

        stage('Check DB Column Type (Intentional Fail Test)') {
            steps {
                echo 'Проверяем: users.created_at должен быть DATETIME'

                sh """
                CONTAINER_ID=\$(docker ps -qf "name=${STACK_NAME}_mysql")

                
                docker exec \$CONTAINER_ID mysql -u${DB_USER} -p${DB_PASS} -D ${DB_NAME} -e "
                SELECT COLUMN_NAME, DATA_TYPE 
                FROM INFORMATION_SCHEMA.COLUMNS 
                WHERE TABLE_NAME='users' AND COLUMN_NAME='created_at';
                " > column_check.txt

                echo '--- SQL RESULT ---'
                cat column_check.txt
                echo '-------------------'

                
                if grep -qi 'datetime' column_check.txt; then
                    echo 'Поле created_at имеет тип DATETIME'
                else
                    echo 'Ошибка: ожидался DATETIME, но найден другой тип'
                    exit 1
                fi
                """
            }
        }
    }

    post {
        success {
            echo 'деплой и тесст проверки завершились успешно'
        }
        failure {
            echo 'пайплайн упал'
        }
    }
}
