PDB-REDO builder
================

This is a collection of Dockerfile's that can be used to create pdb-redo executables
that will run on CentOS 7 without any dependency.

They way this works is by creating docker images that can be used to run `configure` and `make`
for tools that need libpdb-redo and/or libcifpp.

Building the docker images
--------------------------
Of course you need docker. I'm new to this, so I think you need the latest version.

There are four docker files. You need to build them in order. I did this to make incremental development easier. You can of course flatten the final image yourself.

Before you can build the containers, you need to fetch certain datafiles. The first is the source package for ccp4, you can download that here:

[http://www.ccp4.ac.uk/download/index.php#os=src](http://www.ccp4.ac.uk/download/index.php#os=src)

Extract the tar file and place the resulting devtools folder in the same directory as the dockerfiles.

Then download the components.cif file from:

[ftp://ftp.wwpdb.org/pub/pdb/data/monomers/components.cif.gz](ftp://ftp.wwpdb.org/pub/pdb/data/monomers/components.cif.gz)

And put that file, as is, in the same directory.

Now, the steps to build a docker container are:

```
	sudo docker build -t gcc:v9.3 -f Dockerfile.gcc-9.3 .
	sudo docker build -t pdb-redo-boost:v1.75 -f Dockerfile.boost .
	sudo docker build -t pdb-redo-ccp4:v7.1 -f Dockerfile.ccp4 .
	sudo docker build -t pdb-redo-libpdb-redo:v1.0 -f Dockerfile.libpdb-redo . 
```

This will first build a decent C++ compiler followed by boost and then the CCP4 libraries that are required by PDB-REDO and finally the libcifpp and libpdb-redo packages. Also, this setup uses mrc to package data files as resources in the final executables.

Using the docker images
-----------------------

If you now want to build an executable that uses libpdb-redo or libcifpp, you can start a shell inside a docker container and build the tool.

Lets build density-fitness: 

```
cd /tmp/
git clone https://github.com/PDB-REDO/density-fitness.git
sudo docker run --rm -it -v /tmp/density-fitness/:/tmp/density-fitness pdb-redo-libpdb-redo:v1.0
```

This will start a session inside the docker container and will mount the density-fitness source directory inside that container. Now it is simply a matter of `configure` and `make`:

```
cd /tmp/density-fitness
./configure CXX=g++-9 --with-boost=/opt/boost-1.75/ \
	--enable-resources LDFLAGS=-static-libstdc++ \
	PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/ CCP4=/opt/ccp4
make
```
The result is an excutable that can run on CentOS.

Please note the flags I passed to configure. They are essential. I should think of a way to make this tool easier to use. But the important thing is, it works.