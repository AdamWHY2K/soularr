![banner](https://raw.githubusercontent.com/mrusse/soularr/refs/heads/main/resources/banner.png)

<h1 align="center">Soularr</h1>
<p align="center">
  A Python script that connects Lidarr and Readarr with Soulseek!
</p>

<p align="center">
  <a href="https://discord.gg/EznhgYBayN">
    <img src="https://img.shields.io/discord/1292895470301220894?label=Discord&logo=discord&style=for-the-badge&cacheSeconds=60" alt="Join our Discord">
  </a>
</p>

# About

Soularr reads all of your "wanted" albums/artists from Lidarr and books/authors from Readarr, then downloads them using Slskd. It uses the libraries: [pyarr](https://github.com/totaldebug/pyarr) and [slskd-api](https://github.com/bigoulours/slskd-python-api) to make this happen. View the demo below!

![Soularr_small](https://github.com/user-attachments/assets/15c47a82-ddf2-40e3-b143-2ad7f570730f)


After the downloads are complete in Slskd the script will tell the relevent arr application to import the downloaded files, making it a truly hands off process.
# Setup

### Install and configure Lidarr, Readarr and Slskd

**Lidarr**
[https://lidarr.audio/](https://lidarr.audio/)

**Readarr**
[https://readarr.com/](https://readarr.com/)

Make sure Lidarr and Readarr can see your Slskd download directory, if you are running them in a Docker container you may need to mount the directory. You will then need add it to your config (see "download_dir" under "Lidarr" in the example config).

**Slskd**
[https://github.com/slskd/slskd](https://github.com/slskd/slskd)

The script requires an api key from Slskd. Take a look at their [docs](https://github.com/slskd/slskd/blob/master/docs/config.md#authentication) on how to set it up (all you have to do is add it to the yml file under `web, authentication, api_keys, my_api_key`).

### Configure your config file:

The config file has a bunch of different settings that affect how the script runs. Any lists in the config such as "accepted_countries" need to be comma separated with no spaces (e.g. `","` not `" , "` or `" ,"`).

**Example config:**

```ini
[Lidarr]
api_key = yourlidarrapikeygoeshere
host_url = http://localhost:8686
#This should be the path mounted in lidarr that points to your slskd download directory.
#If Lidarr is not running in Docker then this may just be the same dir as Slskd is using below.
download_dir = /lidarr/path/to/slskd/downloads
enabled = True

[Readarr]
api_key = yourreadarrapikeygoeshere
host_url = http://localhost:8787
download_dir = /readarr/path/to/slskd/downloads
enabled = True

[Slskd]
#Api key from Slskd. Need to set this up manually. See link to Slskd docs above.
api_key = yourslskdapikeygoeshere
host_url = http://localhost:5030
#Slskd download directory. Should have set it up when installing Slskd.
download_dir = /path/to/your/Slskd/downloads

[Release Settings]
#Selects the release with the most common amount of tracks out of all the releases.
use_most_common_tracknum = True
allow_multi_disc = True
#See full list of countries below.
accepted_countries = Europe,Japan,United Kingdom,United States,[Worldwide],Australia,Canada
#See full list of formats below.
accepted_formats = CD,Digital Media,Vinyl

[Search Settings]
search_timeout = 5000
maximum_peer_queue = 50
#Min upload speed in bit/s
minimum_peer_upload_speed = 0
#Replace "flac,mp3" with "flac" if you just want flacs.
allowed_filetypes = flac,mp3
readarr_allowed_filetypes = epub,mobi
ignored_users = User1,User2,Fred,Bob
#Set to False if you only want to search for complete albums
search_for_tracks = True
#Set to True if you want to add the creator's name to the beginning of the search for albums/books
album_prepend_artist = False
book_prepend_author = False
#Valid search types: all || incrementing_page || first_page
  #"all" will search for every wanted record everytime soularr is run.
  #"incrementing_page" will start with the first page and increment to the next on each run.
  #"first_page" will repeatedly search the first page.
#If using the search type "first_page" remove_wanted_on_failure should be enabled.
search_type = incrementing_page
#How mancy records to grab each run, must be a number between 1 - 2,147,483,647
number_of_albums_to_grab = 10
number_of_books_to_grab = 10
#Unmonitors the album if Soularr can't find it and places it in "failure_list.txt". 
#Failed albums can be re monitored by filtering "Unmonitored" in the Lidarr wanted list.
remove_wanted_on_failure = False
#Comma separated list of words that can't be in the title of albums or tracks. Case insensitive.
title_blacklist = BlacklistWord1,blacklistword2
readarr_title_blacklist = BlacklistWord1,blacklistword2
```

[Full list of countries from Musicbrainz.](https://musicbrainz.org/doc/Release/Country)

[Full list of formats (also from Musicbrainz but for some reason they don't have a nice list)](https://pastebin.com/raw/pzGVUgaE)


I have included this [example config](https://github.com/mrusse/soularr/blob/main/config.ini) in the repo.


## Docker

The best way to run the script is through Docker. A Docker image is available through [dockerhub](https://hub.docker.com/r/mrusse08/soularr).

Example docker run command:
```shell
docker run -d \
  --name soularr \                           
  --restart unless-stopped \                 
  --hostname soularr \                       
  -e PUID=1000 \                            
  -e PGID=1000 \                            
  -e TZ=Etc/UTC \                           
  -e SCRIPT_INTERVAL=300 \                  
  -v /path/to/slskd/downloads:/downloads \   
  -v /path/to/config/dir:/data \
  --user 1000:1000 \         
  mrusse08/soularr:latest
```

Or you can also set it up with the provided [Docker Compose](https://github.com/mrusse/soularr/blob/main/docker-compose.yml).
```yml
version: "3"
services:
  soularr:
    restart: unless-stopped
    container_name: soularr
    hostname: soularr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      #Script interval in seconds
      - SCRIPT_INTERVAL=300
    user: "1000:1000"
    volumes:
      #"You can set /downloads to whatever you want but will then need to change the Slskd download dir in your config file"
      - /path/to/slskd/downloads:/downloads
      #Select where you are storing your config file. 
      #Leave "/data" since thats where the script expects the config file to be
      - /path/to/config/dir:/data
    image: mrusse08/soularr:latest
```

Note: You **must** edit both volumes in the docker compose above. 

- `/path/to/slskd/downloads:/downloads`

  + This is where you put your Slskd downloads path.

  + You can point it to whatever dir you want but make sure to put the same dir in your config file under `[Slskd] -> download_dir`.

  + For example you could leave it as `/downloads` then in your config your entry would be `download_dir = /downloads`.

- `/path/to/config/dir:/data`

  + This is where put the path you are storing your config file. It must point to `/data`.

You can also edit `SCRIPT_INTERVAL` to choose how often (in seconds) you want the script to run (default is every 300 seconds). Another thing to note is that by default the user perms are set to PUID:1000 and PGID:1000. If you wish to edit this change `user: "1000:1000"` in the Docker compose to whatever you prefer.

## Running Manually

Install the requirements:
```
python -m pip install -r requirements.txt
```

You can simply run the script with:
```
python soularr.py
```
Note: the `config.ini` file needs to be in the same directory as `soularr.py`.

### Scheduling the script:

Even if you are not using Docker you can still schedule the script. I have included an example bash script below that can be scheduled using a [cron job](https://crontab.guru/every-5-minutes).

```bash
#!/bin/bash
cd /path/to/soularr/python/script

dt=$(date '+%d/%m/%Y %H:%M:%S');
echo "Starting Soularr! $dt"

if ps aux | grep "[s]oularr.py" > /dev/null; then
    echo "Soularr is already running. Exiting..."
else
    python soularr.py
fi
```

**Example cron job setup:**

Edit crontab file with 

```
crontab -e
```

Then enter in your schedule followed by the command. For example:

```
*/5 * * * * /path/to/run.sh
``` 

This would run the bash script every 5 minutes.

All of this is focused on Linux but the Python script runs fine on Windows as well. You can use things like the [Windows Task Scheduler](https://en.wikipedia.org/wiki/Windows_Task_Scheduler) to perform similar scheduling operations.

##
<p align="center">
  <a href='https://ko-fi.com/adamwhy2k' target='_blank'><img height='35' style='border:0px;height:46px;' src='https://az743702.vo.msecnd.net/cdn/kofi3.png?v=0' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>
</p>
