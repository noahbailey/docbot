# ElasticSearch Install

References: 

* https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

Prerequisite: [Configure the repository.](/elk/repository-setup)

## Install Package

    sudo apt-get update && sudo apt-get install elasticsearch

## Enable service

    sudo systemctl enable --now elasticsearch.service

