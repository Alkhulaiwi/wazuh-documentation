.. Copyright (C) 2015, Wazuh, Inc.

.. meta::
   :description: PingOne is a platform that enables enterprises to give their users federated access to applications. Learn more about it in this section of our documentation.

PingOne
=======

`PingOne for Enterprise <https://www.pingidentity.com/>`_ is an identity-as-a-service (IDaaS) and single sign-on (SSO) platform. It allows enterprises to give their users federated access to applications. In this guide, we integrate the PingOne IdP to authenticate users into the Wazuh platform.

The single sign-on integration process is divided into three stages.

#. PingOne Configuration
#. Wazuh indexer configuration
#. Wazuh dashboard configuration

PingOne Configuration
---------------------

#. Create an account in Ping Identity. Request a free trial if you don't have a paid license.
#. Go to `PingOne <https://admin.pingone.com/>`_ and sign in with your Ping Identity account.
#. Create an application in **Connections**.

   #. Navigate to **Connections** > **Applications** > **Add Application** and give it a name. In our case, the name is ``wazuh-sso``.

   #. Proceed to the **Choose Application Type** section, and select  **SAML Application** > **Configure**.

      .. thumbnail:: /images/single-sign-on/pingone/01-proceed-to-the-choose-application-type-section.png
          :title: Proceed to the Choose Application Type section
          :align: center
          :width: 80%

   #. Select **Manually Enter** on the **Provide App Metadata** page and add the following configuration, replacing ``<WAZUH_DASHBOARD_URL>`` with the corresponding value:

      - ACS URLs: ``https://<WAZUH_DASHBOARD_URL>/_opendistro/_security/saml/acs``
      - ENTITY ID: ``wazuh-saml``

      .. thumbnail:: /images/single-sign-on/pingone/02-select-manually-enter-on-the-provide-app-metadata.png
          :title: Select Manually Enter on the Provide App Metadata
          :align: center
          :width: 80%

   #. On the **Configuration** tab, click on the edit icon and add the following information:

      - SLO ENDPOINT: ``https://<WAZUH_DASHBOARD_URL>/``
      - SLO BINDING: ``HTTP Redirect``
      - ASSERTION VALIDITY DURATION: ``3600`` (for one hour token validity)
      - VERIFICATION CERTIFICATE (OPTIONAL): Load a PUBLIC CERTIFICATE that corresponds to the PRIVATE KEY that is going to be used on the ``sp.signature_private_key_filepath`` of the ``config.yml`` configuration file on the Wazuh indexer instance. This is necessary as all the logout requests must be signed.

      .. thumbnail:: /images/single-sign-on/pingone/03-on-the-configuration-tab.png
          :title: On the Configuration tab
          :align: center
          :width: 80%

   #. Click on the **Attribute Mappings** tab,  select the edit icon, click on **Add** and insert the following configuration:

      ``Roles`` = ``Group Names`` 

      .. thumbnail:: /images/single-sign-on/pingone/04-click-on-the-attribute-mappings-tab.png
          :title: Click on the Attribute Mappings tab
          :align: center
          :width: 80%

      The ``Roles`` attribute will be used later as the ``sp.entity_id`` in the Wazuh indexer configuration file.

   #. Click on the **Required** checkbox, and click on **Save**.

#. Create a group and assign users.
 
   #. Navigate to **Identities** > **Groups**, and click on the **+** sign. Select the name of the **Group**, in this case, ``Role``.

      .. thumbnail:: /images/single-sign-on/pingone/05-navigate-to-identities-groups.png
          :title: Navigate to Identities > Groups
          :align: center
          :width: 80%

   #. To assign users, open the created **Group**, go to the **Users** tab and select **Add Users Individually**. Add all the members that must log in to the Wazuh dashboard, and click on **Save** when done.

      .. thumbnail:: /images/single-sign-on/pingone/06-assign-users.png
          :title: Assign users
          :align: center
          :width: 80%

      .. thumbnail:: /images/single-sign-on/pingone/07-assign-users.png
          :title: Assign users
          :align: center
          :width: 80%

#. Activate the application and note the necessary parameters.

   #. Navigate to **Connections**, select **Applications**, and enable the application.

      .. thumbnail:: /images/single-sign-on/pingone/08-navigate-to-connections.png
          :title: Navigate to Connections
          :align: center
          :width: 80%
    
   #. Take note of the following parameters from the configuration page of the application. This information will be used in the next step. 

      - **ISSUER ID**: It'll be in the form “https://auth.pingone.com/....”
      - **IDP METADATA URL**: It’ll be in the form “https://auth.pingone.com/....”
      - ``exchange_key``: If you open IDP **IDP METADATA URL** you'll find the X509 Certificate  section, this will be used as the ``exchange_key``.

      .. thumbnail:: /images/single-sign-on/pingone/09-take-note-of-parameters.png
          :title: Take note of parameters from the configuration page
          :align: center
          :width: 80%


Wazuh indexer configuration
---------------------------

Edit the Wazuh indexer security configuration files. It is recommended to back up these files before the configuration is carried out.

