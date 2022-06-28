Production Deployment
=====================

Example how to deploy Phpox for single-instance production environment.

Clone repositories
------------------

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/mushorg-phpox-podman.git
    git clone https://github.com/extra2000/mushorg-phpox.git mushorg-phpox-podman/src/phpox
    git clone https://github.com/extra2000/mushorg-bfr.git mushorg-phpox-podman/src/bfr

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd mushorg-phpox-podman

Build phpox Image
-----------------

From the project root directory, ``cd`` into ``src/phpox/`` and then build:

.. code-block:: bash

    cd src/
    podman build -t extra2000/mushorg/phpox .

Deploy phpox
-------------

From the project root directory, ``cd`` into ``deployment/production/phpox``:

.. code-block:: bash

    cd deployment/production/phpox

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/phpox.yaml{.example,}

Create pod file:

.. code-block:: bash

    cp -v phpox-pod.yaml{.example,}

For SELinux platform, label the following files to allow to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/phpox_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/phpox_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "phpox_podman"

Deploy phpox:

.. code-block:: bash

    podman play kube --configmap configmaps/phpox.yaml --seccomp-profile-root ./seccomp phpox-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name phpox-pod-srv01
    systemctl --user enable container-phpox-pod-srv01.service
