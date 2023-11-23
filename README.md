# random-stuff
Random Things to Keep

## Unifi Controller - Does not show up on the unifi.ui.com site and reemote access is enabled on the controller

Disable remote access in the UI

Look for issues with the connection to the Unifi site
```
# sudo tail -f /var/log/unifi/server.log
```
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
