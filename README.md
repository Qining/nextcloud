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
