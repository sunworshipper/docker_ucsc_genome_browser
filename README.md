### UCSC Genome Browser Docker Image

A minimal UCSC Genome Browser mirror.

http://genome.ucsc.edu/

### License
This is a Dockerized version of the UCSC Genome Browser source code. The license is the same as the UCSC Genome Browser itself. The source code and executables are freely available for academic, nonprofit and personal use. Commercial use requires purchase of a license with setup fee and annual payment. See https://genome-store.ucsc.edu/. 

### Download
```shell
docker pull icebert/ucsc_genome_browser

docker pull icebert/ucsc_genome_browser_db
```

### Demo Run
```shell
docker run -d --name gbdb -p 3338:3306 icebert/ucsc_genome_browser_db

docker run -d --link gbdb:gbdb -p 8038:80 icebert/ucsc_genome_browser
```

### A step by step setup guide for running the UCSC genome browser locally 
Note: The following is me hacking through icebert's original repo to get this to work.  

Assume local data is going to be stored in /my/data/path
You can change this to be your custom location but note that you will need to adjust the commands to reflect that path.

First copy the basic database files into /my/data/path from docker

```shell
docker run -d --name gbdb -p 3338:3306 icebert/ucsc_genome_browser_db

cd /my/data/path && docker cp gbdb:/data ./ && mv data/* ./ && rm -rf data

docker stop gbdb
```

Then put database files into /my/data/path. For example, mirror all the tracks of hg38 from ucsc genome browser

```shell
rm -rf /my/data/path/hg38

rsync -avP --delete --max-delete=20 rsync://hgdownload.soe.ucsc.edu/mysql/hg38 /my/data/path/
```
If you stop here you will get complaints so you also need hgFixed to be in the same folder
```shell
rsync -avP --delete --max-delete=20 rsync://hgdownload.soe.ucsc.edu/mysql/hgFixed /my/data/path/
```
The next error I got was related to missing maf files:

i.e. External file /gbdb/hg38/multiz100way/defaultMaf/chr1.maf cannot be opened or has wrong size. Old size 4890111046, new size -1, error No such file or directory

a bit of browsing through the UCSC ftp structure got me to the right place.  Looks like we are missing files from the /gbdb/ folder.

https://hgdownload.soe.ucsc.edu/gbdb/hg38/

now the question is, where does this gbdb folder need to live. Through some trial and error I found a gbdb folder on the container so I figured this is where this data should live.

For the next step you will need to get inside your container.

```shell
docker exec -it your-container-id-or-name /bin/bash
```
Here you will find the gbdb folder.
Get inside this folder and rsync the data from:

https://hgdownload.soe.ucsc.edu/gbdb/hg38/

to get it to work initially you only need the multiz100way and mane folders and knownGene.ix and knownGene.ixx files.



Finally start the database server and genome browser server

```shell
docker run -d --name gbdb -p 3338:3306 -v /my/data/path:/data icebert/ucsc_genome_browser_db

docker run -d --link gbdb:gbdb -p 8038:80 icebert/ucsc_genome_browser
```

### MySQL Access
The mysql server listens on port 3338. The default username for mysql is 'admin' with password 'admin'.

```shell
mysql -h 127.0.0.1 -P 3338 -u admin -p
```

