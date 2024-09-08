# Some Synology NAS Information

## NAS Access accross VLANs

Check the box with "enable multiple gateways" under "advanced settings" on the synology network settings.

## Access to NAS shares from NFS

- Hostname or IP:  # Set this to the host IP or the subnet range (x.x.x.x/24)
- Privilages:      # Typically Read/Write
- Squash:          # Typically "Map all users to admin" (avoids need to create user accounts, but opens access to everything from every one)
- Security:        # Typically "sys"
- Enable Async:    # Typically checked
- No privilaged:   # Checked sometimes ... why?
- All sub-folders: # Typically NOT checked
