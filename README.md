PDB-REDO builder
================

This is a collection of Dockerfile's that can be used to create pdb-redo executables
that will run on CentOS 7 without any dependency.

They way this works is by creating docker images that can be used to run `configure` and `make`
for tools that need libpdb-redo and/or libcifpp.

Building the docker images
--------------------------
Of course you need docker. I'm new to this, so I think you need the latest version.

There are two docker files. You need to build them in order. I did this to make incremental development easier. You can of course flatten the final image yourself.

Before you can build the containers, you need to fetch certain datafiles. The first is the source package for ccp4, you can download that here:

[http://www.ccp4.ac.uk/download/index.php#os=src](http://www.ccp4.ac.uk/download/index.php#os=src)

Extract the tar file and place the resulting devtools folder in the same directory as the dockerfiles.

Then download the components.cif file from:

[ftp://ftp.wwpdb.org/pub/pdb/data/monomers/components.cif.gz](ftp://ftp.wwpdb.org/pub/pdb/data/monomers/components.cif.gz)

And put that file, as is, in the same directory.

Now, the steps to build a docker container are:

```
	sudo docker build -t pdb-redo-builder-base:v1 -f Dockerfile.base .
	sudo docker build -t pdb-redo-builder:v1 -f Dockerfile.build . 
```

This will first build a decent C++ compiler followed by boost and then the CCP4 libraries that are required by PDB-REDO.
The second docker image will contain the libcifpp and libpdb-redo packages. Also, this setup uses mrc to package data files as resources in the final executables.

Using the docker images
-----------------------

If you now want to build an executable that uses libpdb-redo or libcifpp, you can start a shell inside a docker container and build the tool.

Lets build density-fitness: 

```
cd /tmp/
git clone https://github.com/PDB-REDO/density-fitness.git
cd density-fitness
sudo docker run --rm -v $(pwd):/tmp/build pdb-redo-builder:v1
```
This will run a `configure` and `make` sequence generating an executable that can be transferred to a CentOS machine.
Please note that the default command assumes you've mounted your local directory to `/tmp/build`.

If you prefer to work on the code interatively, you might want to start the docker container in interactive mode:

```
sudo docker run --rm -it -v $(pwd):/tmp/build pdb-redo-builder:v1
```

This will start a session inside the docker container and will mount the density-fitness source directory inside that container. Now it is simply a matter of `configure` and `make`:

```
cd /tmp/build
./configure --with-boost=/opt/boost-1.75/ --enable-resources
make
```
Again, the result is an excutable that can run on CentOS. In this shell the environment has been prepared to contain:

```
ENV LDFLAGS=-static-libstdc++ CXX=g++-9 CCP4=/opt/ccp4 PKG_CONFIG_PATH=/usr/local/lib/pkgconfig CLIBD=$CCP4/lib/data
```

Please note the two flags I passed to configure. They are essential.
