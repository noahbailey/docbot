# Install Kibana

References: 

* https://www.elastic.co/guide/en/kibana/current/deb.html

Prerequisite: [Configure the repository.](/elk/repository-setup)

Install the package: 

    sudo apt-get install kibana

Enable the service to run on boot: 

    sudo systemctl enable --now kibana.service


