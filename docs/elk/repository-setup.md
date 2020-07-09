# ElasticSearch Repository

References: 

* https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html


## Repository

Import the GPG key for all repos. 

    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

Generally installed on newer Ubuntu/Debian systems, but install it anyways. 

    sudo apt-get install apt-transport-https


For the proprietary version (including XPACK)

    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

Or the Apache 2.0 licenced OSS version: 

    echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

