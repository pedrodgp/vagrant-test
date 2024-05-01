Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.ssh.insert_key = false

  # This provision is common for both VMs
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -y
    sudo apt-get install -y iputils-ping avahi-daemon libnss-mdns unzip \
        openjdk-17-jdk-headless
  SHELL

  # Configurations specific to the database VM
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/bionic64"
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "192.168.56.11"

    # We want to access H2 console from the host using port 8082
    # We want to connet to the H2 server using port 9092
    db.vm.network "forwarded_port", guest: 8082, host: 8082
    db.vm.network "forwarded_port", guest: 9092, host: 9092

    # We need to download H2
    db.vm.provision "shell", inline: <<-SHELL
      wget https://repo1.maven.org/maven2/com/h2database/h2/2.2.224/h2-2.2.224.jar 
    SHELL

    # The following provision shell will run ALWAYS so that we can execute the H2 server process
    # This could be done in a different way, for instance, setting H2 as as service
    #
    # To connect to H2 use: jdbc:h2:tcp://192.168.33.11:9092/./jpadb
    db.vm.provision "shell", :run => 'always', inline: <<-SHELL
      java -cp ./h2*.jar org.h2.tools.Server -web -webAllowOthers -tcp -tcpAllowOthers -ifNotExists > ~/out.txt &
    SHELL

    # Rename the VirtualBox VM
    db.vm.provider "virtualbox" do |v|
      v.name = "db_h2_database"
    end
  end

  #============
  # Configurations specific to the webserver VM
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/bionic64"
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.56.10"

    # We set more ram memmory for this VM
    web.vm.provider "virtualbox" do |v|
      v.memory = 1024
    end

    # We want to access tomcat from the host using port 8080
    web.vm.network "forwarded_port", guest: 8080, host: 8080

    # We need to download Tomcat10
    web.vm.provision "shell", inline: <<-SHELL
      wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.23/bin/apache-tomcat-10.1.23.tar.gz
      sudo tar xzvf apache-tomcat-10*tar.gz -C .
      sudo chown -R vagrant:vagrant apache-tomcat-10*
      sudo chmod -R u+x apache-tomcat-10* 
    SHELL

    # The following provision shell will run ALWAYS so that we can execute the Tomcat10 server process
    # This could be done in a different way, for instance, setting Tomcat10 as as service
    #
    web.vm.provision "shell", :run => 'always', inline: <<-SHELL
      ./apache-tomcat-10*/bin/startup.sh
    SHELL
    
    # Clone the repository and build the project
    web.vm.provision "shell", inline: <<-SHELL, privileged: false
      git clone https://github.com/pedrodgp/vagrant-test.git
      cd vagrant-test/CA3/part2/react-and-spring-data-rest-basic
      chmod u+x gradlew
      ./gradlew clean build

      # To deploy the war file to tomcat10 do the following command:
      sudo cp ./build/libs/react-and-spring-data-rest-basic-0.0.1-SNAPSHOT.war ~/apache-tomcat-10*/webapps/
    SHELL

    # Rename the VirtualBox VM
    web.vm.provider "virtualbox" do |v|
      v.name = "web_tomcat10_server"
    end
  end
end
