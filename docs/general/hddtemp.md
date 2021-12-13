# HDDTemp 

Sometimes HDDtemp doesn't read the data correctly. 

First, check smartctl: 

```sh
$ sudo smartctl -a /dev/sda | grep Temp
190 Airflow_Temperature_Cel 0x0032   066   041   000    Old_age   Always       -       34
```

This means that the data is in field 190 of the SMART data. 

Next, get the full name of the device: 

```sh
sudo hddtemp /dev/sda --debug

================= hddtemp 0.3-beta15 ==================
Model: Samsung SSD 860 EVO 500G B
```

This data can be added to the hddtemp data file: 

Find the section for your manufacturer and add the line in this format: 

    <device model> <smartctl field> <unit c/f> <comment>

Example: `/etc/hddtemp.db`

    "Samsung SSD 860 EVO 500G B" 190 C "Samsung 850 Evo 500GB"

If this is done correctly, the output will be fixed: 

```
$ sudo hddtemp /dev/sda 
/dev/sda: Samsung SSD 860 EVO 500G B: 34°C
```
