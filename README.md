# graylog

This repository is made to help anyone who whants to get into Graylog.

## World Map

You must create an account on [dev.maxming.com](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data?lang=en) to be able to download the GeoLite2-City.mmdb. Once you subscribed, you'll receive a mail that will help you go through the process.
Go to the download page and the Geo2Lite City GZIP Binary on your computer.

<aside>
💡 This should be done on all Graylog servers !
</aside>

Upload that archive to your Graylog servers
```bash
scp GeoLite2-City_*.tar.gz root@server:/etc/graylog/server/
```

Rename the archive then move it to where it's supposed to be.

```bash
tar -xvf /etc/graylog/server/GeoLite2-City_*.tar.gz -C /etc/graylog/server/
rm /etc/graylog/server/GeoLite2-City_*.tar.gz
mv /etc/graylog/server/GeoLite2-City_*/ /etc/graylog/server/
```

## On the Graylog UI

### Lookup Table

#### Data Adapter

- System
    - Lookup Tables
        - Data Adapters
            - Create Data Adapter
![alt text](https://raw.githubusercontent.com/nerd-in-a-mooc/graylog/main/images/data_adapter.png)
- **Data Adapter Type**
Geo IP - MaxMind™ Databases
- **Title**
GeoIP - Data Adapter
- **Description**
Geo IP Data Adapter
- **Name**
Should be automatically generated
- **Custom Error TTL**
Nothing to do here
- **File path**
`/etc/graylog/server/GeoLite2-City.mmdb`
- **Database Type**
City Database
- **Refresh File**
Nothing to do here

#### Lookup Cache

- System
    - Lookup Tables
        - Caches
            - Create Cache

- **Cache Type**
Node-local, in-memory cache
- **Title**
GeoIP - Cache
- **Decription**
Geoip Cache.
- **Name**
geoip-cache
- **Expire after access**
60 minutes
- **Expire after write**
Nothing to do here

### Lookup Tables

- System
    - Lookup Tables
        - Lookup Tables
            - Create Lookup Table

- **Title**
GeoIP - Lookup Table
- **Description**
GeoIP Lookup Table
- **Name**
geoip-lookup-table
- **Data Adapter**
geoip-data-adapter
- **Cache**
geoip-cache

### Pipelines

- System
    - Piplines
        - Manage Rules
            - Create Rule

#### Destination IP

- **Title**
GeoIP for destination IP
- **Rule source**
    
```bash
rule "GeoIP | destination_ip"
when
	has_field("destination_ip")
	AND NOT 
    ( // exclude internal IPs
		has_field("destination_ip_is_internal")
		AND
		to_bool($message.destination_ip_is_internal)
    )
then
	let geo = lookup("geoip-lookup", to_string($message.destination_ip));
	set_field("destination_ip_geolocation", geo["coordinates"]);
	set_field("destination_ip_geo_country_code", geo["country"].iso_code);
	set_field("destination_ip_geo_country_name", geo["country"].names.en);
	set_field("destination_ip_geo_city_name", geo["city"].names.en);
end
```

#### Source IP

- **Title**
GeoIP for source IP
- **Rule Source**
    
```bash
rule "GeoIP | source_ip"
when
	has_field("source_ip")
	AND NOT 
	  ( // exclude internal IPs
		has_field("source_ip_is_internal")
		AND
		to_bool($message.source_ip_is_internal)
    )
then
	let geo = lookup("geoip-lookup", to_string($message.source_ip));
	set_field("source_ip_geolocation", geo["coordinates"]);
	set_field("source_ip_geo_country_code", geo["country"].iso_code);
	set_field("source_ip_geo_country_name", geo["country"].names.en);
	set_field("source_ip_geo_city_name", geo["city"].names.en);
end
```

#### Pipeline New Stage

- System
    - Pipelines
        - Manage Pipelines
            - Add New Stage

- **Stage**
1
- **Continue processing on next stage when**
Doesn't matter, there is no other stage.
- **Stage rules**
GeoIP | source_ip
GeoIP | destination_ip

**Is it working ?**

# Dashboard

## source_ip_geolocation

Give it a cool and clear title such as "Source IP" !
![alt text](https://raw.githubusercontent.com/nerd-in-a-mooc/graylog/main/images/geoip_source_ip_geolocation_map.png)


### Rollup Columns
	- Field : `source_ip_geolocation`
	- Limit : 15 (You can change that value : it will change the way the dots are aggregated on the map
	
### Metrics
	- Function : Count

### Visualization
	- Type : World Map

Zoom, latitude and longitude are automatically set when you move around the map with your mouse.


### destination_ip_geolocation

Duplicate the widget, change the title and change the field : `source_ip_geolocation` for `destination_ip_geolocation`

# Sources

- [How to set up Graylog GeoIP Configuration](https://www.graylog.org/post/how-to-set-up-graylog-geoip-configuration)
- [GeoIP does not work](https://community.graylog.org/t/graylog-4-0-5-geoip-does-not-work/19916/8)
