
# ++ wrkz-oci

You can either build `Dockerfile.Wrkzd` for a quick image with a binary supplied from the CI chain, or use `Dockerfile.build.Wrkzd` to compile the latest available source from the development tree at GitHub.

Either way the result is a 25MB Alpine container (with glibc for compatibility).
NOTE: When testing I've just added Dockerfile.build-busybox.  At the moment you need to copy in libpthread.so.0 and librt.so.1 into /lib before the daemon will run.  I did this on Armbian with the following commands;
```
docker cp -L /lib/aarch64-linux-gnu/librt.so.1 2b18f23d8c96:/lib/
docker cp -L /lib/aarch64-linux-gnu/libpthread.so.0 2b18f23d8c96:/lib/
docker commit 2b18f23d8c96 wrkzdev
```
Then you can run Wrkzd in a 9MB container!

Coming soon; more great innovations!


## Create

Some basic instructions on getting the Wrkzcoin Dockerfiles in this repo built into a portable docker(OCI) image.
I wrote these Dockerfiles on Fedora with podman, however I've listed the commands below with docker, since it's more popular and I didn't want to confuse.  If you use podman just replace all instances of `docker` with `podman`.

This is pretty low-level stuff; if you know podman/docker at all you should already know this; for the rest of us, read on!

#### get teh Dockerfile
```
git clone https://github.com/19morpheus80/wrkz-oci.git
cd wrkz-oci
```

#### to build a container with binary from the CI chain
`docker build -f Dockerfile.Wrkzd -t wrkzdev .`

#### NOTE:  you can add build argument 'dl' to supercede the hard coded binary, as I will inevibatably not keep things up to date.
`docker build --build-arg dl="https://wrkzcoin.s3.fr-par.scw.cloud/cli/20200213-1405-7cf341a2-wrkzcoin-linux-leveldb.tar.gz" -f Dockerfile.Wrkzd -t wrkzdev .`

##### or to build everything from the GitHub development tree (takes a while)
`docker build -f Dockerfile.build.Wrkzd -t wrkzdev .`

#### if that works the following line will show the Wrkzd version
`docker run --rm wrkzdev`

#### see the image you have
`docker images`

#### if you don't have an existing blockchain you'll want the following lines to get it
##### NOTE: visit https://wrkz-data-dir.bot.tips/ for the latest version
```
mkdir data
wget -P data https://wrkz-data-dir.bot.tips/update_20200211/blocks.wrkz.bin
wget -P data https://wrkz-data-dir.bot.tips/update_20200211/blockindexes.wrkz.bin
```

#### get the checkpoints
`wget -P data https://net.wrkz.work/checkpoints/ -O wrkzcoin_checkpoints.csv`

#### create start script
##### assumes blocks are in subdirectory 'data'
if you dont want an interactive session, remove `-it` from the following line

```
echo "docker run -it --rm --name=wrkzd -p 17855:17855 -p 17856:17856 -v ./data:/root/data:z wrkzdev \
Wrkzd --data-dir=/root/data --load-checkpoints=/root/data/wrkzcoin_checkpoints.csv --rpc-bind-ip=0.0.0.0" > startwrkzd.sh
chmod +x startwrkzd.sh
```

#### start the container
`./startwrkzd.sh`

#### detach from an interactive session
`Ctrl+p, Ctrl+q`

#### attach to a running session
`docker attach wrkzd`

#### stop the container
`docker stop wrkzd`
If started with `--rm` then that's all you need to do.  Otherwise, `podman rm wrkzd` clears what's left.


