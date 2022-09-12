# DayTripper To Phone Alerting

## What
Use a `DayTripper` to detect movement and send a Growl notification to your phone/watch/anywhere. What is daytripper?  Read: [https://github.com/dekuNukem/daytripper](https://github.com/dekuNukem/daytripper)

## How does this work
We use a keylogger to look at just the keystrokes of just our HID device on a dedicated computer e.g. Raspberry Pi.

The keylogger logs to a log file (it will always log the same key pattern)

We use the `when-changed` script to see if the file got bigger i.e. was another event triggered

Everytime `when-changed` sees a change it launches the Growl/Prowl wrapper script
Prowl script sends notification to notify of event

## Requirements

### Prerequisites
Get all the required packages and software

```
apt-get -y install build-essential autotools-dev autoconf kbd curl python-pip
pip install https://github.com/joh/when-changed/archive/master.zip
curl -o /usr/local/bin/prowl.pl  https://www.prowlapp.com/static/prowl.pl
```
### API Key

Get a free API key from [https://www.prowlapp.com/](https://www.prowlapp.com/) and put the API key in `prowl_key.txt`.

### Configuration

  - Put the sender of the daytripper in `CUSTOM COMMAND` mode by setting the selecter to `CSTM` as seen [here](https://github.com/dekuNukem/daytripper/blob/master/resources/photos/rxback.jpg)

  - Put the line below in sendprowl.sh somewhere to be referenced by listen.sh and make sure it can find the API file
```
perl ./prowl.pl -apikeyfile=prowl_key.txt -event="DayTripper" -notification="TRIGGERED at $(date +"%Y-%m-%d %T")"
```
  - Download, compile and install logkeys
  ```
git clone https://github.com/kernc/logkeys.git
cd logkeys-master  && ./autogen.sh  && cd build  && ../configure && make && sudo -s  && make install
```
## Running

#### Run the keylogger

Run the keylogger on the right detected virtual keyboard:

```./logairkeys.sh
export DEV=$(find /sys/devices | grep -i 04B3:302 | grep -E 'event[0-9]$' | awk -F "/" '{print $1,$NF}' | tr -d ' ' | tr -d '\n')
logkeys -s -d /dev/input/$DEV -o logkeys.out
```
#### Run the listener

```
./listen.sh
screen -S when-changed -d -m when-changed logkeys.out ./sendprowl.sh
```
