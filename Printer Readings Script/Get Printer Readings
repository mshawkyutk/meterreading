### This script will use the SNMP protocol on Konica Minolta printers to get black/white and color readings seperately, by querying printers in a file called "printers.txt" 
### Created by Mina Shawky June 24th, 2024

path=$(dirname "$0")

printfile=$path/printers.txt
meterfile=$path/meterreads.txt

# Check if there is no print file, if not then the script will generate one, prompt the user to fill it out, and exit
if ! test -f "$printfile"
then
	echo "\nError: Cannot find list of printers. \nGenerating new file called printers.txt..."
	sleep 1
	echo "\nPlease enter list of printers in newly generated file using this format: \nprinter.utk.edu\nprinter2.utk.edu\nprinter3.utk.edu"
	echo "" >$printfile
	exit 1
fi

# Check if meter read file exists, give user option to overwrite/append meter read to existing file
if test -f "$meterfile"
then
	read -p "
Meter read file already present, do you wish to overwrite? If not, script will add meter reads to existing meter read file. [Y/N] " response
	if [[ "$response" =~ ^([yY][eE][sS]|[yY])$ ]]
	then
		echo "Overwriting current meter read file.."
    	rm -rf "$meterfile"
	else
		echo "Appending meter reads to existing file.."
	fi
fi

# Reads list of printers from printers.txt and declares an array using list
printf "\n" >> $printfile
printlist=$(while read -r line
do
    printf '%s\n' "$line"
done < "$printfile")
declare -a arr=($printlist)

# List the meter readings for color pages, non-color printers will be marked with N/A
echo "Color:" >> $meterfile
for i in "${arr[@]}"
do
	echo "Getting color meter readings for $i.."
	model=$(snmpget -v1 -c public -O qv $i 1.3.6.1.2.1.25.3.2.1.3.1 2> /dev/null)
	modelv=$(echo "$model" | awk '{print $NF}')
	
	if [[ $modelv =~ 'c'|'C' ]]
	then
		clrc=$(snmpget -v1 -c public -O qv $i 1.3.6.1.4.1.18334.1.1.1.5.7.2.2.1.5.2.1 2> /dev/null) # Obtain color print information
		clrp=$(snmpget -v1 -c public -O qv $i 1.3.6.1.4.1.18334.1.1.1.5.7.2.2.1.5.2.2 2> /dev/null) # Obtain color copy information
		clrt=$((clrc + clrp))
	elif [[ $model =~ "KONICA MINOLTA" ]]
	then
		echo "$i is not a color printer, appending N/A to output..."
		clrt="N/A"
	else
		echo "Printer $i is offline, or is not a valid printer."
		clrt="OFFLINE"
	fi
	
	echo $clrt >> $meterfile
done

# List the meter readings for black and white pages
echo "\nBlack/White:" >> $meterfile
for i in "${arr[@]}"
do
	echo "Getting B/W meter readings for $i.."
	model=$(snmpget -v1 -c public -O qv $i 1.3.6.1.2.1.25.3.2.1.3.1 2> /dev/null)
	modelv=$(echo "$model" | awk '{print $NF}')
	
	# Check for B/W print+copy readings for color printers
	if [[ $modelv =~ 'c'|'C' ]]
	then
		bwc=$(snmpget -v1 -c  public -O qv $i 1.3.6.1.4.1.18334.1.1.1.5.7.2.2.1.5.1.1 2> /dev/null) # Obtain B/W print information
		bwp=$(snmpget -v1 -c  public -O qv $i 1.3.6.1.4.1.18334.1.1.1.5.7.2.2.1.5.1.2 2> /dev/null) # Obtain B/W copy information
		bwt=$((bwc + bwp)) # Add print+copy information
		if [[ $bwt -eq 0 ]] # For whatever reason, some color printers don't use the above OIDs to store B/W print+copy information, in that case, they'll use the other method
		then
			bwt=$(snmpget -v1 -c public -O qv $i 1.3.6.1.2.1.43.10.2.1.4.1.1)
		fi
	# Check for B/W print+copy readings for non-color printers
	elif [[ $model =~ "KONICA MINOLTA" ]]
	then
		bwt=$(snmpget -v1 -c public -O qv $i 1.3.6.1.2.1.43.10.2.1.4.1.1)
	# If printer does not have KONICA MINOLTA in the received model name, it is either offline or not a valid printer
	else
		echo "Printer $i is offline, or is not a valid printer."
		bwt="OFFLINE" # Append OFFLINE to the output text fil
	fi
	
	echo $bwt >> $meterfile
done

echo "\nMeter readings generated."
# Adds extra line to meter read file so it doesn't cut off printers
printf "\n" >> $meterfile