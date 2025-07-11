# -*- mode: ruby -*-
# vi: set ft=ruby :

# Скрипт для установки зависимостей Java
# Этот скрипт будет выполнен во время провижинга VM
install_deps = <<-SCRIPT
  # Обновляем список пакетов системы
  apt-get update
  # Устанавливаем OpenJDK 17 для запуска Java приложений
  apt-get install -y openjdk-17-jdk
SCRIPT

# Конфигурация Vagrant версии 2
Vagrant.configure("2") do |config|
  
  # Базовый образ виртуальной машины - Ubuntu 22.04 LTS
  config.vm.box = "ubuntu/jammy64"
  
  # Настройка частной сети с фиксированным IP адресом
  # Позволяет обращаться к VM по адресу 192.168.100.100
  config.vm.network "private_network", ip: "192.168.100.100"
  
  # Проброс портов: порт 8080 в VM доступен как 8080 на хосте
  # Это позволяет обращаться к приложению через localhost:8080
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  
  # Настройка параметров VirtualBox провайдера
  config.vm.provider "virtualbox" do |vb|
    # Выделяем 4096 МБ (4 ГБ) оперативной памяти
    vb.memory = "4096"
    # Выделяем 2 процессорных ядра для улучшения производительности
    vb.cpus = 2
  end
  
  # Провижинг через shell скрипт
  # Выполняется один раз при первом запуске VM
  config.vm.provision "shell", inline: install_deps
  
  # Триггер, выполняющийся после успешного запуска VM
  # Автоматически запускает Java приложение
  config.trigger.after :up do |trigger|
    trigger.info = "Запуск Java приложения..."
    # Переходим в директорию /vagrant (синхронизированная с хостом)
    # и запускаем Spring Boot приложение через Gradle
    trigger.run_remote = {inline: "cd /vagrant && ./gradlew bootRun"}
  end
  
end
