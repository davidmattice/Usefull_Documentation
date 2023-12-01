# Unifi

This was my experiance when migrating my controller from a Raspberry Pi (7.3.83) to a Firewalla Docker container (7.5.176) in November of 2023.

## Migrating Self Hosted Controller to a new Device

1. If using remote access, verify it is working first (see connection issue below)
2. Download a backup file from the active conroller
3. Install and start the new controller (see optional docker instructions below)
4. Restore the backup file (from the login screen)
5. Verify devices show up in the UI on the new controller, but are offline
6. Verify the new controller shows on remote access site (if remote access is enabled) - unifi.ui.com
7. On the OLD controller go to "Setting -> System -> General" and click **Export Site**
8. Download the export file (for reference), but there is no need to import it, the backup that was restored had everything needed
9. Enter the IP (or DNS name) of the new controller and select the the devices to migrate (all of them) - **Clicking continue will cause a brief outage**
10. Verify all of the devices are visible in the new controller and are online
11. Shutdown the old controller

References
- [How-to Migrate Unifi Controller](https://lazyadmin.nl/home-network/migrate-unifi-controller/)
- [Similar process, but keeping the old IP on the new controller](https://community.ui.com/questions/How-to-migrate-UniFi-Controller-from-one-host-to-another/1c4bb6b7-f9b2-4628-8903-5b9a09cc5294#answer/ef467264-86c3-4f49-99f7-9b9af95182dc)
- [Setting up on a Google Compute VM](https://metis.fi/en/2018/02/unifi-on-gcp/)

## Migrating to a Docker Container (on a firewalla)

The steps are the same as above, but there are some [specific instructions for step 3](https://help.firewalla.com/hc/en-us/articles/360053441074-Guide-How-to-run-UniFi-Controller-on-the-Firewalla-Gold-Series-Boxes)

References
- [Scripts for firewalla](https://github.com/mbierman/unifi-installer-for-Firewalla/tree/main)
- [Latest Docker Images](https://hub.docker.com/r/jacobalberty/unifi/tags)

## Unifi Controller - Does not show up on the unifi.ui.com site with remote access enabled

Look at the server logs for issues with the connection to the Unifi site
```
# sudo tail -f /var/log/unifi/server.log
```
If you see connecting lost errors, disable remote access from the UI and see if they stop.

Updated the MongoDB to remove the cloud access key
```
# sudo mongo —-port 27117
use ace
db.setting.find({"key" : "super_cloudaccess"}).forEach(printjson)
db.setting.remove({"key" : "super_cloudaccess"})
exit
```
Restart the unifi controller
```
systemctl restart unifi
```

Re-enable remote accesss in the UI
