# SIEMS
- WAZUH installation 
- Wazuh is a comprehensive open-source security monitoring platform that includes features for intrusion detection, vulnerability detection, configuration assessment, and more.  Setting up Wazuh on an Ubuntu server involves installing both the Wazuh manager and the Wazuh agent.

- Here's a step-by-step guide to installing Wazuh on a kali linux server:
- install the following packages if missing.
- apt-get install gnupg apt-transport-https
  <br><img src="https://github.com/collinsbigomba/SIEMS/blob/main/images/waz.png"></br>
- Install the GPG key.
- curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
  <br><img src="https://github.com/collinsbigomba/SIEMS/blob/main/images/waz1.png"></br>
- Add the repository.
- echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
  <br><img src="https://github.com/collinsbigomba/SIEMS/blob/main/images/waz2.png"></br>
- Update the packages information.
- apt-get update
  <br><img src="https://github.com/collinsbigomba/SIEMS/blob/main/images/waz3.png"></br>
## Installing the Wazuh manager
- apt-get -y install wazuh-manager
  <br><img src="https://github.com/collinsbigomba/SIEMS/blob/main/images/waz4.png"></br>
- Enable and start the Wazuh manager service.
- systemctl daemon-reload
- systemctl enable wazuh-manager
- systemctl start wazuh-manager
- Run the following command to verify the Wazuh manager status.
- systemctl status wazuh-manager 
## Installing Filebeat
- Install the Filebeat package.
- apt-get -y install filebeat
- Configuring Filebeat
- Download the preconfigured Filebeat configuration file.
- curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.7/tpl/wazuh/filebeat/filebeat.yml
- Edit the /etc/filebeat/filebeat.yml configuration file and replace the following value:
- hosts: The list of Wazuh indexer nodes to connect to. You can use either IP addresses or hostnames. By default, the host is set to localhost hosts: ["127.0.0.1:9200"].      Replace it with your Wazuh indexer address accordingly.
- If you have more than one Wazuh indexer node, you can separate the addresses using commas. For example, hosts: ["10.0.0.1:9200", "10.0.0.2:9200", "10.0.0.3:9200"]
         # Wazuh - Filebeat configuration file
         output.elasticsearch:
         hosts: ["10.0.0.1:9200"]
         protocol: https
         username: ${username}
         password: ${password}
- Create a Filebeat keystore to securely store authentication credentials.
- filebeat keystore create
- Add the default username and password admin:admin to the secrets keystore.
- echo admin | filebeat keystore add username --stdin --force
- echo admin | filebeat keystore add password --stdin --force
- Download the alerts template for the Wazuh indexer.
- curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.7.4/extensions/elasticsearch/7.x/wazuh-template.json
- chmod go+r /etc/filebeat/wazuh-template.json
- Install the Wazuh module for Filebeat.
- curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.3.tar.gz | tar -xvz -C /usr/share/filebeat/module
- Deploying certificates
  ## NB
- Make sure that a copy of the wazuh-certificates.tar file, created during the initial configuration step, is placed in your working directory.
- Replace <SERVER_NODE_NAME> with your Wazuh server node certificate name, the same one used in config.yml when creating the certificates. Then, move the certificates to their corresponding location.
- NODE_NAME=<SERVER_NODE_NAME>
- mkdir /etc/filebeat/certs
- tar -xf ./wazuh-certificates.tar -C /etc/filebeat/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
- mv -n /etc/filebeat/certs/$NODE_NAME.pem /etc/filebeat/certs/filebeat.pem
- mv -n /etc/filebeat/certs/$NODE_NAME-key.pem /etc/filebeat/certs/filebeat-key.pem
- chmod 500 /etc/filebeat/certs
- chmod 400 /etc/filebeat/certs/*
- chown -R root:root /etc/filebeat/certs
## Starting the Filebeat service
- Enable and start the Filebeat service.
- systemctl daemon-reload
- systemctl enable filebeat
- systemctl start filebeat
- Run the following command to verify that Filebeat is successfully installed.
- filebeat test output
- Expand the output to see an example response.
        Output
        elasticsearch: https://127.0.0.1:9200...
          parse url... OK
          connection...
            parse host... OK
            dns lookup... OK
            addresses: 127.0.0.1
            dial up... OK
          TLS...
            security: server's certificate chain verification is enabled
            handshake... OK
            TLS version: TLSv1.3
            dial up... OK
          talk to server... OK
          version: 7.10.2
- Your Wazuh server node is now successfully installed. Repeat this stage of the installation process for every Wazuh server node in your Wazuh cluster, then proceed with configuring the Wazuh cluster. If you want a Wazuh server single-node cluster, everything is set and you can proceed directly with Installing the Wazuh dashboard step by step.
