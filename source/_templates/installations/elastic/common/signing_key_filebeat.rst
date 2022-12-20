.. Copyright (C) 2015, Wazuh, Inc.

.. tabs::


  .. group-tab:: APT

    .. code-block:: console

        # wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/elasticsearch.gpg --import && chmod 644 /usr/share/keyrings/elasticsearch.gpg
        # echo "deb [signed-by=/usr/share/keyrings/elasticsearch.gpg] https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list
        # apt-get update 


  .. group-tab:: Yum

    .. code-block:: console

        # rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

        # cat > /etc/yum.repos.d/elastic.repo << EOF
        [elastic-7.x]
        name=Elastic repository for 7.x packages
        baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
        gpgcheck=1
        gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
        enabled=1
        autorefresh=1
        type=rpm-md
        EOF         

.. End of include file
