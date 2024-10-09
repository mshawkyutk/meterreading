# Printer Readings Script

This script uses the SNMP protocol to generate a text file with the amount of successful prints completed on a printer, for the purpose of meter readings. It currently only works on Konica Minolta printers, but support for additional printers is planned in the future.

# Functionality

To run, the script requires a text file (named printers.txt) with a list of IPs for it to query, with each printer seperated by a newline. If no such text file is found, the script will generate one for the user and prompt them to fill it out before proceeding.

It will then go through each printer IP listed and check for its print reading, then generate and fill out the "meterreads.txt" file at the end, which is seperated by color and black and white columns to easily paste into a spreadsheet.

# Considerations

This script works for both color and black and white printers, and checks if they are offline. If a printer is black and white only, it will fill "N/A" in the Color section of the meterreads.txt file for continuity purposes. If a printer is offline, that specific printer will read "OFFLINE" in the meter reading to notify the user that it is down.
