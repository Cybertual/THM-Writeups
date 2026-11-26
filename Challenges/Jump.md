
`Step 1: Scan the target using nmap`

lets try seeing if anonymous access to the FTP server is possible.

`Step 2:  Enumerate the FTP server and gain access to recon_user`

There are two directories `incoming` and `pub`.
`incoming` seems empty and `pub` has a `README.txt` file 

The `README.txt` file seems to be a notice

The notice says that files inside the `incoming` directory is executed. So I created a reverse shell inside a  file using nano and placed it inside the `incoming` directory:

and boom it executed and I got access to `recon_user` shell and the flag.
The dev_user directory was accessible and the flag was accessible too.

`Step 3:  Monitor_user flag`

So now we have to enumerate the file system and see if something interesting can be found.
We use the command-> `find / -user username 2>/dev/null` to find files and directories belonging to the `monitor_user`.
One of them turned out to be a script under `/opt` (which is for plugins and such and not part of default file system) named `deploy_helper.sh`.
So I just thought to append a reverse shell in that just like before
But permission denied so I looked for something else instead.

So under `/opt/dev/` we found `backup.sh` which is owned by `dev_user` so  we appended the reverse_shell payload which was used earlier into it and then on our listener we got the shell of `dev` user.

So now here we see a file under  `/opt/dev/bin/ps` and it seems to be writable by the `dev` user. So we just add a reverse shell in it and change the permission and then execute it and the listener will catch it and give you `monitor_user` shell.

`Step 4: Ops_user flag`

`sudo -l` tells us that `deploy.sh` can be run with sudo permission.
So when we cat `deploy.sh` we see that inside it execution of `deploy_helper.sh` is taking place. And `deploy_helper.sh` is writable by our user. So we add a reverse shell in it like always.
But during execution we ensure that `deploy.sh` is executed by the permission of `ops_user`.
-> `sudo -u ops_user ./deploy.sh ` and we catch the reverse shell of `ops_user` and retrieve the flag.

`Step 5: Root flag`

`sudo -l` tells us that `less` command can be executed by using `sudo`. So we used `gtfobins` to find priv esc vector for `less` binary and it seems to be a simple one. `less /etc/hosts` and then using the filter utility we do `!/bin/sh`. Initially I also tried the `LD_PRELOAD` method cause `env_keep` was enable but `gcc` was missing.
so now one issue that can be faced is that `less` doesn't give us the filter. This is cause tty might be missing. So we need to stabilize the shell first using the following set of commands.

After which we use the `less` command -> `sudo less /etc/hosts` and then `!/bin/sh` and then obtain root shell and get the flag.

Key takeaways->
1. Always check which files the current user can execute and also whether files executed by other users have written permission for current user.
2. Track what files are being executed inside a script and evaluate which ones have permission for the current user for adding reverse shell.
3. use `sudo -l` to check which binaries can be executed using root privilege.
