Docker
=========

參考文件

- `iottalk-docker README.rst <https://gitlab.com/IoTtalk/iottalk-docker>`_

- joejoe2 ``docker-compose.yml``

|

Config Files
---------------

相關的設定檔

- Docker

  - docker-compose
  - DockerFile
  
    - ``iottalk-autogen/Dockerfile``
    - 產生 access_log_file, error_log_file 在 docker container 裡面的 file 路徑

      .. code:: sh

        mkdir -p /var/log/autogen && \
        mkdir -p /var/log/device

|

- Django

  - Django settings
  
    - ``iottalk-autogen/_/settings.py``
      
      - 裡面使用 ``iottalk-autogen/_/env/env`` 參數
  
        .. code:: py

          #iottalk-autogen/_/settings.py
          load_dotenv(os.path.join(BASE_DIR, '_/env/env'))


  - gunicorn config
  
    - ``iottalk-autogen/_/gunicorn/config.py``

      .. code:: py

        # iottalk-autogen/_/gunicorn/config.py

        'log_file': {
            'class': 'logging.handlers.RotatingFileHandler',

            # original code
            # there's sth wrong with the path in Docker Container

            # 'filename': os.path.join(settings.LOG_DIR, 'gunicorn_access.log'),

            # hotfix
            'filename': os.path.join('/var/log/autogen', 'gunicorn_access.log'),
            'maxBytes': 10 * 1024 * 1024,
            'backupCount': 3,
            'encoding': 'UTF-8',
        },
        'error_log_file': {
            'class': 'logging.handlers.RotatingFileHandler',

            # original code
            # 'filename': os.path.join(settings.LOG_DIR, 'gunicorn_error.log'),

            # hotfix
            'filename': os.path.join('/var/log/autogen', 'gunicorn_error.log'),
            'maxBytes': 10 * 1024 * 1024,
            'backupCount': 3,
            'encoding': 'UTF-8',
        },

|

相關 error msg

- ``Error: Unable to configure handler 'error_log_file'``

|

Account System
------------------

- 舊版: account.demo.iottalk.tw

- 新版: demo.iottalk.tw/account-system


|

Manage Docker as a non-root user
-----------------------------------

`Post-installation steps for Linux <https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user>`_

.. code::

  sudo groupadd docker
  sudo usermod -aG docker $USER
  sudo systemctl enable docker.service
  sudo systemctl enable containerd.service


|

Trouble shooting
---------------------

Error

.. code::

  ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?


Get docker info

.. code::

  docker info


Change priviledge

.. code::

  sudo chown $USER /var/run/docker.sock


- `ref <https://stackoverflow.com/a/52798075>`_


Nginx
--------

Check Listening Ports

.. code::

  sudo netstat -tunlp


- `ref <https://linuxize.com/post/check-listening-ports-linux/>`_


|


Check Nginx status

.. code::

  sudo systemctl status nginx

|

kill process by name

.. code::

  pkill nginx

|

Trouble shooting

.. code::

  Nginx: [emerg] BIO_new_file(“/etc/letsencrypt/live/domain.com/fullchain.pem”) failed 
  (SSL: error:02001002:system library:fopen:No such file or directory:fopen(‘/etc/letsencrypt/live/domain.com/fullchain.pem’,’r’)


Solution

``docker-compose.yml``

=> add ``/etc/letsencrypt/archive/demo.iottalk.tw:/etc/letsencrypt/archive/demo.iottalk.tw:ro`` in nginx service


.. code::

    nginx:
      image: nginx:1.15.11-alpine
      restart: always
      ports:
        - 80:80
        - 443:443
        - 8883:8883
        - 8884:8884
      volumes:
        - static-root:/usr/share/nginx/account_subsystem/staticfiles:ro
        - ./nginx/nginx.conf:/etc/nginx/nginx.conf
        - ./nginx/conf.d:/etc/nginx/conf.d
        - /etc/letsencrypt/live/demo.iottalk.tw:/etc/letsencrypt/live/demo.iottalk.tw:ro
        - /etc/letsencrypt/archive/demo.iottalk.tw:/etc/letsencrypt/archive/demo.iottalk.tw:ro
        # Apply certificate in nginx container
        # or comment out the line below and mount the certificate into nginx container
        # - nginx-cert:/etc/letsencrypt
        # - acme-challenge:/var/www/letsencrypt:ro
      networks:
        - iottalk-net
      depends_on:
        - mosquitto


|

Open Ports (might help, not sure)

.. code::

  sudo ufw allow 8883
  sudo ufw allow 8884

|



