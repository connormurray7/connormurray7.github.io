---
layout: post
tags: project
title: Google Drive Script
---

[This script](https://github.com/connormurray7/google-drive-script) implements a launchd task on macOS (similar to cron) that opens Google Drive (or Dropbox, Box, etc) in a defined interval for some amount of time. The purpose of the application is to sync your data without having the syncing service always on and hurting your battery.

For example, this script opens up Google Drive every hour for three minutes (plenty of time to sync for what I work on). And then closes Google Drive.

### _Why I made this_

I use sync services like many people do, but my laptop is frequently not plugged in, and a lot of times these _always on_ sync services cause a big battery hit. To combat this I foolishly tried to manually open Google Drive every so often, but I found that I would regularly forget--unlike a computer... so I created a script that will do that for you.

### _.plist file_
The launchd daemon manager is an open-soure project that Apple uses in OS X (now macOS). Each process can be loaded with a .plist file that defines what the daemon will do. Below is the .plist that I created, which is based off the template from [launchd.info](http://launchd.info)

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://	www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
    	<key>Label</key>
    	<string>com.connormurray7.google_drive_script</string>

    	<key>ProgramArguments</key>
    	<array>
        	<string>/Users/<user_name>/g_drive.sh</string>
    	</array>

    	<key>StartInterval</key>
    	<integer>3600</integer>

    	<key>KeepAlive</key>
    	<false/>
	</dict>
	</plist>

Here notice that I placed by `g_drive.sh` in `/Users/<user_name>/g_drive.sh`. Also notice that the `StartInterval` is `3600`, so every 3600 seconds or 1 hour, that script is executed. Then the script looks like

	#!/bin/sh
	open /Applications/Google\ Drive.app
	sleep 180
	kill pgrep Google\ Drive

Which just opens Google Drive, waits 3 minutes, and closes it. Plenty of time for it to sync. To use another service just change it to whatever app you wish.

###_Installation_
Change the `user_name` field in the .plist to your username. Move the .plist file into `/Library/LaunchDaemons`, which is the folder for global daemons. Put the `g_drive.sh` in your home folder.  And then run the command

	launchctl load /Library/LaunchDaemons/com.connormurray7.google_drive_script.plist

That's it!
