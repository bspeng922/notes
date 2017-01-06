# Heat Orchestration Template


### basic template to simply deploy a single compute instance

```
heat_template_version: 2015-04-30

description: Simple template to deploy a single compute instance

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: my_key
      image: F18-x86_64-cfntools
      flavor: m1.small
```

+ Each HOT template has to include the heat_template_version key with a valid version of HOT.

+ the description is optional, it is good practice to include some useful text that describes what users can do with the template.you can provide multi-line text in YAML

```
description: >
  This is how you can provide a longer description
  of your template that goes over several lines.
```

+ The resources section is required and must contain at least one resource definition.


###  defining a set of input parameters instead of hard-coding values

Input parameters defined in the parameters section of a HOT template allow users to customize a template during deployment.

```
heat_template_version: 2015-04-30

description: Simple template to deploy a single compute instance

parameters:
  key_name:
    type: string
    label: Key Name
    description: Name of key-pair to be used for compute instance
  image_id:
    type: string
    label: Image ID
    description: Image to be used for compute instance
  instance_type:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: { get_param: key_name }
      image: { get_param: image_id }
      flavor: { get_param: instance_type }
```

You can also define default values for input parameters which will be used in case the user does not provide the respective parameter during deployment. 

```
parameters:
  instance_type:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used
    default: m1.small
```

Another option that can be specified for a parameter is to hide its value when users request information about a stack deployed from a template. This is achieved by the hidden attribute and useful, for example when requesting passwords as user input:

```
parameters:
  database_password:
    type: string
    label: Database Password
    description: Password to be used for database
    hidden: true
```

### Restricting user input

In some cases you might want to restrict the values of input parameters that users can supply.For example, you might know that the software running in a compute instance needs a certain amount of resources so you might want to restrict the instance_type parameter introduced above. 

```
parameters:
  instance_type:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used
    constraints:
      - allowed_values: [ m1.medium, m1.large, m1.xlarge ]
        description: Value must be one of m1.medium, m1.large or m1.xlarge.
```

The constraints section allows for defining a list of constraints(约束) that must all be fulfilled by user input. 

```
parameters:
  database_password:
    type: string
    label: Database Password
    description: Password to be used for database
    hidden: true
    constraints:
      - length: { min: 6, max: 8 }
        description: Password length must be between 6 and 8 characters.
      - allowed_pattern: "[a-zA-Z0-9]+"
        description: Password must consist of characters and numbers only.
      - allowed_pattern: "[A-Z]+[a-zA-Z0-9]*"
        description: Password must start with an uppercase character.
```

Note that you can define multiple constraints of the same type. Especially in the case of allowed patterns this not only allows for keeping regular expressions simple and maintainable(可维持的), but also for keeping error messages to be presented to users precise.


### Providing template outputs

you will typically want to provide outputs to users, which can be done in the outputs section of a template. For example, the IP address by which the instance defined in the example above can be accessed should be provided to users.

```
outputs:
  instance_ip:
    description: The IP address of the deployed instance
    value: { get_attr: [my_instance, first_address] }
```



## WordPress 

```
heat_template_version: 2013-05-23

description: >
  Heat WordPress template to support F20, using only Heat OpenStack-native
  resource types, and without the requirement for heat-cfntools in the image.
  WordPress is web software you can use to create a beautiful website or blog.
  This template installs a single-instance WordPress deployment using a local
  MySQL database to store the data.

parameters:

  key_name:
    type: string
    description: Name of a KeyPair to enable SSH access to the instance
  instance_type:
    type: string
    description: Instance type for WordPress server
    default: m1.small
  image_id:
    type: string
    description: >
      Name or ID of the image to use for the WordPress server.
      Recommended values are fedora-20.i386 or fedora-20.x86_64;
      get them from http://cloud.fedoraproject.org/fedora-20.i386.qcow2
      or http://cloud.fedoraproject.org/fedora-20.x86_64.qcow2 .
    default: fedora-20.x86_64

  db_name:
    type: string
    description: WordPress database name
    default: wordpress
    constraints:
      - length: { min: 1, max: 64 }
        description: db_name must be between 1 and 64 characters
      - allowed_pattern: '[a-zA-Z][a-zA-Z0-9]*'
        description: >
          db_name must begin with a letter and contain only alphanumeric
          characters
  db_username:
    type: string
    description: The WordPress database admin account username
    default: admin
    hidden: true
    constraints:
      - length: { min: 1, max: 16 }
        description: db_username must be between 1 and 16 characters
      - allowed_pattern: '[a-zA-Z][a-zA-Z0-9]*'
        description: >
          db_username must begin with a letter and contain only alphanumeric
          characters
  db_password:
    type: string
    description: The WordPress database admin account password
    default: admin
    hidden: true
    constraints:
      - length: { min: 1, max: 41 }
        description: db_password must be between 1 and 41 characters
      - allowed_pattern: '[a-zA-Z0-9]*'
        description: db_password must contain only alphanumeric characters
  db_root_password:
    type: string
    description: Root password for MySQL
    default: admin
    hidden: true
    constraints:
      - length: { min: 1, max: 41 }
        description: db_root_password must be between 1 and 41 characters
      - allowed_pattern: '[a-zA-Z0-9]*'
        description: db_root_password must contain only alphanumeric characters

resources:
  wordpress_instance:
    type: OS::Nova::Server
    properties:
      image: { get_param: image_id }
      flavor: { get_param: instance_type }
      key_name: { get_param: key_name }
      user_data:
        str_replace:
          template: |
            #!/bin/bash -v

            yum -y install mariadb mariadb-server httpd wordpress
            touch /var/log/mariadb/mariadb.log
            chown mysql.mysql /var/log/mariadb/mariadb.log
            systemctl start mariadb.service

            # Setup MySQL root password and create a user
            mysqladmin -u root password db_rootpassword
            cat << EOF | mysql -u root --password=db_rootpassword
            CREATE DATABASE db_name;
            GRANT ALL PRIVILEGES ON db_name.* TO "db_user"@"localhost"
            IDENTIFIED BY "db_password";
            FLUSH PRIVILEGES;
            EXIT
            EOF

            sed -i "/Deny from All/d" /etc/httpd/conf.d/wordpress.conf
            sed -i "s/Require local/Require all granted/" /etc/httpd/conf.d/wordpress.conf
            sed -i s/database_name_here/db_name/ /etc/wordpress/wp-config.php
            sed -i s/username_here/db_user/ /etc/wordpress/wp-config.php
            sed -i s/password_here/db_password/ /etc/wordpress/wp-config.php

            systemctl start httpd.service
          params:
            db_rootpassword: { get_param: db_root_password }
            db_name: { get_param: db_name }
            db_user: { get_param: db_username }
            db_password: { get_param: db_password }

outputs:
  WebsiteURL:
    description: URL for Wordpress wiki
    value:
      str_replace:
        template: http://host/wordpress
        params:
          host: { get_attr: [wordpress_instance, first_address] }
```

