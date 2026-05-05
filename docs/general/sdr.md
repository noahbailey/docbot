# Software Defined Radio

## Install libraries

    sudo apt install rtl-sdr librtlsdr0

## Install GUI program

### GQRX

Basic GUI program for listening to known frequencies. Just Works.

    sudo apt install gqrx-sdr

### SdrAngel

More advanced tool for scanning, digital signal decoding, etc. 

    sudo flatpak install org.sdrangel.SDRangel

### SSTV

Slow-Scan Television

    sudo apt install qsstv

## Install a CLI program

### SatDump

    sudo apt install satdump

### Download from GOES satellites

https://www.ospo.noaa.gov/operations/goes/hrit/index.html
https://usradioguy.com/receiving-goes-hrit-with-satdump/
https://www.blanchbyte.com/hunting-weather-satellites-my-diy-goes-19-antenna-build/
https://www.dishpointer.com/

GOES-18 (West)

    satdump live goes_hrit GOES_18 --source rtlsdr --samplerate 2.4e6 --frequency 1694.1e6 --gain 40 --fill_missing

GOES-19 (East)

    satdump live goes_hrit GOES_19 --source rtlsdr --samplerate 2.4e6 --frequency 1694.1e6 --gain 40 --fill_missing
