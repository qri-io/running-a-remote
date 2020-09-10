# Running A Remote

A "remote" is a qri _node_ that other users can push datasets to. "Running a remote" means setting up an always-on server so users can push & pull data to it. You may want to do this for a few reasons:

* You're working with private data, which can't be pushed to qri.cloud (private hosting)
* You have datasets that are larger than qri.cloud will accept (big data)
* You want to create a backup of datasets that you control ("lots of copies keeps stuff safe" or LOCKSS)
* You're working with a small group of collaborators and want to use a remote as a point of curation (team server)
* You want your own "copy" of the qri API to build on top of (self-hosted integrations)
* You have an extra computer kicking around and can wire it up for free dataset hosting (network-attached-storage)
* You want to stream data to a server for automated version creation, from IOT data collection points or at the end of a message queue like kafka (API-backed data sink)

Qri has the capacity to run as a remote built-in, but needs to be configured to enable "remote mode". We're actively using this repo to develop remote configurations & gather feedback on how they could be better.

### Working with a remote

Before we get into setting up a remote, it helps to see what one can actually do. Here's a quick rundown on pushing to a remote we've already set up:

#### Install qri locally

```sh
# install qri if you haven't already. You may need to run with sudo:
curl -fsSL https://qri.io/install.sh | bash -
```

#### Configure a remote

```sh

# we've set up a remote named "doug" that you can play with,
# let's add it to our configuration, using "doug" as our alias for https://doug.qri.cloud
qri config set remotes.doug https://doug.qri.cloud
```

#### Make a new dataset, save, and push
```sh
# next you'll need a dataset.
# If you don't already have one, the CLI quickstart has instructions for
# creating one
# https://qri.io/docs/getting-started/qri-cli-quickstart
# or use this simple earthquakes csv:

# download a csv of earthquakes data from usgs
curl https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.csv -o quakes.csv

# make a commit, which also creates the new dataset
qri save --body quakes.csv me/earthquakes

# push a dataset to the doug remote:
qri push me/earthquakes --remote doug

# party. Visit https://doug.qri.cloud/get/{username}/{dataset} to see your
# dataset hosted by doug
```


### Run a local remote in docker

If you have docker installed, you can run a remote on your local machine and push to it. In one terminal, spin up the following:

```shell
docker run -p 9000:2503 qriio/public-remote:latest
```

Here we're mapping port 2503 of the container (qri's default API port) to port 9000 on `localhost`, mainly so it doesn't conflict with the "main" version of qri already, which will need port 2503.

To push to this remote, open another terminal, add the remote to your config, and push:

```
qri config set remotes.local-docker-test http://localhost:9000

# push a dataset:
qri push b5/usgs_earthquakes --remote local-docker-test
```

Visit http://localhost:9000/list to see your dataset hosted by the local docker remote.

## Next Steps

The steps above will let you run an ephemeral qri remote for testing purposes.  If you want to set up something more permanent, here are some other steps to consider:

### Persisting Data Using Docker Volumes

If you use `docker run` with the `qriio/public-remote` docker image, you'll have a new remote running that stores its qri configuration and data inside the running container.  When the docker container is stopped, you'll lose everything.

To persist the configuration and data, you can use docker volumes to ensure that the `.qri` directory lives on after the container stops.  `qriio/public-remote` stores data in `/data/qri`.

In this `docker run` command, we map the directory `/root/qri` on the local filesystem to `/data/qri` in the container.  

```shell
docker run -p 9000:2503 -v /root/qri:/data/qri qriio/public-remote:latest
```

The container can be stopped and destroyed. Running the same `docker run` command again will start a new remote with the same qri identity and data.
