# Devolution Modem Emulation (Wii/Wii U)
Devolution is a useful tool for Wii consoles, even with the rise of Nintendont, especially for testing modem connections in _Phantasy Star Online_ for the GameCube. For instance, the _Phantasy Star Online Episode I&II Trial Edition_ requires the modem for online play and doesn't allow for the Broadband Adapter (BBA) to be used at all.

In order to test the _Trial Edition_ and modem connections on a private server without using DNS redirection and someone else's PPP emulator, you can either use newserv or install slirp and socat on a server or Raspberry Pi to do everything, whether that's running locally or on a VPS.

## Option 1: newserv
If you're hosting a [newserv](https://github.com/fuzziqersoftware/newserv) instance, you can easily enable a built-in PPP emulator Devolution can connect to.

1. Install `newserv` if not already present by [following instructions in the Setup section](https://github.com/fuzziqersoftware/newserv?tab=readme-ov-file#setup).
2. Edit your `config.json` file in the `system` folder.
   * If this file is not present, copy `config.sample.json` to `config.json`.
3. Find the field named `PPPRawListen` and adjust this field to match either your internal or external IP address and a port of your choice (for example, "192.168.1.100:63335")
   * Using the example above, your line will look like `"PPPRawListen": ["192.168.1.100:63335"],`
4. Save the configuration file, then start newserv.
5. With your IP address and port in mind, [use this tool to help generate the phone number to dial into from Devolution](https://shiftadeband.github.io/Devolution-slirp-Modem-Emulation-Guide/phone-number-generator.html).
6. Boot up your Wii and make sure it's connected to your network in some form. (Wired is preferred, but wireless will work.)
7. Launch the version of _Phantasy Star Online_ you'd like to test from Devolution, and select the 'Website' option from the main menu.
8. Depending if you've set your network settings up before, you'll either create or load a configuration on the memory card. If you're presented with the network setup menu, proceed, otherwise, press the 'Y' button and select 'Setup.'
9. While in the network settings menu, you'll only need to change the following options:
    * **Username:** Set this to anything -- it cannot be left blank.
    * **Password:** Set this to anything -- it cannot be left blank.
    * **Phone Number:** Fill in the phone number you generated in step 5, carefully checking that it's correct.
10. Save your configuration and select 'Return to game' from the menu bar by pressing 'Y'.
11. Back in _PSO_, attempt to connect to the server. 
    * If you get to the lobby select screen, you're set! 
    * If not, check steps above or see 'Troubleshooting' below.
    * If you'd like to add authentication using the username and password you set in step 8, see the next section.

## Option 2: socat and slirp
**Overview:** You'll be installing `socat` and `slirp`. socat will handle the incoming connections and passing them to slirp, and slirp will handle the PPP emulation.

If you follow this example exactly, you'll be using port `63335` for these connections. Make sure your firewall, if any is installed, allows connections to this port. Everything below is executed in the home folder, but you can set this up anywhere that's appropriate for you - just make sure you update your paths to adjust.

I would highly recommend using a wired connection for both the Raspberry Pi and Wii to improve the speed of the emulated dial up modem.

The instructions below were run locally on a Raspberry Pi (no desktop environment), but should be similar for any other distro or OS, except Windows.

1. Run `sudo apt install slirp socat -y`.
2. Run `ifconfig` and copy/take note of your private IP on the network interface you'll be using (or public IP if you're using a VPS/port forwarding).
3. Run `sudo nano /etc/hosts`.
    * In your hosts file, change the line your hostname is on from `[hostname] 127.0.1.1` to `[hostname] [ip]`, where `[hostname]` is your server's hostname, and `[ip]` is your private or public IP, depending on how you'll be connecting to your `slirp` instance.
    * When done editing the file, press `CTRL + X`, then type `y` and hit enter.
4. With your IP address and port in mind, [use this tool to help generate the phone number to dial into from Devolution](https://shiftadeband.github.io/Devolution-slirp-Modem-Emulation-Guide/phone-number-generator.html).
5. Boot up your Wii and make sure it's connected to your network in some form. (Wired is preferred, but wireless will work.)
6. Launch the version of _Phantasy Star Online_ you'd like to test from Devolution, and select the 'Website' option from the main menu.
7. Depending if you've set your network settings up before, you'll either create or load a configuration on the memory card. If you're presented with the network setup menu, proceed, otherwise, press the 'Y' button and select 'Setup.'
8. While in the network settings menu, you'll only need to change the following options:
    * **Username:** If you'd like to use authentication, it will be the username you'd like to authenticate with. Otherwise it can be anything, but it cannot be left blank.
    * **Password:** If you'd like to use authentication, it will be the password you'd like to authenticate with. Otherwise it can be anything, but it cannot be left blank.
    * **Phone Number:** Fill in the phone number you generated in step 4, carefully checking that it's correct.
9. Save your configuration and select 'Return to game' from the menu bar by pressing 'Y'.
10. Execute the command:
    * `sudo socat -d -d tcp-l:63335,reuseaddr,fork,keepalive,nodelay,keepcnt=5,keepidle=300,keepintvl=60 exec:'slirp -P -dppp nozeros \"dns [DNS SERVER]\"',pty,ctty,su-d=[USER] &`
    * Note: `[DNS Server]` should be set to your private server's IP address, e.g. `192.168.5.250`.
    * Note: `[USER]` should have root permissions, e.g. your current user if they have root, or by creating a new super user.
11. Back in _PSO_, attempt to connect to the server. 
    * If you get to the lobby select screen, you're set! 
    * If not, check steps above or see 'Troubleshooting' below.
    * If you'd like to add authentication using the username and password you set in step 8, see the next section.

### Using Authentication
If you plan to leave your slirp instance publicly accessible, I'd highly recommend using authentication. This is not much of an issue if you are not exposing your slirp instance to the world, but if you do expose it publicly, by default, anyone can connect using any combination of username and password. 

To add authentication:

1. In your user's home directory, create a file named `.chap-secrets`.
    * You can do this by executing the command `nano .chap-secrets`.
2. The format for this file will look like this:
    * `[username] [hostname] [password] 10.0.2.15`
    * Your username and password will be whatever you set in your _PSO_ network configuration.
    * Your hostname is the hostname of the server that `slirp` is running on.
    * Example: `DEVO slirp gc 10.0.2.15`
3. Save your .chap-secrets file.
4. If you have your `socat` command above still running, kill it using `sudo killall socat`.
5. Now, run the new command:
    * `sudo socat -d -d tcp-l:63335,reuseaddr,fork,keepalive,nodelay,keepcnt=5,keepidle=300,keepintvl=60 exec:'slirp -P -dppp +chap nozeros \"dns [DNS SERVER]\"',pty,ctty,su-d=[USER] &`

### Troubleshooting
If you're having trouble connecting at all, make sure you're allowing access to the port that `socat` is running on. Using the defaults above, that port is 63335. You can check that this port using a tool like `nc -vnzu [ip address] 63335` or `Test-NetConnection [ip address] -Port 63335`.

If your connection loads to the 'Connecting to DNS' screen but does not progress, make sure your `/etc/hosts` file is properly updated as outlined in step 3 of the 'How To' section.

If the network connection is slow, remember that the Wii/Devolution is simulating a dial-up connection, and the Wii's WiFi is pretty weak. You can improve these by adding a wired connection to the Wii, and making sure that your server is wired as well.

If you get a 'username or password is incorrect' message and you're using `.chap-secrets`, double check that you entered in the correct username and password in the network settings menu in PSO, and check that your `.chap-secrets` file is using the correct username, password, and hostname.

If you're having other issues and not sure where to start, check your slirp log in your home directory. This file is called `slirp_pppdebug` and can be opened using tail (run `tail -200 slirp_pppdebug`) or by opening it in nano (run `nano slirp_pppdebug`).
