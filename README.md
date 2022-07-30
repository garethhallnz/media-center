# Overview of Docker bases media center

Sonarr / Radarr / Bazarr / Jackett / NZBGet / Transmission / Plex



## Monitor TV shows/movies with Sonarr and Radarr

[Sonarr](https://sonarr.tv/) and [Radarr](https://radarr.video) 
Will automatically take care of analyzing existing episodes, seasons and movies. It compares what you have on disk with the release schedule, and triggers download for missing files. It also takes care of upgrading your existing files if a better quality matching your criterias is available out there.

When the download is over the file to the appropriate location 
(`/tv/show-name/season-1/01-title.mp4`) or (`/movies/movie-name/title.mp4`) and renames the file if needed.


## Search for releases automatically with Usenet and torrent indexers

Sonarr and Radarr can both download files from Torrents


Files are searched automatically by Sonarr/Radarr through a list of _indexers_ that you have to configure. Indexers are APIs that allow searching for particular releases organized by categories. Think browsing the Pirate Bay programmatically. This is a pretty common feature for newsgroups indexers that respect a common API (called `Newznab`).
However this common protocol does not really exist for torrent indexers. That's why we'll be using another tool called [Jackett](https://github.com/Jackett/Jackett). You can consider it as a local proxy API for the most popular torrent indexers. It searches and parse information from heterogeneous websites.

The best release matching your criteria is selected by Sonarr/Radarr (eg. non-blacklisted 1080p release with enough seeds). Then the download is passed on to another set of tools.

## Handle bittorrent downloads with Tranmission

Sonarr and Radarr are plugged to downloaders for:

- [Tanmission](https://transmissionbt.com) handles torrent download.

Both are daemons coming with a nice Web UI, making them perfect candidates for being installed on a server. Sonarr & Radarr already have integration with them, meaning they rely on each service API to pass on downloads, request download status and handle finished downloads.

Both are very standard and popular tools. I'm using them for their integration with Sonarr/Radarr but also as standalone downloaders for everything else.


## Organize libraries and play videos with Plex

[Plex](https://www.plex.tv/) Media Server organize all your medias as libraries. You can set up one for TV shows and another one for movies.
It automatically grabs metadata for each new release (description, actors, images, release date).


Plex keeps track of your position in the entire library: what episode of a given TV show season you've watched, what movie you've not watched yet, what episode was added to the library since last time. It also remembers where you stopped within a video file. Basically you can pause a movie in your bedroom, then resume playback from another device in your bathroom.

Plex comes with [clients](https://www.plex.tv/apps/) in a lot of different systems (Web UI, Linux, Windows, OSX, iOS, Android, Android TV, Chromecast, PS4, Smart TV, etc.) that allow you to display and watch all your shows/movies in a nice Netflix-like UI.

The server has transcoding abilities: it automatically transcodes video quality if needed (eg. stream your 1080p movie in 480p if watched from a mobile with low bandwidth).


## Software stack

**Downloaders**:

- [Tranmission](https://transmissionbt.com): torrent downloader with a web UI
- [Jackett](https://github.com/Jackett/Jackett): API to search torrents from multiple indexers
- [Bazarr](https://www.bazarr.media/): A companion tool for Radarr and Sonarr which will automatically pull subtitles for all of your TV and movie downloads.

**Download orchestration**:

- [Sonarr](https://sonarr.tv): manage TV show, automatic downloads, sort & rename
- [Radarr](https://radarr.video): basically the same as Sonarr, but for movies

**Media Center**:

- [Plex](https://plex.tv): media center server with streaming transcoding features, useful plugins and a beautiful UI. Clients available for a lot of systems (Linux/OSX/Windows, Web, Android, Chromecast, Android TV, etc.)


## Installation guide

### Introduction

The idea is to set up all these components as Docker containers in a `docker-compose.yml` file.
We'll reuse community-maintained images (special thanks to [linuxserver.io](https://www.linuxserver.io/) for many of them).
I'm assuming you have some basic knowledge of Linux and Docker.

The stack is not really plug-and-play. You'll see that manual human configuration is required for most of these tools. Configuration is not fully automated (yet?), but is persisted on reboot. Some steps also depend on external accounts that you need to set up yourself (usenet indexers, torrent indexers, vpn server, plex account, etc.). We'll walk through it.


### Install docker and docker-compose

See the [official instructions](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce-1) to install Docker.


Also install docker-compose (see the [official instructions](https://docs.docker.com/compose/install/#install-compose)).

### (optional) Use premade docker-compose

This tutorial will guide you along the full process of making your own docker-compose file and configuring every app within it, however, to prevent errors or to reduce your typing, you can also use the general-purpose docker-compose file provided in this repository.

1. First, `git clone https://github.com/garethhallnz/media-center.git` into a directory. This is where you will run the full setup from (note: this isn't the same as your media directory). I you don't have git then just download and uncompress the zip file.
2. Rename the `.env.sample` file included in the repo to `.env`.
3. Continue this guide, and the docker-compose file snippets you see are already ready for you to use. You'll still need to manually configure your `.env` file and other manual configurations.


### Setup environment variables

For each of these images, there is some unique configuration that needs to be done. Instead of editing the docker-compose file to hardcode these values in, we'll instead put these values in a .env file. A .env file is a file for storing environment variables that can later be accessed in a general-purpose docker-compose.yml file, like the example one in this repository.

Here is an example of what your `.env` file should look like, use values that fit for your own setup.

```sh
# Your timezone, https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
TZ=Pacific/Auckland
PUID=1000
PGID=1000
# The directory where data and configuration will be stored.
ROOT=/media/my_user/storage/
```

Things to notice:

- TZ is based on your [tz time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
- The PUID and PGID are your user's ids. Find them with `id $USER`.
- This file should be in the same directory as your `docker-compose.yml` file so the values can be read in.

### Start up the containers
Then run the container with `docker-compose up -d`.

### Access Tranmission
Tranmission web UI is available on port 9091 [localhost:9091](http:localhost:9091)

### Access Jackett
Jackett web UI is available on port 9117 [localhost:9117](http:localhost:9117)

### Access NZBGet
NZBGet web UI is available on port 6789 [localhost:6789](http:localhost:6789)

### Access Plex
Plex web UI should be available at [localhost:32400/web](http:localhost:32400/web)

### Access/Setup Sonarr
Sonarr web UI is available on port 8989 [localhost:6789](http:localhost:8989)

In `Media Management`, you can choose to rename episodes automatically. This is a very nice feature I've been using for a long time.
In `profiles` you can set new quality profiles, default ones are fairly good. There is an important option at the bottom of the page: do you want to give priority to Usenet or Torrents for downloading episodes?

`Indexers` is the important tab: that's where Sonarr will grab information about released episodes.

For torrents indexers, I activate Torznab custom indexers that point to my local Jackett service. This allows searches across all torrent indexers configured in Jackett. You have to configure them one by one though.

Get torrent indexers Jackett proxy URLs by clicking `Copy Torznab Feed` in Jackett Web UI. Use the global Jackett API key as authentication.

`Download Clients` tab is where we'll configure links with our two download clients: Transmission and/or NZBGet.

Enable `Advanced Settings`, and tick `Remove` in the Completed Download Handling section. This tells Sonarr to remove torrents from tranmission once processed.

In `Connect` tab, we'll configure Sonarr to send notifications to Plex when a new episode is ready:

_Note: You may need to `chown -R $USER:$USER /path/to/root/directory` so Sonarr and the rest of the apps have the proper permissions to modify and move around files. This Docker image of Sonarr uses an internal user account inside the container called `abc` some you may have to set this user as owner of the directory where it will place the media files after download. This note also applies for Radarr._


### Access/Setup Radarr
Radarr web UI is available on port 7878 [localhost:7878](http:localhost:7878)

Radarr is a fork of Sonarr, made for movies instead of TV shows. So setup just like Sonarr


### Access/Setup Bazarr
Bazarr web UI is available on port 6767 [localhost:6767](http:localhost:6767)

Load Bazarr and you will be greeted with this setup page:

Click next and we will be on the Sonarr setup page. For this part we will need our Sonarr API key. To get this, open up sonarr in a separate tab and navigate to `Settings > General > Security` and copy the api key listed there.


Head back over Bazarr and check the "Use Sonarr" box and some settings will pop up. Paste your API key in the proper field, and you can leave the other options default. If you would like, you can tick the box for "Download Only Monitored" which will prevent Bazarr from downloading subtitles for tv shows you have in your Sonarr library but have possibly deleted from your drive. Then click "Test" and Sonarr should be all set!


The next step is connecting to Radarr and the process should be identical. The only difference is that you'll have to grab your Radarr API key instead of Sonarr. Once that's done click Finish and you will be brought to your main screen where you will be greeted with a message saying that you need to restart. Click this and Bazarr should reload. Once that's all set, you should be good to go! Bazarr should now automatically downlaod subtitles for the content you add through Radarr and Sonarr that is not already found within the media files themselves.


# *NOTES
One issue that might creep up is that docker will reserve the HDD mount before the OS mounts it. This is even true if you have specified the HDD to auto mount. Say your hd is called `hd1` and it's internal; that means it will be mounted at `/media/$USER/hd1` what happens is docker starts and mount attemps to mount `/media/$USER/hd1` (as specified in the .env file). Since the OS still has not mounted the HDD docker reserves the name and the OS mounts in as `/media/$USER/hd11` therefore the mount in docker is now incorrect.

To fix this is it's recommended to mount the HDD using fstab.

1. **Create a directory to mount your HDD to**
```bash
sudo mkdir /hd1
```

2. **Find you disk**
```bash
sudo fdisk -l
```
*You should see some like*
```
Disk /dev/sdb: 2.73 TiB, 3000592982016 bytes, 5860533168 sectors
Disk model: WDC WD30EFRX-68E
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: AD7443B6-2372-4D65-8451-875DC4AD9B25

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 5860532223 5860530176  2.7T Linux filesystem
```
Take note of the "Device" FYI you can also get this detail out of the "Disks" UI app.

3. **Add the device to fstab**
```bash
sudo vim /ets/fstab # can also use nano or vi
```
*And add following to the end of the file:*
```
/dev/sdb1    /hdd    ext4    defaults    0    0
```

4. *Mount the HDD*
```bash
sudo mount /hd1
```
