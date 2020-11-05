# Nginx for serving static content

This little exercise project is based on the linkedin learning course [Learning Nginx](https://www.linkedin.com/learning/learning-nginx) by [Michael Jenkins](https://www.linkedin.com/in/michaelpjenkins). 

While the course uses Vagrant to manage a virtual machine to run  Nginx and MongoDB to serve a little website, I wanted to experiment a bit and replicate the instructions using Docker instead.  

The offical documentation appeared to be a great place to learn how to use the nginx docker container: [how to use official docker image](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image/)

# Run the web service

To run the webservice use docker compose:

	docker-compose up

In the browser use:

	localhost:8080

Your browser should complain about the invalid cert, as the cert was created locally and is not authorized by a trusted cert issuer. Just accept the connection and the web service will be served over https. 

# Configuration

Instead of baking the configuration into a new container with a dockerfile I decided to use docker volumes, which allow me to edit and test configurations while the nginx container is still running. Therefore I don't have to rebuild the image after every change and I can use `nginx` commands to test and reload configurations as if it is running on a server.

How the configurations are mapped into the container can be found in the [docker-compose.yml](docker-compose.yml).

## Static Content files

The static web files are located in the [wisdompetmed.local](wisdompetmed.local) folder. I added a admin page which is password protected by nginx.

The comply with security practices the file permission are set to read-only for everyone exept the root user:

	find wisdompetmed.local/ -type f -exec chmod 644 {} \;
	find wisdompetmed.local/ -type d -exec chmod 755 {} \;

## Routes

The routes are defined in the [wisdompetmed.local.conf](conf.d/wisdompetmed.local.conf) file. 

## Admin page Authentication

The users and passwords for the admin page was generated with the [apache utils docker image](https://hub.docker.com/r/elswork/apache2-utils). So no apache server installation is necessary. The generated credentials are saved to the [auth](auth/) folder. 

The apache util docker image was used as followed:

	docker run -v $(pwd)/auth:/workspace -w /workspace elswork/apache2-utils htpasswd -b -c passwords admin admin

This will generate a passwords file in the auth folder with the user credentials:

- username: admin
- password: admin

For help on how to use the `htpasswd` tool run:

	docker run elswork/apache2-utils htpasswd --help

## SSL cert with open SSL:

The SSL key and cert are configured as follows:

	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /ssl/private/nginx.key -out /ssl/certs/nginx.crt 

Note: When running the command the first time I had to create the /ssl/private and /ssl/certs folder manually. 

# Coming Next
The online course includes a configuration for dynamic content with PHP and a MongoDB database, which is not part of this docker experiment. When I have time I will add these by creating a Dockerfile. 

# Last Notes to the course
The online course also contained instructions to use nginx as a proxy pass and loadbalancer. But with docker the instructions couldn't be replicated. The root cause wasn't explored yet. 