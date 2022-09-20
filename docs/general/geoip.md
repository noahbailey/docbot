# GeoIP Database

## Install

Install the updater tool & the query command line tools

    sudo apt install geoipupdate geoip-bin mmdb-bin

## Config file

Before configuring the client, you must obtain a license from MaxMind. 

`/etc/GeoIP.conf`

```ini
AccountID 123456
LicenseKey 1q2w3e4r5t
EditionIDs GeoLite2-Country GeoLite2-City
```

## Download/Update database files

    sudo geoipupdate

The resulting files will be located at `/var/lib/GeoIP/`

## Query Database

The `mmdblookup` utility can query the local database files:

    mmdblookup  --file /var/lib/GeoIP/GeoLite2-Country.mmdb --ip 8.8.8.8

The output is a JSON-like format:

```
  {                       
    "continent": 
      {               
        "code":           
          "NA" <utf8_string>
        "geoname_id":       
          6255149 <uint32>
        "names": 
          {       
            "en":                          
              "North America" <utf8_string>
      }
    "country": 
      {        
        "geoname_id": 
          6252001 <uint32>
        "iso_code": 
          "US" <utf8_string>
        "names": 
          {
            "en": 
              "United States" <utf8_string>
          }
  }
```
