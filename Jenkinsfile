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
                    docker login -u $DOCKER_USER -p$DOCKER_PASS
                    docker stack deploy -c ${COMPOSE_FILE} ${STACK_NAME}
                    """
                }
            }
        }

        stage('Wait for MySQL') {
            steps {
                echo 'Ждём пока MySQL станет доступен...'
                sh """
                CONTAINER_ID=\$(docker ps -qf "name=${STACK_NAME}_mysql")
                until docker exec \$CONTAINER_ID mysqladmin ping -u${DB_USER} -p${DB_PASS} --silent; do
                  echo "MySQL ещё не готов, ждём 5 секунд..."
                  sleep 5
                done
                echo "MySQL готов к подключениям"
                """
            }
        }

        stage('Check DB Columns') {
            steps {
                echo 'Проверка существования столбца name в таблице users...'
                sh """
                CONTAINER_ID=\$(docker ps -qf "name=${STACK_NAME}_mysql")

                # Проверка наличия столбца name
                docker exec \$CONTAINER_ID mysql -u${DB_USER} -p${DB_PASS} -D ${DB_NAME} -e "
                SELECT COLUMN_NAME
                FROM INFORMATION_SCHEMA.COLUMNS
                WHERE TABLE_NAME='users';
                " > columns_check.txt
                cat columns_check.txt
                if ! grep -qw 'name' columns_check.txt; then
                    echo 'Ошибка: поле name отсутствует в таблице users!'
                    exit 1
                else
                    echo 'Поле name присутствует в таблице users.'
                fi
                """
            }
        }

        stage('Dump users Table') {
            steps {
                echo 'Создаём дамп таблицы users...'
                sh """
                CONTAINER_ID=\$(docker ps -qf "name=${STACK_NAME}_mysql")
                docker exec \$CONTAINER_ID mysqldump -u${DB_USER} -p${DB_PASS} ${DB_NAME} users > users_table.sql
                echo "Дамп users_table.sql создан на хосте Jenkins"
                """
            }
        }
    }

    post {
        success {
            echo 'Пайплайн успешно выполнен'
        }
        failure {
            echo 'Пайплайн завершился с ошибкой'
        }
    }
}
