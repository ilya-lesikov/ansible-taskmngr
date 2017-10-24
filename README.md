# Ansible playbooks для [CI-CD-pipeline](https://github.com/bititanb/CI-CD-pipeline)

## Краткий обзор
Ansible используется для развертывания инфраструктуры и приложения.  
Часть ролей написана с нуля, часть взята с Ansible Galaxy и адаптирована (эти имеют имя автора префиксом, напр. *geerlingguy*.filebeat).

Нужно две (виртуальные) машины, на одной будут:
* Jenkins
* Kubernetes
* Docker Registry
* Elasticsearch
* Logstash
* Kibana
* Zabbix

На второй только клиенты для некоторых сервисов:
* Kubernetes
* Filebeat
* Zabbix

## Подготовка
> Подготовку можно пропустить, если разворачивать, используя [Vagrant/Virtualbox](https://github.com/bititanb/CI-CD-pipeline/tree/master/vagrant) (рекомендуется) или [Packer/KVM](https://github.com/bititanb/CI-CD-pipeline/tree/master/packer).

### Зависимости
* Ansible 2.3.2+
* git
* sshpass

### Хосты
* Два Centos **7.2**
* Первый должен иметь минимум 3.8ГБ ОЗУ, второй — 800МБ+
* Должны быть доступны по DN taskmngr1 и taskmngr2 соответственно
* SELinux отключен

### Пользователи
* Пользователь deploy с паролем deploy
* Пользователь user1 с паролем 1
* Пользователь jenkins со сгенерированным ssh rsa ключом без пароля на него

### Сеть
* Разворачивание всех сервисов создает серьезную нагрузку на сеть, поэтому очень рекомендую bridge вместо NAT и подобного

## Развертывание

### Инфраструктура
```shell
MASTER_IP="XXX.XXX.XXX.XXX"   # IP основного хоста с 3.8ГБ ОЗУ, по умолчанию - 192.168.59.2 (Vagrant)

ansible-playbook -e kube_master_ip="${MASTER_IP}" /etc/ansible/taskmngr.yaml
```
### Приложениe

Во время развертывания инфраструктуры разворачивается и Jenkins, который вытягивает репозиторий с приложением и собирает приложение, используя роль *taskmngr-kubernetes* [(более подробно здесь)](./roles/taskmngr-kubernetes/README.md)

## Отладка

```shell
ansible-playbook -e kube_master_ip="${MASTER_IP}" \
  -vvvv                # очень подробные логи \
  -t ${SPECIFIC_TAG}   # выполнять только с тегом X; теги в taskmngr.yaml \
  --start-at-task="add something somewhere"   # начать с конкретного таска \
  /etc/ansible/taskmngr.yaml
```

```shell
# kubernetes
kubectl get all --all-namespaces
kubectl describe ...
kubectl logs ...
journalctl -u kubelet
docker ps --all
docker logs ...
docker inspect ...
```

### Дополнительные флаги
Для многих ролей доступны флаги для форсированного переразвертывания. Имеют общий префикс "*force_*". Найти можно так:
```shell
grep -R "force_" ./roles
```
Пример использования:
```shell
ansible-playbook -e force_something=true /etc/ansible/taskmngr.yaml
```
