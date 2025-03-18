# Vagrant

Create a `Vagrantfile` that automates the creation and configuration of a `VirtualBox` virtual machine to run a Java application
* Use the `ubuntu/jammy64` box to provision a `VirtualBox` VM.
* Configure private networking with IP address `192.168.100.100`.
* Allocate at least `2` CPUs and `4096` MB of memory to the VM.
* Write an inline shell script in the `install_deps` variable to install the `openjdk-17-jdk` APT package Write an inline shell script in the `install_deps` variable.
* Use the `shell` provisioner to execute this script.
* Configure a Vagrant trigger to execute after vagrant up, which changes the directory to `/vagrant` and runs `./gradlew bootRun`.
Once everything is set up, verify that you can see a JSON list of dogs by navigating to http://192.168.100.100:8080/dogs in your web browser.
