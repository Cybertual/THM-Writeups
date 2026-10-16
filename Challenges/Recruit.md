Step 1: Scan the Target

`nmap -p- -Pn <ip> -T4` reveals that three ports are open with services being `ssh,dns and http`
Lets try to visit the website and see what we have in it.

Step 2: Enumerate the website

So the webpage has a login form and we don't know the credentials, putting `'` doesn't give any specific or generic error so let's assume its not vulnerable to SQL Injection, at least at this stage.
We click on `Access API` button which is below. So we are directed to `api.php` and  it mentions something about fetching CV from external sources during recruitment process.
And an endpoint is given with a parameter where we can put CV to access it. Also it is mentioned below that requesting restricted location on target is not allowed or blocked by API.
[!API](/Images/API_restriction.png)

Step 3: Directory Bruteforcing

Upon Bruteforcing for directories we found 2 things that could be interesting, first being `/phpmyadmin` and second being `/mail` so when we tried to check `/mail` we see that it has an email from HR and it seems the username for Login for the HR is `hr` and the credentials are stored in some `config.php` file so this tells us we can get it if they forgot to hide this file.
Admin credentials are stored securely in the database.
API documentation is live, so we can try to find that as well.

So now either we can try to brute force the login or look for API endpoints.

Step 4: Bruteforcing Login Page and `config.php`
So No interesting API endpoints seems to exist so lets try Bruteforcing the Login
and wow 16 valid password, interesting.....
But again false positives so none of them worked. SAD.. :(

When we tried accessing `Config.php` we get a blank page but as we know this file exists inside the system `locally` and `/file?cv=URL` this endpoint when tried to access says it requires a local file so we tried to access the Config.php file using this endpoint but as the file is locally stored we need to use the `file` protocol so it becomes -> `/file?cv=file://config.php`
and boom we get the `password` of the `hr` user. Also Worth nothing is that API being enabled is again hinted towards.
And after logging in we receive the User flag.

Step 5: Exploiting SQL Injection and retrieving the last flag

So infront of us we see candidate applications and such records. Search bar allows us to search for a candidate and we can see the value reflect into a query parameter on top of the URL. after typing `'` we confirm that it is vulnerable to SQL Injection. So either manually or using Sqlmap the vulnerability can be exploited. I decided to do it manually but using Sqlmap is simple too.

Ok so the username and password is found.
and we log in using the credentials and get the final flag.

The challenge was mostly simple if such challenges are solved before just the access to the `config.php` file was something to think and connect.





