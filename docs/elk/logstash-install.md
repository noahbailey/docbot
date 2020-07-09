# Logstash Install

References: 

* https://www.elastic.co/guide/en/logstash/current/installing-logstash.html

Prerequisite: [Configure the repository.](/elk/repository-setup)

Install the package: 

    sudo apt-get install logstash

Enable the service to run on startup: 

    sudo systemctl enable --now logstash.service

