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

## Youtube Downloader

The `ocDownloader` NextCloud app is supposed to be working for HTTP and BT download once it is installed.

However, the **Youtube Downloader** feature of it may not work properly at the first time.
Note that **youtube-dl** is donwloaded to to `/usr/local/bin/youtube-dl` in the nextcloud image built from `./nextcloud/Dockerfile`.

This problem can be solved by (on NextCloud 18.04):

- go to the **Additional Settings** page of the admin's setting page
- clear the **youtube-dl** binary path, the page will say something like "can't find the youtube-dl binary"
- reset the **youtube-dl** binary path to `/usr/local/bin/youtube-dl`, the page will say something like "good"

It seems like it is just a bug in the **ocDownloader** that the default path is working properly, users need to
set the path at least once to make it work.
