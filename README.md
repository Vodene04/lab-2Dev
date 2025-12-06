
# Автоматизация сборки и деплоя Python Docker-приложений через Jenkins с интеграцией GitHub
***

# Ход работы

## Шаг 0. Клонирование репозитория 
Выгрузил репозиторий с исходным кодом в свой репозиторий

## Шаг 1. Создание и конфигурирование Jenkins Job

1. Авторизовался на Jenkins-сервере по указанному IP и порту.
2. В верхнем меню выбрал кнопку **New Item** для создания нового проекта.
3. Задал уникальное имя проекта — student-Kamalov.
4. В списке типов проектов выбрал **Pipeline** и нажмите **OK**.
5. В открывшемся окне перешел к разделу **Pipeline**:
   - В поле **Definition** выбрал **Pipeline script**.
   - В поле ниже вставил заготовленный Pipeline скрипт (Jenkinsfile), предоставленный преподавателем.

6. В Jenkinsfile отдельно отредактировал параметры:
   - В поле `STUDENT_NAME` указал своё имя — Artyom
   - В поле `PORT` указал уникальный порт - 8083

7. Сохранил конфигурацию проекта.

***

## Шаг 2. Структура Jenkinsfile

```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'STUDENT_NAME', defaultValue: 'Artyom', description: 'Камалов Артём')
        string(name: 'PORT', defaultValue: '8083', description: 'Порт')
    }
    
    stages {
        stage('Выгружаем код из репозитория') {
            steps {
                git 'https://github.com/Vodene04/lab-2Dev.git'
            }
        }
        stage('Собираем docker image') {
            steps {
                script {
                    dockerImage = docker.build("artyomka")
                }
            }
        }
        stage('Запускаем тесты в докере') {
            steps {
                script {
                    dockerImage.inside {
                        sh 'python -m unittest test_app.py'
                    }
                }
            }
        }
        stage('Запускаем докер контейнер') {
            steps {
                script {
                    sh "docker rm -f student-kamalovartyom-container || true"
                    sh "docker run -d --name student-kamalovartyomka-container -p ${params.PORT}:${params.PORT} -e STUDENT_NAME='${params.STUDENT_NAME}' -e PORT=${params.PORT} artyomka"
                }
            }
        }
    }
```


***

## Шаг 3. Настройка GitHub webhook для автоматического запуска сборки

1. Перешел в репозиторий на GitHub.
2. Во вкладке **Settings** выбрал пункт **Webhooks**.
3. Нажал кнопку **Add webhook**.
4. В поле **Payload URL** ввел адрес Jenkins вебхука:
   ```
   http://158.160.194.244:8080/github-webhook/
   ```
5. В поле **Content type** выбрал `application/json`.
6. В разделе **Which events would you like to trigger this webhook?** выбрал **Just the push event**.
7. Подтвердил добавление вебхука.

***

## Шаг 4. Работа с Jenkins и проверка

1. В интерфейсе Jenkins открыл созданную job.
2. Нажал кнопку **Build Now**, чтобы инициировать первую сборку.
3. Проверил, что приложение собрано
4. Открыл приложение в браузере по адресу:
   ```
  http://158.160.194.244:8083
   ```
5. Внес изменения в репозиторий и выполнил коммит в ветку `master`.


***
