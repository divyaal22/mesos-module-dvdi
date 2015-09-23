# docker-volume-driver-isolator
This is an Isolator for Mesos that provides external storage management for Mesos containers

for all builds  (docker host should have these already)


install bash-completion maven autoconf automake libtool subversion

install protobug-2.5.0
Mesos has this in tar.gz form but it must be uncompressed and installed
in the mesos/3rdparty/libprocess/3rdparty/protobuf-2.5.0 directory
also install gtest, glog, and picojson

export MESOS_HOME=/path/to/mesos
export GTEST_HOME=/path/to/gtest-1.7.0  
MAVEN_HOME = /usr/local/Cellar/maven/3.3.1



To build with vagrant docker image:

cd isolator
./bootstrap
rm -rf build
mkdir build
cd build
export LD_LIBRARY_PATH=LD_LIBRARY_PATH:/usr/local/lib
../configure --with-mesos-root=/mesos --with-mesos-build-dir=/mesos

A libmesos_dvdi_isolator.so and libmesos_dvdi_isolator-0.1.so will be generated
in isolator/build/.libs

Build Docker Images
-------------------
`docker build -t name/mesos-build-module-dev:0.23.0 - < Dockerfile-build-mesos-dev`

`docker build -t name/mesos-build-module-dvdi:0.23.0 - < Dockerfile-build-build-module-dvdi`


Build DVDI Module
-----------------
`git clone https://github.com/cantbewong/docker-volume-driver-isolator && cd docker-volume-driver-isolator`
`docker run -ti -v path-to-docker-volume-driver-isolator:/isolator clintonskitson/dvdi-modules:0.23.0`

Run Slave
---------
```
nohup  /usr/sbin/mesos-slave \ --isolation="com_emc_mesos_DockerVolumeDriverIsolator" \
--master=zk://172.31.0.11:2181/mesos \
--log_dir=/var/log/mesos \
--containerizers=docker,mesos \
--executor_registration_timeout=5mins \
--ip=172.31.2.11 --work_dir=/tmp/mesos \
--modules=file:///home/ubuntu/docker-volume-driver-isolator/isolator/modules.json &
```

REX-Ray Available At
---
[REX-Ray](https://github.com/emccode/rexray)

DVD CLI Available At
---
[dvdcli](https://github.com/clintonskitson/dvdcli)


Example DVD CLI Call
---

```
/go/src/github.com/clintonskitson/dvdcli/dvdcli mount --volumedriver=rexray --volumename=test123456789  --volumeopts=size=5 --volumeopts=iops=150 --volumeopts=volumetype=io1 --volumeopts=newFsType=ext4 --volumeopts=overwritefs=true
```

Example Marathon Call - test.json
---

`curl -i -H 'Content-Type: application/json' -d @test.json localhost:8080/v2/apps`

```
{
  "id": "hello-play",
  "cmd": "while [ true ] ; do touch /var/lib/rexray/volumes/test12345/hello ; sleep 5 ; done",
  "mem": 32,
  "cpus": 0.1,
  "instances": 1,
  "env": {
    "DVDI_VOLUME_NAME": "test1234567890",
    "DVDI_VOLUME_OPTS": "size=5,iops=150,volumetype=io1,newfstype=xfs,overwritefs=true"
  },
  "volumes": {
    "/var/lib/rexray/volumes/test12345":"/test12345"
  }
}
```
