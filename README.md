# tc-vmware-ubuntu-zestyzapus64-rocksdb

Ubuntu 17.04 Vagrant WMVare Development machine for RocksDB development with Golang.

## Getting started

```bash
cd $HOME/Desktop
git clone https://github.com/topconnector/tc-vmware-ubuntu-zestyzapus64-rocksdb.git
cd tc-vmware-ubuntu-zestyzapus64-rocksdb
```

You must have the following installed:

* VMware Fusion (Pro) >= 8.5
  
  https://www.vmware.com/products/fusion/fusion-evaluation.html
    
* Vagrant >= 1.9.7

  Download and install from https://www.vagrantup.com/.

  Vagrant + VMware
  
  Download and install from https://www.vagrantup.com/vmware/index.html
     
* update VMWare box

  Install by running: 
    
```bash
   vagrant box add bento/ubuntu-17.04
```
    
* run Virtual machine (VM)

  Install by running: 
  
```bash
    vagrant up --provider vmware_fusion
```

## Compiling rocksdb for Ubuntu

In order to use rockdb in Kubernetes we need rocksdb compiled static library in Docker container. Rather compiling
inside Docker which will make it large, we are building it here and the using only the librarty in a different container.

### Get the latest rockdb repositoty:

```bash
git clone https://github.com/facebook/rocksdb.git
```

```bash
vagrant ssh tcmaster01
```

Update packages:

```bash
vagrant@tcmaster01:~$ sudo apt-get update && sudo apt-get dist-upgrade
vagrant@tcmaster01:~$ sudo apt-get -y install make llvm g++ libgflags-dev libsnappy-dev  zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```

compiling RocksDB static library in release mode:

```bash
vagrant@tcmaster01:~$ cd /vagrant/rocksdb
vagrant@tcmaster01:/vagrant/rocksdb$ make static_lib 

...
vagrant@tcmaster01:/vagrant/rocksdb$ ar: creating librocksdb.a
```

The resulting RocksDB static library (343 MB) is in 
$HOME/Desktop/tc-ubuntu-xenial64-rocksdb/rocksdb/librocksdb.a 
folder in your host machine.

Install latest Golang:

```bash
cd ..
vagrant@tcmaster01:/vagrant$ sudo -s
root@tcmaster01:/vagrant$ snap install --classic go
root@tcmaster01:/vagrant$ exit
```

Check Golang version:

```bash
vagrant@tcmaster01:/vagrant$ go version
go version go1.8.3 linux/amd64
```

### Set the GOPATH environment variable

The GOPATH environment variable specifies the location of your workspace. 

```bash
export GOPATH=/vagrant/mygo
```

### Compile gorocksdb, a Go wrapper for RocksDB

```bash
vagrant@tcmaster01:~$ CGO_CFLAGS="-I/vagrant/rocksdb/include" \
CGO_LDFLAGS="-L/vagrant/rocksdb -lrocksdb -lstdc++ -lm -lz -lbz2 -lsnappy -llz4 -lzstd" \
  go get github.com/tecbot/gorocksdb
```
The generated library (1 MB) is stored on the Mac host:

$HOME/Desktop/tc-vmware-ubuntu-zestyzapus64-rocksdb/mygo/pkg/linux_amd64/github.com/tecbot/gorocksdb.a

```bash
vagrant@tcmaster01:~$ cd /vagrant/mygo/src/tc-rocksdb-list
vagrant@tcmaster01:/vagrant/mygo/src/tc-rocksdb-list$ cp /vagrant/rocksdb/librocksdb.a librocksdb.a

```
