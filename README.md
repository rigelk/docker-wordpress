Press Gang
==========

A dev wordpress image, to run [Wordpress](https://wordpress.org/download/) inside a self-sufficient [Docker](https://www.docker.com) container. Ships with nginx, php, latest wordpress and a mysql base.

Getting it
----------

Build the image:

    docker build -t ubermuda/docker-wordpress .

Or just pull the trusted build:

    docker pull ubermuda/docker-wordpress

Usage
-----

Running it alone uses the inner mysql database and creates a new wordpress instance :

    docker run -d -P ubermuda/docker-wordpress

You might want to have the database and `wp-content` in shared volumes. I myself like to run the following for convenience and super-easy dev :

    docker run -d -p 80:80 --name wordpress -v $(pwd)/wordpress/wp-content/:/var/www/wordpress/wp-content ubermuda/docker-wordpress

---

To have a more reliable solution, you need to create a data container that will only hold data. You just have to run a container with the `-v` option to create volumes:

    docker run \
        --name wordpress-data \
        -v /var/lib/mysql \
        -v /var/www/wordpress/wp-content \
        busybox \
        /bin/true

__OR__ you could write a Dockerfile for your data container, for example in a `wordpress-data/`:

    FROM        busybox
    VOLUME      ["/var/lib/mysql", "/var/www/wordpress/wp-content"]
    ENTRYPOINT  ["/bin/true"]

then build and run it:

    docker build -t ubermuda/docker-wordpress-data wordpress-data/
    docker run --name wordpress-data ubermuda/docker-wordpress-data

Any way you chose, you can now use your newly created volumes in the wordpress container:

    docker run -d -P \
        --name wordpress \
        --volumes-from wordpress-data \
        ubermuda/docker-wordpress

Configure nginx as a reverse proxy
----------------------------------

Retrieve your container's port:

    docker port wordpress 80

Use this configuration in nginx, changing `{{ port }}` and `{{ server_name }}` accordingly:

    server {
            listen  80;
            server_name {{ server_name }};
            location / {
                proxy_pass http://127.0.0.1:{{ port }}/;
                proxy_redirect     off;
                proxy_set_header   Host $host;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;
            }
    }

Disclaimer
----------

This image contains default configurations for nginx, php5 and mysql. It is *not* recommended for production. However, if you're willing to contribute better configurations, please open a pull request!