#. Edit the ``/usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/config.yml`` file and change the following values:

   - Set the ``order`` in ``basic_internal_auth_domain`` to ``0`` and the ``challenge`` flag to ``false``. 

   - Include a ``saml_auth_domain`` configuration under the ``authc`` section similar to the following:


   .. code-block:: console
      :emphasize-lines: 7,10,22,23,25,26,27,28

          authc:
      ...
            basic_internal_auth_domain:
              description: "Authenticate via HTTP Basic against internal users database"
              http_enabled: true
              transport_enabled: true
              order: 0
              http_authenticator:
                type: "basic"
                challenge: false
              authentication_backend:
                type: "intern"
            saml_auth_domain:
              http_enabled: true
              transport_enabled: false
              order: 1
              http_authenticator:
                type: saml
                challenge: true
                config:
                  idp:
                    metadata_file: “/usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/Google_Metadata.xml”
                    entity_id: “https://accounts.google.com/o/saml2?idpid=C02…”
                  sp:
                    entity_id: wazuh-saml
                  kibana_url: https://<WAZUH_DASHBOARD_ADDRESS>
                  roles_key: Roles
                  exchange_key: 'X509Certificate'
              authentication_backend:
                type: noop

   Ensure to change the following parameters to their corresponding value:

   - ``idp.metadata_file``
   - ``idp.entity_id``
   - ``sp.entity_id``
   - ``kibana_url``
   - ``roles_key``
   - ``exchange_key``

#. Run the ``securityadmin`` script to load the configuration changes made in the ``config.yml`` file. 

   .. code-block:: console

      # export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/config.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h localhost -nhnv

   The ``-h`` flag is used to specify the hostname or the IP address of the Wazuh indexer node. Note that this command uses localhost, set your Wazuh indexer address if necessary.

   The command output must be similar to the following:

   .. code-block:: console
      :class: output

      Will connect to localhost:9300 ... done
      Connected as CN=admin,OU=Wazuh,O=Wazuh,L=California,C=US
      OpenSearch Version: 1.2.4
      OpenSearch Security Version: 1.2.4.0
      Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
      Clustername: wazuh-cluster
      Clusterstate: GREEN
      Number of nodes: 1
      Number of data nodes: 1
      .opendistro_security index already exists, so we do not need to create one.
      Populate config from /home/wazuh
      Will update '_doc/config' with /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/config.yml 
         SUCC: Configuration for 'config' created or updated
      Done with success

#. Edit ``/usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/roles_mapping.yml`` file and change the following values:
   
   Map the Group (Role) that is in PingOne to the ``all_access`` role in Wazuh indexer:

   .. code-block:: console
      :emphasize-lines: 6

      all_access:
        reserved: false
        hidden: false
        backend_roles:
        - "admin"
        - "Role"
        description: "Maps admin to all_access"


#. Run the ``securityadmin`` script to load the configuration changes made in the ``roles_mapping.yml`` file. 

   .. code-block:: console

      # export JAVA_HOME=/usr/share/wazuh-indexer/jdk/ && bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -f /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/roles_mapping.yml -icl -key /etc/wazuh-indexer/certs/admin-key.pem -cert /etc/wazuh-indexer/certs/admin.pem -cacert /etc/wazuh-indexer/certs/root-ca.pem -h localhost -nhnv

   The ``-h`` flag is used to specify the hostname or the IP address of the Wazuh indexer node. Note that this command uses localhost, set your Wazuh indexer address if necessary.

   The command output must be similar to the following:

   .. code-block:: console
      :class: output
            
      Security Admin v7
      Will connect to localhost:9300 ... done
      Connected as CN=admin,OU=Wazuh,O=Wazuh,L=California,C=US
      OpenSearch Version: 1.2.4
      OpenSearch Security Version: 1.2.4.0
      Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
      Clustername: wazuh-cluster
      Clusterstate: GREEN
      Number of nodes: 1
      Number of data nodes: 1
      .opendistro_security index already exists, so we do not need to create one.
      Populate config from /home/wazuh
      Will update '_doc/rolesmapping' with /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig/roles_mapping.yml 
         SUCC: Configuration for 'rolesmapping' created or updated
      Done with success

Wazuh dashboard configuration
-----------------------------

#. Edit the Wazuh dashboard configuration file.

   Add these configurations to ``/etc/wazuh-dashboard/opensearch_dashboards.yml``. It is recommended to back up this file before the configuration is changed.

   .. code-block:: console  

      opensearch_security.auth.type: "saml"
      server.xsrf.whitelist: ["/_plugins/_security/saml/acs", "/_plugins/_security/saml/logout", "/_opendistro/_security/saml/acs", "/_opendistro/_security/saml/logout", "/_opendistro/_security/saml/acs/idpinitiated"]

   .. note::
      :class: not-long

      *For versions 4.3.9 and earlier*, also replace ``path: `/auth/logout``` with ``path: `/logout``` in ``/usr/share/wazuh-dashboard/plugins/securityDashboards/server/auth/types/saml/routes.js``.

      .. code-block:: console
         :emphasize-lines: 3

         ...
            this.router.get({
               path: `/logout`,
               validate: false
         ...

#. Restart the Wazuh dashboard service.

   .. include:: /_templates/common/restart_dashboard.rst

#. Test the configuration.
   
   To test the configuration, go to your Wazuh dashboard URL and log in with your Ping One account. 
