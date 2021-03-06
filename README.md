**Fork of https://github.com/gfacciol/zotero_dataserver-docker**

------------------------------

**Look at https://github.com/SamuelHassine/zotero-prime which is more frequently maintained than this repo**

------------------------------

# Docker image for Zotero Data Server

This image was build following the instructions for installing a Zotero dataserver at (http://git.27o.de/dataserver/about/), which is an updated procedure of [this document](https://github.com/Panzerkampfwagen/dataserver/blob/master/misc/Zotero_Data_Server_Installation_Debian.pdf).


## Build the image

    docker build -t zotero .

The resulting image is configured to run a dataserver on https://localhost/.

To customize the installation the following files must be edited:
* SSL certificate: `apache/zotero.{cert,key}`. The current certificate is self-signed for localhost.
* Apache site config: `apache/sites-zotero.conf`. 
* Dataserver: `dataserver/config.inc.php` to match the site config.
* MySQL credentials/passwords: `mysql/setup\_db` and `dataserver/dbconnect.inc.php` accordingly.

The build procedure also creates a couple of test users using the user administration tools: test:test and test2:test2.


## Start the dataserver

    # 1st run. Named container simplifies the access (to the data) across runs
    docker run  -p 80:80 -p 443:443 --name=FOO -t -i zotero 
    # all the subsequent runs
    docker start FOO; docker attach FOO

This will start the dataserver on [https://localhost/](https://localhost/sync/login?version=9&username=test&password=test). Because of the self-signed certificate some browsers may refuse to connect to the server.


## Patch the standalone client to use the new dataserver

Following the procedure of (http://git.27o.de/dataserver/about/Zotero-Client.md).
Download the Zotero client, and change these two lines in `resource/config.js` inside the zotero.jar archive (zip)

    SYNC_URL: 'https://localhost/sync/',
    API_URL: 'https://localhost/',

If the server uses a self-signed certificate an exception should be added to the client. A `cert\_override.txt` file must be added to the user profile generated by zotero client:

    ~/Library/Application\ Support/Zotero/Profiles/<random>.default/      MAC
    ~/.zotero/Profiles/<random>.default/                                  Linux
    c:Users/<username>/AppData/Roaming/Zotero/Zotero/                     Win

The `cert\_override.txt` file can be generated with Firefox as explained here (https://groups.google.com/d/msg/zotero-dev/MEwLaptJIzI/PVDAFJiqEgAJ). The override file in this directory corresponds to the self-signed certificate in the apache directory.


## User administration

    cd /srv/zotero/dataserver/admin 
    ./add_user 101 testuser  testpassword
    ./add_user 102 testuser2 testpassword2
    ./add_group -o testuser -f members -r members -e members testgroup 
    ./add_groupuser testgroup testuser2 member 

add\_user is a patched version of the script from http://git.27o.de that allows to set the password from the command line.


