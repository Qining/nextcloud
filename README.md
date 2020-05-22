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
