# Лабораторная работа №3 со звёздочкой

## Условие:
Сделать красиво работу с секретами. Например, поднять Hashicorp Vault (или другую секретохранилку) и сделать так, чтобы ci/cd пайплайн (или любой другой ваш сервис) ходил туда, брал секрет, использовал его не светя в логах. В Readme аргументировать почему ваш способ красивый, а также описать, почему хранение секретов в CI/CD переменных репозитория не является хорошей практикой.

## Решение:

Сначала нужно определиться, каким будем пользоваться сервисом для хранения секретов. Из предлагаемых показалось, что с помощью Doppler будет сделать работу проще всего. Сперва нужно зарегистрироваться на сервисе, далее создать свой проект:

![image](https://github.com/user-attachments/assets/5dd81e17-a1b2-4eb6-b841-249c348ad56b)

Для него было выбрано название ci-cd-study. На следующем шаге необходимо создать секреты: 

![image](https://github.com/user-attachments/assets/81453c88-0a9c-4c8a-840e-64744702291f)

Далее нужно сгенерировать токен для доступа к сервису:

![image](https://github.com/user-attachments/assets/6da83dad-8c78-4a22-816b-7f091e9e6a44)

Назван он был DOPPLER_TOKEN, по-другому называть его нельзя* (было проверено), сам код был скопирован. Переходим в GitHub, в своём ренпозитории в разделе Settings -> Secrets and variables -> Actions создаём секрет DOPPLER_TOKEN, далее добавляем в него код, который ранее скопировали. 

Теперь надо написать воркфлоу:
```
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Install Doppler CLI
        uses: dopplerhq/cli-action@v1

      - name: Fetch secrets using Doppler
        env:
          DOPPLER_TOKEN: ${{ secrets.DOPPLER_TOKEN }}
        run: |
          doppler run -- echo "Здесь могла бы быть ваша реклама" && \
          doppler run -- printenv | grep -E "BOBRINA|ANIRBOB"
```

Здесь нужно обратить внимание на исползование Doppler CLI, это нужно для получения секретов, аутентификацию в Doppler, устанавливается переменная окружения DOPPLER_TOKEN, в неё передаётся код секрета, который ранее был скопирован и вставлен. Далее в разделе использование секретов они из Doppler автоматически экспортированы как переменные окружения, ну и выводит имена секретов с их значениями 

Почему данный способ красивый? Сам по себе сервис достаточно удобный, позволяет управление из терминала, а работа с секретами относительно проста. 

Хранение секретов в CI/CD переменных репозитория не является хорошей практикой, потому что у GitHub возможно утечка информации, а также возникнут проблемы с доступом к ним, разрешениями и т.п..

Лог выполнения:

https://github.com/Bobr2005/testWorkflows/actions/runs/12395979142/job/34602941085
