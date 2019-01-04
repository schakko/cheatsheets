# Mount directories with correct options to prevent file permission issues

	$ vim /etc/wsl.conf
	[automount]
	root = /
	options = "metadata"

and restart

# Have same home directory in Windows and WSL

	$ vim /etc/passwod
	# change /home/${USERNAME} to /c/Users/${USERNAME}
