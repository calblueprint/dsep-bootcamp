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
