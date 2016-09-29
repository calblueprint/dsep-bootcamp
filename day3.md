# Bootcamp Day 3! 👢🏕

**Agenda:**

- Literally ~~unplayable~~ unscalable JupyterHub
- A thought exercise
- How we scale the system for Data 8
- Deploy JupyterHub for reals
- How to look into the system

## Literally ~~unplayable~~ unscalable JupyterHub

So we've deployed a JupyterHub! First of all, what's the difference between
JupyterHub and a regular Jupyter notebook?

Now, we're done, right? We have a system where multiple people can log in and
create notebooks to do data analysis. Unfortunately, not quite. How many users
can our deployment handle at once if we give each user 0.5 GB of RAM?

## A thought exercise

Okay, so our computer isn't beefy enough for our needs. Why not just get a
bigger computer?

Okay, so even a really big computer won't be enough to serve the number of
students we want to handle. This is a case where we need
[horizontal scaling][horizontal] rather than vertical.

[horizontal]: http://stackoverflow.com/questions/11707879/difference-between-scaling-horizontally-and-vertically-for-databases

Data 8 has 527 students as of Sept 26, 2016. If we expect at most 60% of the
class to be online at any one point in time and we give each student 2 GB of
RAM, how many VMs do we need if each VM has 14 GB of RAM?

That's quite a few computers! One simple way to serve all these students is to
run the `docker run ... jupyterhub/jupyterhub` on each computer. What are
reasons to do this? What are reasons not to do this?

## How we scale the system for Data 8

Okay, so we're going to need something a bit more complicated than the simple
solution we've proposed just now. Before we can discuss how to scale this
system, we have to define our goals. What are our goals with this deployment of
JupyterHub?

Now, let's read over [Jess Hamrick's blog post][jess] on scaling JupyterHub.
Our deployment is almost exactly the same as hers. I'll explain the
differences. You don't have to understand all the concepts right now. Just try
to get a sense of how she scaled the system for more users.

[jess]: https://developer.rackspace.com/blog/deploying-jupyterhub-for-education/

## Deploy JupyterHub for reals

Enough talk! Let's deploy this system for reals.

Open up the [Deployment Notes][deployment] and click down to the Azure section.
Although the current deployment is on Microsoft Azure, we're going to use
Google Cloud since we're already using all the resources we have on Azure. It's
basically the same after you spin up the computers.

Don't worry about installing the `azure` command line tool. (Skip that
section.)

Let's spin up the computers we need. You're going to create 4 new computers:
the Hub, two Nodes, and a Proxy. Name them `<yourname>-<type>`. For example, I
named my Hub `sam-hub`.

All the computers need to run Ubuntu 16.04. The Hub and Proxy will have 1 vCPU,
the Nodes will have 2 vCPUs each. You need to check the Allow HTTP(S) traffic
boxes for the Proxy, but everything else you can leave alone.

Your original `<name>-test` machines are going to be your provisioners. You'll
only be using them to deploy JupyterHub which is why they're wimpy.

Once those are up and running, walk through the commands in the deployment doc
to deploy JupyterHub!

You're going to run into problems. I ran into like 5 just making this tutorial,
including a really nasty one that I fixed with [this][fix] but idk what
happened... Anyway, just let me know when you do. Also, remember that these
problems are the ones we're going to have to fix!

[deployment]: https://docs.google.com/document/u/1/d/1YcI-GtSDUK3PsAEAB8-JU9BkICvsmZqpCcDz3m4mdo0/edit?usp=drive_web
[fix]: https://github.com/docker/docker/issues/866#issuecomment-19218300

## How to look into the system

Phew, you're done. That was pretty painful, wasn't it?

Now that you have the system up and running, let's look at how to check that
it's working.

First, you should be able to see your JupyterHub up and running at your proxy's
IP address. You should hopefully be able to log in there and start your server.

Next, you can `ssh` into your `hub` and run `docker ps` to see a list of
running containers.

    root@hub-dev:~# docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    fdf1c925c4e6        cull                "python cull_idle_ser"   2 weeks ago         Up 2 weeks                              cull
    b1172abf9386        jupyterhub          "jupyterhub -f /srv/j"   2 weeks ago         Up 2 weeks                              jupyterhub
    97cdde8c9978        swarm               "/swarm manage --stra"   2 weeks ago
    Up 2 weeks                              swarm


Running

    docker logs -f --tail=100 jupyterhub

Will show you the logs for the `jupyterhub` container. (`-f` means follow the
logs, so any new log lines will show up immediately. `--tail=100` means show
the last 100 lines.)

This is very, very useful for finding errors.

I've mentioned the Interact server before. That's isn't running in a container,
but we can find out where it's logging by looking for its process:

    root@hub-dev:~# ps aux | grep interact
    root     31166  0.0  0.0  10320   160 ?        S    Sep11   0:00 daemon -n interact -o /var/log/interact.log --chdir=/srv -- python3 /srv/interact/run.py --production
    root     31167  0.3  0.3  82112 27396 ?        S    Sep11  84:34 /usr/bin/python3 /srv/interact/run.py --production
    root     47262  0.0  0.0  10468  2196 pts/0    S+   08:05   0:00 grep --color=auto interact


It looks like the `daemon` program is running our server and outputting logs to
`/var/log/interact.log`, so let's take a look at that:

    root@hub-dev:~# tail -f /var/log/interact.log
    [09/11/2016 07:08:26 AM]: INFO -- (samlau95) From https://github.com/data-8/textbook
    [09/11/2016 07:08:26 AM]: INFO -- (samlau95)  = [up to date]      gh-pages   -> origin/gh-pages
    [09/11/2016 07:08:26 AM]: INFO -- (samlau95)  = [up to date]      old-staging -> origin/old-staging
    [09/11/2016 07:08:26 AM]: INFO -- (samlau95)  = [up to date]      staging    -> origin/staging
    [09/11/2016 07:08:26 AM]: INFO -- Pulled from origin
    [09/11/2016 07:08:26 AM]: INFO -- Redirecting to /user/samlau95/tree/textbook/notebooks/Plotting_the_Classics.ipynb
    [09/11/2016 07:08:26 AM]: INFO -- /home/samlau95/textbook chown'd to samlau95
    [09/11/2016 07:08:26 AM]: INFO -- Sent message: {'payload': '/user/samlau95/tree/textbook/notebooks/Plotting_the_Classics.ipynb', 'type': 'REDIRECT'}
    [09/11/2016 07:09:41 AM]: INFO -- /srv/interact/app/handlers.py modified; restarting server
    [09/11/2016 07:09:41 AM]: INFO -- Starting interact app on port 8002

Finally, all of this log output is already being shown at
https://papertrailapp.com/, a service we're using to collect the log output.

That service also sets up alerts which go to the Data 8 Staff Slack channel —
https://ds8-fall2016.slack.com/messages/jhub-errors/.

We've been running into strange kernel issues recently. Let's use these log
outputs to see if we can diagnose a real error!
