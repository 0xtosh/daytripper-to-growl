# DayTripper To Phone Alerting

## What
Use a `DayTripper` to detect movement and send a Growl notification to your phone/watch/anywhere. What is `DayTripper`?  A board made by @dekuNukem for more info read: [https://github.com/dekuNukem/daytripper](https://github.com/dekuNukem/daytripper)

## How does this work
  - We use a keylogger to look at just the keystrokes of just our HID device on a dedicated computer e.g. Raspberry Pi.
  - The keylogger logs to a log file (it will always log the same key pattern)
  - We use the `when-changed` script to see if the file got bigger i.e. was another event triggered
  - Everytime `when-changed` sees a change it launches the Growl/Prowl wrapper script
  - The Prowl script sends notifications to notify of the event wherever you have a Growl compatible listener installed e.g. computer, smart mirror, phone, watch, etc.

## Requirements

### Prerequisites

  - Get a Raspberry Pi or dedicated computer and plug in the receiver USB controller / HID device side of the DayTripper
  - Get all the required packages and software:
```
sudo apt-get -y install build-essential autotools-dev autoconf kbd curl python-pip
pip install https://github.com/joh/when-changed/archive/master.zip
curl -o /usr/local/bin/prowl.pl  https://www.prowlapp.com/static/prowl.pl
```
  - Get a free API key from [https://www.prowlapp.com/](https://www.prowlapp.com/) and put the API key in `prowl_key.txt`.

### Configuration

  - Put the sender of the daytripper in `CUSTOM COMMAND` mode by setting the selecter to `CSTM` as seen [here](https://github.com/dekuNukem/daytripper/blob/master/resources/photos/rxback.jpg)
  - Download, compile and install logkeys
  ```
git clone https://github.com/kernc/logkeys.git
cd logkeys-master  && ./autogen.sh  && cd build  && ../configure && make && sudo -s  && make install
```
## Running

#### Make sure a destination is set i.e. API key and wrapper

Put the line below in `sendprowl.sh` and make sure it can find the API file `prowl_key.txt`
```
perl ./prowl.pl -apikeyfile=prowl_key.txt -event="DayTripper" -notification="TRIGGERED at $(date +"%Y-%m-%d %T")"
```
#### Run the keylogger

Run the keylogger on the right detected virtual keyboard with `logairkeys.sh` containing:

```
export DEV=$(find /sys/devices | grep -i 04B3:302 | grep -E 'event[0-9]$' | awk -F "/" '{print $1,$NF}' | tr -d ' ' | tr -d '\n')
logkeys -s -d /dev/input/$DEV -o logkeys.out
```
#### Run the listener

Run the listener that will detect any size changes of the output file of the specific HID with `listen.sh` which contains:
```
screen -S when-changed -d -m when-changed logkeys.out ./sendprowl.sh
```
