# Redshift

## Install

    sudo apt install redshift redshift-gtk

## Configure

I don't like Geoclue, so I prefer to set a hardcoded location in `.config/redshift.conf`

```ini
[redshift]
location-provider=manual

[manual]
lat=47.6671322
lon=-68.9887822
```
