# -*- mode: ruby -*-
# vi: set ft=ruby :

# Конфигурационные переменные для легкой настройки
VM_NAME = "java-dogs-app"
VM_HOSTNAME = "dogs-api-server"
VM_IP = "192.168.100.100"
VM_MEMORY = 4096
VM_CPUS = 2
HOST_PORT = 8080
GUEST_PORT = 8080

# Скрипт для установки зависимостей с проверкой ошибок
install_deps = <<-SCRIPT
  # Логирование всех команд для отладки
  set -e
  echo "=== Начинаем установку зависимостей ==="
  
  # Обновляем пакеты с retry механизмом
  for i in {1..3}; do
    if apt-get update; then
      break
    else
      echo "Попытка $i не удалась, повторяем через 5 сек..."
      sleep 5
    fi
  done
  
  # Проверяем доступность пакета перед установкой
  if apt-cache show openjdk-17-jdk >/dev/null 2>&1; then
    echo "=== Устанавливаем OpenJDK 17 ==="
    DEBIAN_FRONTEND=noninteractive apt-get install -y openjdk-17-jdk
    
    # Проверяем успешность установки
    if java -version 2>&1 | grep -q "17"; then
      echo "✅ Java 17 успешно установлен"
      java -version
    else
      echo "❌ Ошибка установки Java"
      exit 1
    fi
  else
    echo "❌ Пакет openjdk-17-jdk недоступен"
    exit 1
  fi
  
  echo "=== Установка зависимостей завершена ==="
SCRIPT

# Скрипт для запуска приложения с проверками
start_app = <<-SCRIPT
  set -e
  echo "=== Запускаем Java приложение ==="
  
  cd /vagrant
  
  # Проверяем наличие gradlew
  if [ ! -f "./gradlew" ]; then
    echo "❌ gradlew не найден"
    exit 1
  fi
  
  # Делаем gradlew исполняемым
  chmod +x ./gradlew
  
  # Проверяем доступность порта
  if lsof -Pi :#{GUEST_PORT} -sTCP:LISTEN -t >/dev/null; then
    echo "⚠️  Порт #{GUEST_PORT} уже занят, останавливаем процесс"
    pkill -f "gradle.*bootRun" || true
    sleep 3
  fi
  
  # Запускаем приложение в фоне с логированием
  echo "🚀 Запускаем Spring Boot приложение..."
  nohup ./gradlew bootRun > /var/log/spring-boot.log 2>&1 &
  
  # Ждем запуска приложения
  echo "⏳ Ожидаем запуска приложения..."
  for i in {1..30}; do
    if curl -s http://localhost:#{GUEST_PORT}/dogs >/dev/null 2>&1; then
      echo "✅ Приложение успешно запущено!"
      echo "🔗 Доступно по адресу: http://#{VM_IP}:#{GUEST_PORT}/dogs"
      exit 0
    fi
    sleep 2
  done
  
  echo "❌ Приложение не удалось запустить за 60 секунд"
  echo "📋 Последние логи:"
  tail -20 /var/log/spring-boot.log
  exit 1
SCRIPT

# Основная конфигурация Vagrant
Vagrant.configure("2") do |config|
  
  # ===== БАЗОВАЯ КОНФИГУРАЦИЯ =====
  
  # Базовый образ - Ubuntu 22.04 LTS (стабильная версия)
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = true
  
  # Настройка hostname для удобства идентификации
  config.vm.hostname = VM_HOSTNAME
  
  # ===== СЕТЕВАЯ КОНФИГУРАЦИЯ =====
  
  # Частная сеть с фиксированным IP для прямого доступа
  config.vm.network "private_network", 
    ip: VM_IP,
    netmask: "255.255.255.0"
  
  # Port forwarding для доступа с хоста
  config.vm.network "forwarded_port", 
    guest: GUEST_PORT, 
    host: HOST_PORT,
    auto_correct: true,
    protocol: "tcp"
  
  # ===== SSH КОНФИГУРАЦИЯ =====
  
  # Оптимизация SSH подключений
  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true
  config.ssh.keep_alive = true
  
  # ===== КОНФИГУРАЦИЯ VIRTUALBOX =====
  
  config.vm.provider "virtualbox" do |vb|
    # Название VM в VirtualBox
    vb.name = VM_NAME
    
    # Ресурсы виртуальной машины
    vb.memory = VM_MEMORY
    vb.cpus = VM_CPUS
    
    # Оптимизация производительности
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
    vb.customize ["modifyvm", :id, "--audio", "none"]
    vb.customize ["modifyvm", :id, "--usb", "off"]
    vb.customize ["modifyvm", :id, "--usbehci", "off"]
    
    # Улучшение графики (если нужен GUI)
    vb.gui = false
    vb.customize ["modifyvm", :id, "--vram", "16"]
    vb.customize ["modifyvm", :id, "--accelerate3d", "off"]
  end
  
  # ===== КОНФИГУРАЦИЯ ДЛЯ ДРУГИХ ПРОВАЙДЕРОВ =====
  
  # Поддержка VMware (если установлен)
  config.vm.provider "vmware_desktop" do |vmware|
    vmware.vmx["memsize"] = VM_MEMORY
    vmware.vmx["numvcpus"] = VM_CPUS
    vmware.vmx["displayName"] = VM_NAME
  end
  
  # Поддержка Hyper-V (для Windows)
  config.vm.provider "hyperv" do |hv|
    hv.memory = VM_MEMORY
    hv.cpus = VM_CPUS
    hv.vmname = VM_NAME
  end
  
  # ===== СИНХРОНИЗАЦИЯ ПАПОК =====
  
  # Оптимизированная синхронизация с исключениями
  config.vm.synced_folder ".", "/vagrant",
    type: "virtualbox",
    owner: "vagrant",
    group: "vagrant",
    mount_options: ["dmode=755", "fmode=644"]
  
  # Исключаем ненужные файлы из синхронизации
  config.vm.synced_folder ".", "/vagrant",
    rsync__exclude: [".git/", ".vagrant/", "build/", "*.log"]
  
  # ===== PROVISIONING =====
  
  # Первичная настройка системы
  config.vm.provision "shell", 
    name: "install-dependencies",
    inline: install_deps,
    privileged: true
  
  # ===== ТРИГГЕРЫ =====
  
  # Автоматический запуск приложения после up
  config.trigger.after :up do |trigger|
    trigger.name = "Start Java Application"
    trigger.info = "🚀 Запускаем Spring Boot приложение..."
    trigger.run_remote = {
      inline: start_app,
      privileged: true
    }
  end
  
  # Информационное сообщение при завершении
  config.trigger.after :up do |trigger|
    trigger.name = "Setup Complete"
    trigger.info = <<~EOF
      
      🎉 ===== УСТАНОВКА ЗАВЕРШЕНА! =====
      
      📱 Приложение доступно по адресам:
         • http://#{VM_IP}:#{GUEST_PORT}/dogs
         • http://localhost:#{HOST_PORT}/dogs
      
      🔧 Управление VM:
         • vagrant ssh     - подключиться к VM
         • vagrant halt    - остановить VM
         • vagrant reload  - перезапустить VM
         • vagrant destroy - удалить VM
      
      📋 Логи приложения: /var/log/spring-boot.log
      
      =======================================
    EOF
  end
  
  # Остановка приложения при halt
  config.trigger.before :halt do |trigger|
    trigger.name = "Stop Java Application"
    trigger.info = "🛑 Останавливаем приложение..."
    trigger.run_remote = {
      inline: "pkill -f 'gradle.*bootRun' || true",
      privileged: true
    }
  end
  
end
