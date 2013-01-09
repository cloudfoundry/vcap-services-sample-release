## Echo service under ccng
### Deploy echo service
Deploying echo service nodes under ccng is very much the same as that under
legacy CC. Add below section under `jobs` section:

    - name: echo_node
      template: echo_node
      instances: 1
      resource_pool: infrastructure
      persistent_disk: 128
      networks:
      - name: default
        static_ips:
        - 10.42.200.125
    - name: echo_gateway
      template: echo_gateway
      instances: 1
      resource_pool: infrastructure
      networks:
      - name: default

and below lines under `properties` section:

    properties:
      echo_gateway:
        token: changeechotoken
        service_timeout: 15
        node_timeout: 10
      echoserver:
        port: 5002

### Make echo service available
Under ccng, additional steps are necessary to make new services available
to users. To achieve that, you should use the high version vmc and the
vmc admin plugin:

    gem install vmc --pre
    gem install admin-vmc-plugin

login as the cc administator:

    vmc login $ADMIN_USER --password $ADMIN_PASSWORD
    vmc create-service-auth-token --provider core --token changeechotoken --label echo

$ADMIN_USER and $ADMIN_PASSWORD could be found in your bosh deployment
manifest under section `uaa.scim.users`. It may take several minutes
before it takes effect. Issue `vmc info --services` to make sure that
echo service is available.

### Create a user:

Creating a user becomes very much more complex under ccng since
`vmc register` is not supported. A simple way to do that is using the
script [setup_yeti](https://github.com/cloudfoundry/cloud_controller_ng/blob/master/bin/setup_yeti):

    setup_yeti <uaa_url> <uaa_cc_secret> <cc_url> <admin_email> <admin_password> <num_parallel_users> <user_email> <user_password> <org_name> <space_name>

arguments like `uaa_url, uaa_cc_secret, cc_url ...`  can be found in the deployment manifest.
