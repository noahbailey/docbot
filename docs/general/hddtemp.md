# HDDTemp 

## SATA SSD/HDD

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

## NVME SSD

NVME SSD's do not work with HDDtemp or usual SMART tools correctly, since they use an entirely different bus and protocol. 

Instead, the nvme CLI tools must be installed: 

    sudo apt install nvme-cli

To query SMART data from the NVME SSD:

    sudo nvme smart-log /dev/nvme0

The output will look something like this: 

```
Smart Log for NVME device:nvme0 namespace-id:ffffffff
critical_warning			: 0
temperature				: 34 C
available_spare				: 100%
available_spare_threshold		: 10%
percentage_used				: 0%
endurance group critical warning summary: 0
data_units_read				: 8,780
data_units_written			: 135,654
host_read_commands			: 121,749
host_write_commands			: 1,784,910
controller_busy_time			: 5
power_cycles				: 4
power_on_hours				: 2
unsafe_shutdowns			: 0
media_errors				: 0
num_err_log_entries			: 14
Warning Temperature Time		: 0
Critical Composite Temperature Time	: 0
Temperature Sensor 1           : 34 C
Temperature Sensor 2           : 30 C
Thermal Management T1 Trans Count	: 0
Thermal Management T2 Trans Count	: 0
Thermal Management T1 Total Time	: 0
Thermal Management T2 Total Time	: 0
```