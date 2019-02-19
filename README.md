COA Cheat Sheet
===============


### Identity: Keystone


1.  Create a project named "new-project1", a user "new-user1" with password "new-user1" , and assign role "manager" to that user.
   
    Create new project
    ```
    openstack project create new-project1 --description "new project" --domain default
    ```
    Create new user with password
    ```
    openstack user create --project new-project1 --domain default --email "new-user1@user.id" --password new-user1
    ```
    Assign role manager
    ```
    openstack role add --user new-user1 --project new-project1 --user-domain default manager
    ```
2.  Without accessing Horizon, build a user credential file to get access to CLI openstack client.  
    - project domain id : default
    - user domain id : default
    - project name : admin
    - user name : admin
    - type of password : password
    
    Get project name & id    
    ```
    openstack --os-auth-url http://192.168.27.2:5000/v3/ --os-project-domain-id default \
        --os-user-domain-id default --project-name admin --os-username admin \
        --os-password rahasia --os-identity-api-version 3 project list
    ```
    Get neutron endpoint id
    ```
    openstack --os-auth-url http://192.168.27.2:5000/v3/ --os-project-domain-id default \
        --os-user-domain-id default --project-name admin --os-username admin \
        --os-password rahasia --os-identity-api-version 3 endpoint list
    ```
    Using endpoint id, get auth url
    ```
    openstack --os-auth-url http://192.168.27.2:5000/v3/ --os-project-domain-id default \
        --os-user-domain-id default --project-name admin --os-username admin \
        --os-password rahasia --os-identity-api-version 3 endpoint show ENDPOINT-ID
    ```
    Build the file
    ```
    #!/bin.bash
    export OS_AUTH_URL=http://x.x.x.x:35357/v3.0
    export OS_TENANT_ID=xxxxx
    export OS_TENANT_NAME="admin"
    export OS_USERNAME="admin"
    export OS_PASSWORD="rahasia"
    ```
    Source to file
3. A new service is developed to manage projects IPv4 and IPv6 address space (IPAM). You are asked to create a service and associate the appropriate endpoints:

    Service

       - type: ipam
       - name: ipam
       - description: ipam service

    Endpoint Interfaces

        - public http://controller:60666
        - admin http://controller:60666
        - internal http://controller:60666
        - Region: RegionOne


    Create service ipam
    ```
    openstack service create --name ipam --description "ipam service" ipam
    ```
    Create endpoint

    ```
    openstack endpoint create --region RegionOne ipam public http://192.168.27.2:60666 
    openstack endpoint create --region RegionOne ipam admin http://192.168.27.2:60666
    openstack endpoint create --region RegionOne ipam internal http://192.168.27.2:60666
    ```


    #### Domain spesific configuration

    - Create a domain named "domain1"
        ```
        openstack domain create domain1
        ```
    - Create a project named "domain1_project1" within the domain "domain1"
        ```
        openstack project create domain1_project1 --domain domain1
        ```
    - Create a user "domain1_admin" in the domain "domain1"
        ```
        openstack user create --domain domain1 --email "domain1_admin@domain1.id" --password rahasia
        ```
    - Assign domain scope "admin" role to user "domain1_admin"
        ```
        openstack role add --domain domain1 --user domain1_admin admin
        ```
    - Create a regular user within the domain "domain1"
        ```
        openstack user create --domain domain1 --project domain1_project1 --email "user1@user.id" --password rahasia
        ```

### Nova

1. (Keypair) From demo account, generate a public keypair named mypubkey1 for use with openstack instances
   ```
   nova keypair-add mypubkey1 > key1.pem
   chmod 600 key1.pem
   ```
2. (Flavor) From admin account, create a new flavor named m1.extra_tiny with
   - RAM: 64mb
   - Root disk size: 0
   - CPU: 1
    ```
    . admin_rc.sh
    nova flavor-create m1.extra_tiny auto 64 0 1 --rxt-factor 1.0
    ```
3. Allow the tenant (project) "demo" to access the flavor
   ```
   nova flavor-access-add m1.extra_tiny demo
   ```
4. Add new rules to the default security group "default" to allow access instances from internet through SSH, http and ICMP.
   ```
   openstack security group rule create --proto icmp
   openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 22 default
   openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 80 default
   ```
5. Provision the following instance
   
   - image: cirros-0.3.4-x86_64-uec
   - flavor: m1.tiny
   - keypair: key1
   - security group: default

6. Create a pair key with ssh-keygen (we want to have a keypair created outside of openstack). Now, add the created key with openstack with name "mykey1"
7. This time, create a pair key directly with openstack and name it "mykey2"
8. Provision the following instance
9.  Create a floating IP (from the public subnet)and assign it to the instance7
10. Log to the machine console using the keypair mykey1 using the floating IP assigned to the instance7

### Quotas

1. Make sure tenant demo have the following limits
2. Make sure user demo from tenant demo have the following limits
3. Using p1_user1/openstack, check that project1 cannot create more instances
4. Make sure that users in project1 can create 2 more instances
5. Check that the new quotas is also applied for user=p1_user2,pass=openstack
6. We want user=p2_user1,pass=openstack is able to create only 2 floating ips, but 5 floating ips for p2_user2/openstack
7. Change the default quotas for any new project to:
8. Create project "project3" and user "user3" within.
9. Again, change the default quotas for any new project to:
10. Create project "project4" and two users "user41" and "user42" within.
11. Change the quotas for "user41" within "project4" to:
    
    - max 1 instances
    - max 1 cores
    - max 11111 RAM
    - max 11 floating_ips

