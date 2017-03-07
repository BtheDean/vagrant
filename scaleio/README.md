vagrant-scaleio
---------------

# Description

This Vagrant setup will automatically deploy ScaleIO in an isolated environment on top of VirtualBox.

Environment Details:

- Three CentOS 7.1 nodes
- Each node gets installed with the latest the ScaleIO software
- Configuration happens automatically to have a fully redundant ScaleIO cluster.

Other Optional Software Installations for Container Usage (edit the Vagrant file):

- [Docker](https://docker.com) (Installed by default)
- [REX-Ray](https://github.com/codedellemc/rexray) (Installed by default)
- [Apache Mesos](http://mesos.apache.org/) and [Marathon by Mesosphere](https://github.com/mesosphere/marathon) (NOT Installed by default)

## Requirements:

VirtualBox and Vagrant

For optional proxy setup, make sure you have the `vagrant-proxyconf` plugin installed.

## Usage

1. `git clone https://github.com/codedellemc/vagrant.git`
2. `cd vagrant/scaleio`
2. Edit the proxies (if needed)
3. Edit the `clusterinstall` parameter to adjust for different installation methods (default is True which mean a fully working ScaleIO cluster gets installed)
3. Edit the `dockerinstall`, `rexrayinstall`, or `mesosinstall` parameter to adjust enable or disable installation
4. Run `vagrant up` (if you have more than one Vagrant Provider on your machine run `vagrant up --provider virtualbox` instead)

Note, the cluster will come up with the default unlimited license for dev and test use.

### SSH
To login to the ScaleIO nodes, use the following commands: ```vagrant ssh mdm1```, ```vagrant ssh mdm2```, or ```vagrant ssh tb```.

### Cluster install function

In the Vagrantfile there is a variable named `clusterinstall` that controls how Vagrant provisions ScaleIO during `vagrant up` process.

If set to `True` (default), a fully functional ScaleIO cluster is installed with IM, MDM, TB, SDC and SDS on three nodes.

If set to `False`, three base VMs are installed with IM running on the machine named MDM1. To install your cluster when using `clusterinstall=False` you do `vagrant up` as usual but once complete use your web browser and point it to https://192.168.50.12. Login with `admin` and `Scaleio123`. From here you can deploy a new ScaleIO cluster using IM, great for demo and learning purposes.

### Example CSV file for deployment of ScaleIO cluster using IM:

```
IPs,Password,Operating System,Is MDM/TB,Is SDS,SDS Device List,Is SDC
192.168.50.12,vagrant,linux,Primary,Yes,/home/vagrant/scaleio1,Yes
192.168.50.13,vagrant,linux,Secondary,Yes,/home/vagrant/scaleio1,Yes
192.168.50.11,vagrant,linux,TB,Yes,/home/vagrant/scaleio1,Yes
```

### Docker, REX-Ray, and Mesos Installation

The sets of scripts will automatically install Docker and REX-Ray on all three nodes. Each will configure REX-Ray to use libStorage to manage ScaleIO volumes for persistent applications in containers.

To run a container with persistent data stored on ScaleIO, from any of the cluster nodes you can run the following examples:

Run Busybox with a volume mounted at `/data`:
```
docker -it --volume-driver=rexray -v data:/data busybox
```

Run Redis with a volume mounted at `/data`:
```
docker run -d --volume-driver=rexray -v redis-data:/data redis
```

Run MySQL with a volume mounted at `/var/lib/mysql`:
````
docker run -d --volume-driver=rexray -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql
````

For [Apache Mesos](http://mesos.apache.org/) and [Marathon by Mesosphere](https://github.com/mesosphere/marathon)  instructions for deploying containers, visit the [{code} Labs](https://github.com/codedellemc/labs) and try [Storage Persistence with Postgres using Mesos, Marathon, Docker, and REX-Ray](Storage Persistence with Postgres using Mesos, Marathon, Docker, and REX-Ray)

#### Docker High Availability

Since the nodes all have access to the ScaleIO environment, fail over services with REX-Ray are available by stopping a container with a persistent volume on one host, and start it on another. Docker's integration with REX-Ray will automatically map the same volume to the new container, and your application can continue working as intended.

### ScaleIO GUI

The ScaleIO GUI is automatically extracted and put into the `vagrant/scaleio/gui` directory, just run `run.sh` and it should start up. Connect to your instance with the credentials outlined in the [Cluster install function](# Cluster install function).

The end result will look something like this:

![alt text](docs/images/scaleio-docker-rexray.png)

# Troubleshooting

If anything goes wrong during the deployment, run `vagrant destroy -f` to remove all the VMs and then `vagrant up` again to restart the deployment.
