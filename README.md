# Домашнее задание к занятию "09.04 Jenkins"

## Подготовка к выполнению

1. Создал 2 VM в **cloud.yandex** с помощью **terraform** с параметрами согласно заданию.
2. Установил **Jenkins** при помощи `playbook`. Всё завершилось успешно:
```
PLAY RECAP **********************************************************************************************
jenkins-agent-01           : ok=17   changed=14   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
jenkins-master-01          : ok=11   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
3. Произвёл первоначальную настройку **Jenkins**.
- Создал нового администратора
- Удалил исполнителей на мастере
- Подключил нового агента
- Настроил агента как в лекции c labels = centos

## Основная часть

1. Сделал *Freestyle Job*, который запускает `molecule test` из репозитория с ролью Vector.
- Для этого задания сделал отдельный [репозиторый с ролью Vector](https://github.com/andrey-mishin/ansible-vector-role-molecule "Репозиторий с vector-role").
- Настроил *job* как в лекции
- Итоговый вариант *job*:
```
pip3 install --user "molecule==3.3.0" "molecule-docker"
molecule --version
cd roles/vector-role/
molecule test
``` 
- Проверил работу:
```
18:46:19 PLAY RECAP *********************************************************************
18:46:19 localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
18:46:19
18:46:20 INFO     Pruning extra files from scenario ephemeral directory
18:46:20 Finished: SUCCESS
```

2. Сделал *Declarative Pipeline Job*, который запускает `molecule test` из моего [репозитория с ролью Vector](https://github.com/andrey-mishin/ansible-vector-role-molecule "Репозиторий vector-role").
- Итоговый вариант *job*:
```
pipeline {
    agent any
        stages {
            stage('download repo') {
                steps {
                    git 'https://github.com/andrey-mishin/ansible-vector-role-molecule'
                }
            }
            stage('test') {
                steps {
                    dir('./roles/vector-role/') {
                        sh 'molecule test'
                    }
                }
            }
        }
}
```
- Проверил работу:
```
PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
[Pipeline] }
[Pipeline] // dir
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

3. Перенёс *Declarative Pipeline* в мой репозиторий в файл [Jenkinsfile](https://github.com/andrey-mishin/ansible-vector-role-molecule/Jenkinsfile "Ссылка на Jenkinsfile").
4. Создал *Multibranch Pipeline* на запуск `Jenkinsfile` из моего репозитория.
- Сделал отдельную ветку `pipeline`, в которой добавил `Jenkinsfile`
- Проверил работу:
```
Checking branches...
  Checking branch pipeline
      ‘Jenkinsfile’ found
    Met criteria
No changes detected: pipeline (still at 1144ca3c10a63baa39a706ae851cc33b3bf07629)
  Checking branch master
      ‘Jenkinsfile’ not found
    Does not meet criteria
Processed 2 branches
[Thu Sep 22 06:17:24 UTC 2022] Finished branch indexing. Indexing took 1.6 sec
Finished: SUCCESS
```

5. Создал *Scripted Pipeline*, наполнил его скриптом из [pipeline](https://github.com/netology-code/mnt-homeworks/tree/MNT-13/09-ci-04-jenkins/pipeline "Ссылка на Jenkinsfile из ДЗ")
6. Внёс необходимые изменения, чтобы *Pipeline* запускал `ansible-playbook` без флагов `--check --diff`, если не установлен параметр при запуске джобы (prod_run = True). По умолчанию параметр имеет значение False и запускает прогон с флагами `--check --diff`.
- Итоговый вариант job:
```
node("centos"){
    stage("Git checkout"){
        git 'https://github.com/aragastmatb/example-playbook.git'
    }
    stage("Sample define secret_check"){
        prod_run=false
    }
    stage("Run playbook"){
        if (prod_run){
            sh 'ansible-playbook site.yml -i inventory/prod.yml'
        }
        else{
            sh 'ansible-playbook site.yml -i inventory/prod.yml --check --diff'
        }

    }
}
```
7. Проверил работоспособность, исправил ошибки, исправленный *Pipeline* вложил в [репозиторий](https://github.com/andrey-mishin/ansible-vector-role-molecule "Репозиторий") в файл [ScriptedJenkinsfile](https://github.com/andrey-mishin/ansible-vector-role-molecule/ScriptedJenkinsfile "Ссылка на ScriptedJenkinsfile").
- Если **prod_run = true**, то `ansible-playbook` запускается **без флагов** `--check --diff`
- Если **prod_run = false**, то `ansible-playbook` запускается **с флагами** `--check --diff`
- Всё отрабатывает корректно
