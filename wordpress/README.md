# Wordpress yaml with phpMyAdmin

This is a yaml file with a Wordpress container configuration where the database and the mail html folders and files of the Wordpress instalation is stored locally.
I crete in this way so is an easy access to the DB via phpMyAdmin and also to the file in order to edit plugins, config and themes with a ftp connection (or via ssh)

Ports in Wordpress and phpMyAdmin can be change according to the ones free in your setup.
    ports:
        - "8078:80"
    ports:
        - "8080:80" 

One you have this completed, with the command:

  sudo docker-compose -p "wordpress1" up -d

dockers are created.

The -p "wordpress1" is to name all relative dockers with the same stack name.

If you manage to use different ports and stack names is possible to have as many Wordpress instalation as you want.
On ports, rememeber to use different internal ports but always reference the 80. Like:

Wordpress1
    ports:
        - "8078:80"
phpMyAdmin1
    ports:
        - "8080:80" 

Wordpress2
    ports:
        - "8082:80"
phpMyAdmin2
    ports:
        - "8084:80" 

Wordpress3
    ports:
        - "8086:80"
phpMyAdmin3
    ports:
        - "8088:80" 

This is very helpful and powerful for testing and development.

Remember that in my Proxmox repository, I explain how to install Portainer and access remotely via Internet, even with HTTPS
https://github.com/danyelous/Proxmox
