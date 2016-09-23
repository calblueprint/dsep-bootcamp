# Bootcamp Day 2! 👢🏕

**Agenda:**

- Learn to Docker!
- Notebook in the cloud
- Persisting notebooks
- Adding dependencies
- JupyterHub: Down the rabbit hole (if time allows)

## Learn to Docker!

But first, [what is Docker][what]? If you like videos, [here's a longer intro to
Docker][vid].

[what]: https://www.docker.com/what-docker
[vid]: https://www.youtube.com/watch?v=Q5POuMHxW-0

*Insert discussion here*

Install docker: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

Using your Google Cloud VM, run through the Docker tutorial
https://docs.docker.com/engine/getstarted/ .

*Insert discussion here*

## Notebook in the cloud

Docker lets us get applications up and running super fast! Let's get a Jupyter
notebook server up and running on your Google Cloud VM in one command.

https://github.com/jupyter/docker-stacks/tree/master/minimal-notebook

## Persisting notebooks

You'll notice that when you make a notebook in that container and then `docker
stop <container>`, the notebook is gone. Try it out by making a notebook and
then `docker stop <container>`, then `docker run` the minimal notebook image.

Mount your home directory to the container so that notebooks will persist
across runs.

See https://docs.docker.com/engine/tutorials/dockervolumes/ for more about
mounting.

Note that you may run into permission denied issues in Jupyter once you mount.
This is because the UID of the user in the host machine is different from the
UID of `jovyan` in the container. You can test this by running

    echo $UID

In your Google machine and in the container (using `docker run -it
jupyter/minimal-notebook bash`).

If this is the case, you'll have to use the `NB_UID` option while starting your
notebook container. See
https://github.com/jupyter/docker-stacks/tree/master/minimal-notebook#docker-options
for more info.

## Adding dependencies

We'd like the `numpy` library but it's not installed. Install it by running

    !conda install numpy -y

In a notebook cell.

You should now be able to run:

    import numpy as np
    np.arange(15).reshape(3, 5)

And see output that looks like

    array([[ 0,  1,  2,  3,  4],
           [ 5,  6,  7,  8,  9],
           [10, 11, 12, 13, 14]])

What happens when you `docker stop` and then `docker run` again? Is the `numpy`
library still installed?

Modify the Dockerfile of
https://github.com/jupyter/docker-stacks/tree/master/minimal-notebook to have
`numpy` installed by default.

Build it as an image named `numpy-notebook`.

You should now be able to run

    import numpy as np
    np.arange(15).reshape(3, 5)

Right away in that notebook after `docker run`.

## JupyterHub: Down the rabbit hole

Alright, so you have your notebook container running. You probably don't want
it running for long, since anyone with your IP can find your Jupyter notebook
and run any command they want on your machine.

Instead, let's spin up a container running JupyterHub, a multi-user Jupyter
notebook.

Visit https://github.com/jupyterhub/jupyterhub and the README a pass over.
You'll notice that there's a command there to run a docker image with
JupyterHub installed. Go ahead and run it (make sure to pass the `-p
8000:8000` flag to `docker run`)

You should see a screen that looks like:

![][jhub]

[jhub]: https://www.dropbox.com/s/rghwceclsznvxw8/Screenshot%202016-09-22%2023.02.41.png?dl=1

Now, there are no users in the JupyterHub system right now so you won't be able
to log in. Let's create a user:

    docker exec -it jupyterhub bash
    # In the container:
    root@7b88829ea00d:/srv/jupyterhub# adduser <username>

Where `<username>` is anything you like. This will let you log into JupyterHub
and promptly run into a 500 error:

![][error]

[error]: https://www.dropbox.com/s/li9hbh1lbeu3p7b/Screenshot%202016-09-22%2023.08.23.png?dl=1

Why is this happening? Let's check out the logs:

    docker logs jupyterhub

    # You should see something that looks like:

    ...
    [I 2016-09-23 06:06:53.466 JupyterHub spawner:534] Spawning jupyterhub-singleuser --user=sam --cookie-name=jupyter-hub-token-sam --base-url=/user/sam --hub-host= --hub-prefix=/hub/ --hub-api-url=http://127.0.0.1:8081/hub/api --ip=127.0.0.1 --port=56719
    Traceback (most recent call last):
      File "/opt/conda/lib/python3.5/site-packages/jupyterhub/singleuser.py", line 15, in <module>
        import notebook
    ImportError: No module named 'notebook'
    ...

We're missing a Python package that didn't get included with the JHub image.
Let's install it.

    docker exec -it jupyterhub bash
    # In the container:
    root@7b88829ea00d:/srv/jupyterhub# conda install notebook -y

Now we should be able to log in and then see something that looks very
familiar:

![][success]

[success]: https://www.dropbox.com/s/3jxcvmq68idep5n/Screenshot%202016-09-22%2023.10.25.png?dl=1

Congrats! You've just used `docker` to get JupyterHub up and running!

Now you can go and give your url to classes at Berkeley. It'll work just fine,
until you get a class with more than 5 students. How can we scale it? That's a
topic for next time ;).
