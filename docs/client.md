# Ubuntu 24.04 Time Client Setup

These instructions setup Ubuntu 24.04 as a PTP client.

1. [Turn off NTP](#turn-off-ntp)
2. [PTP Client without Services](#ptp-client-without-services)
3. [PTP Client with Services](#ptp-client-with-services)
4. [Logging](#logging)
5. [Useful Links](#useful-links)

## Turn off NTP

The PPS signal at the server is not necessarily synchronized to a time standard. Therefore, the client should turn off NTP services as these may contradict the PTP master.

    sudo timedatectl set-ntp no

Verify NTP services are inactive:

    timedatectl

## PTP Client without Services

Install the necessary packages:

    sudo apt install linuxptp

Then run:

    sudo ptp4l -f ptp4l_client.conf -i enp4s0 -m
    sudo phc2sys -w -s enp4s0 -n 55 -m

The option `-n 55` is the domain and must match the server's domain.

## PTP Client with Services

Edit `/etc/linuxptp/ptp4l.conf` and change the following values:

    slaveOnly               1
    domainNumber            55

The value `domainNumber` must match the server's domain.

Start the service:

    sudo systemctl enable ptp4l@enp4s0
    sudo systemctl start ptp4l@enp4s0

To synchronize the system clock with the PHC, `phc2sys` is used. Edit `/lib/systemd/system/phc2sys@.service` lines to match:

    Requires=ptp4l@enp4s0.service
    After=ptp4l@enp4s0.service

    ExecStart=/usr/sbin/phc2sys -w -s %I -n 55

Start the service:

    sudo systemctl enable phc2sys@enp4s0
    sudo systemctl start phc2sys@enp4s0

## Logging

    journalctl -f -u ptp4l@enp4s0
    journalctl -f -u phc2sys@enp4s0

## Useful Links

[ptp4l High Values of Master offset freq and path delay](https://stackoverflow.com/questions/65949602/ptp4l-high-values-of-master-offset-freq-and-path-delay)
