
Step 1: Scan the target and resolve the host

Upon scanning the target I found two open ports-> ssh and http
intially I tried to access the website but it indicated that the page was not found or can't be reached.
I even tried searching for an exploit related to the ssh and http versions respectively
The issue was the ip resolved to a domain called 'olympus.thm' and it had to be locally resolved
so in the `/etc/hosts/` file an entry was to be made: <ip> olympus.thm
Now the website was accessible

Step 2: Directory Enumeration

The website didn't give away much from the content that was displayed so the classic approach in such challenges would be to try to bruteforce for endpoints or basically perform `directory bruteforcing`.
so I initially used the IP of the target for the url which again created an issue as when Vhosts or Virtual hosts are present then an ip address can point to multiple servers so the host will be used for directing traffic
to the correct server.
The following command I ran: `gobuster dir -u http://olympus.thm -w /usr/share/wordlists/dirb/common.txt`  Note: if the wordlist in the `dirbuster` directory does not work, the `common.txt` can be tried as well.
One endpoint looked weird it started with "~" so I tried accessing it

Step 3: SQL Injection

The webpage looked like it had a search functionality along with filters and login form plus nothing else seemed interesting

In the search functionality I entered ` ' ` and an error was generated which indicated towards some mysql syntax error, I see... SQL injection it is...

Now for post requests you have to have a copy of the request header in a file which you can get through the `Network tab` of dev tools for the request or use BurpSuite and save the request

I also saw a GET parameter was present but when trying out ` ' ` it didn't return anything just blank, but I ended up trying that using `SQLMap` and for some reason it worked but it was `boolean based blind sql 
Injection`.

The intended path was POST though, but anyways I still decided to proceed as it was vulnerable and then I ended up dumping the database, as seen shown below

 ![sql_databases:](Images/Sql_db.png)

 The olympus database seems interesting so lets dumps all of its tables- > `sqlmap -u http://olympus.thm/~webmaster/category.php?cat_id=1 -D olympus --tables`

 So I see that it has a table called flag so obv that is where we found the first flag -> `sqlmap -u http://olympus.thm/~webmaster/category.php?cat_id=1 -D olympus -T flag --dump`

 Step 4: Further Enumeration of the database

 So I see a table called users so decided to view that and found that it has the names,roles,email,salt and finally the hashed password for some users.

 Prometheus didn't have any salt so I was like lets try his password first, through research(AI lol) I found that the hash format was bcrypt.

 So now either John or Hashcat could be used with most likely `rockyou.txt` to crack the password, I used hashcat-> `hashcat -a 0 -m  3200 <hash_file> rockyou.txt`

 Ok the password is `summertime` but WHERE DO WE APPLY THAT!

 Step 5: Hidden Subdomain

 So looks like the password is for some login form so I remembered initially that `~` endpoint had a login form so we could try it there. After trying it we could login and alot of contents were present but nothing useful as our rights were low.

 So had to do a bit of researching(things aren't always that easy) and turned out the `chat.olympus.thm` which was found in the emails for other users except Prometheus was a subdomain, Like WOW that was a bit unexpected

Step 6: Web Shell Upload

So I forgot to mention that after we logged in as prometheus in the `~` endpoint for the `olympus.thm` domain we did find an upload feature but it wasn't really working when a web shell or a reverse shell was uploaded.

But after we log in using the user prometheus and the password we cracked for the subdomain `chat.olympus.thm` we see that some chats are present and they are talking about some files that are being uploaded but inorder to find the file or execute them basically the pathname to access them is hard or unpredictable cause some weird function is being used. I see...

Previous knowledge is important for connecting things, The chats endpoint


 

 

