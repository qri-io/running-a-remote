### Setup Dokku on Amazon EC2

1. Provision an Amazon EC2 instance. I chose a small ubuntu image with at least a few gigs of attached storage.
2. Set up security groups to allow port 80 ingress at a minimum.
3. [Install Dokku](http://dokku.viewdocs.io/dokku/getting-started/installation/)
```
wget https://raw.githubusercontent.com/dokku/dokku/v0.21.4/bootstrap.sh;
sudo DOKKU_TAG=v0.21.4 bash bootstrap.sh
```
4. Proceed with the rest of the instructions in this folder's readme