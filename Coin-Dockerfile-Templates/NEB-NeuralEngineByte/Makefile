# change image name
image_name=neb
# version tag
version=v1.0.9
# local storage for container dumps
container_savepath=${CONTAINER_SAVEPATH}

# change STRATUM_UPDATEPASS to your own password
STRATUM_UPDATEPASS=tu8tu5

# coin paths
walletuserid=2000
walletgroupid=2000
user_path=/home/walletuser
wallet_directory=.neb
wallet_daemon=neb-node
dataset=/usr/share/neb/neb-datasets.bin

# JSON-RPC (8432) and work service (3441) bind address, default 127.0.0.1
# both have NO authentication: firewall them to the stratum host when binding 0.0.0.0
rpc_bind=0.0.0.0
work_bind=0.0.0.0

# 64 hex lock paid by the jobs of the work service: neb-wallet lock-of neb1...
# without it the work service (port 3441) refuses miners
miner_lock=

#change container registry path and credentials
registry=ghcr.io/<username>
registry_username=xxx
registry_password=xxx

daemon_args=--run --dataset ${dataset} --datadir ${user_path}/${wallet_directory} --listen 0.0.0.0 --max-peers 8 --rpc-bind ${rpc_bind} --work-bind ${work_bind} --color never $(if ${miner_lock},--miner-lock ${miner_lock})

gitclone:
	git clone https://github.com/SilasEigen/neb

build:
	podman build --tag ${image_name} --build-arg STRATUM_UPDATEPASS=${STRATUM_UPDATEPASS} --build-arg walletuserid=${walletuserid} --build-arg walletgroupid=${walletgroupid} -f Dockerfile
push:
	podman push ${image_name} ${registry}/${image_name}:${version}
save:
	podman save -o ${container_savepath}container-${image_name}-${version}.tar ${image_name}
load:
	podman load -i ${container_savepath}container-${image_name}-${version}.tar
load-gz:
	podman load -i ${container_savepath}container-${image_name}-${version}.tar.gz
run:
	podman rm -i ${image_name} && podman run --name=${image_name} -dt --network=host --log-driver=journald -v /dev/log:/dev/log -v ${user_path}/${wallet_directory}:${user_path}/${wallet_directory} ${image_name} ${wallet_daemon} ${daemon_args}
