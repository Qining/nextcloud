![Docker Image CI](https://github.com/Qining/nextcloud/workflows/Docker%20Image%20CI/badge.svg?branch=master)

# Use `docker-compose`

- Fill in the dot.env and rename to .env
- `docker-compose up -d`
- To shutdown: `docker-compose down`
- After installed (admin created?), we need to add two more config to make it work behind proxy
  - `docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set overwriteprotocol --value="https" --type=string`
  - `docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set overwritehost --value="https" --type=string`
- Nextcloud DB optimization as suggested in the setting overview page
  - `docker exec --user www-data <nextcloud_app_container_name> php occ db:convert-filecache-bigint`
  - `docker exec --user www-data <nextcloud_app_container_name> php occ db:add-missing-indices`

## YouTube Downloader

The `ocDownloader` NextCloud app is supposed to be working for HTTP and BT download once it is installed.

However, the **YouTube Downloader** feature of it may not work properly at the first time.
Note that **youtube-dl** is donwloaded to to `/usr/local/bin/youtube-dl` in the nextcloud image built from `./nextcloud/Dockerfile`.

This problem can be solved by (on NextCloud 18.04):

- go to the **Additional Settings** page of the admin's setting page
- clear the **youtube-dl** binary path, the page will say something like "can't find the youtube-dl binary"
- reset the **youtube-dl** binary path to `/usr/local/bin/youtube-dl`, the page will say something like "good"

It seems like it is just a bug in the **ocDownloader** that the default path is working properly, users need to
set the path at least once to make it work.

## Set Session Lifetime
```shell
# Disable keep-alive
docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set \
  session_keepalive --value=false --type=bool

# 10 Minute Lifetime
docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set \
  session_lifetime --value=600 --type=integer
```

## Enable Preview Generation
```shell
# Explicitly enable preview
docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set \
  enable_previews --value="true" --type=bool
  
# Explicitly list the providers to use
docker exec --user www-data <nextcloud_app_container_name> php occ config:system:set \
  enabledPreviewProvider <index> --value="OC\Preview\<provider>" --type=string
```
Indices and Provider
- 0: PNG
- 1: JPEG
- 2: GIF
- 3: HEIC
- 4: BMP
- 5: XBitmap
- 6: MP3
- 7: TXT
- 8: MarkDown
- 9: Movie
- 10: PDF
- 11: MSOfficeDoc
- 12: MSOffice2003
- 13: MSOffice2007
- 14: SVG
- 15: Font

## Pre-Generate Preview
It is also important to pre-generate previews for all existing data. Otherwise when using
apps like *Photos*, IO congestion can happen due to large amount of preview generation
tasks running at the same time.

1. Install this app: https://apps.nextcloud.com/apps/previewgenerator
1. Run the pre-generate command: `docker exec --user www-data <nextcloud_container_name> occ php preview:generate-all -vvv`.
Note this may run a long time, depends on how many existing files are there.
1. Schdule preview generation timer (below)

[The official document](https://nextcloud.com/blog/setup-cron-or-systemd-timers-for-the-nextcloud-preview-generator/).
Note the official document is not designed for NextCloud on *docker* specifically.

## Add Scheduled Preview Generation Task:
```shell
# login to the shell of nextcloud container
docker exec -it <nextcloud_container_name> bash
# use the 'crontab' in the busybox, with user: www-data
busybox crontab -e -u www-data
# In the opened file, add (assume the scheduled time is 4AM):
0 4 * * * php occ preview:pre-generate
```
