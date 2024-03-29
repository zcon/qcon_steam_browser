# README #

This was the first version of the quakecon server browser.  
I do not plan on updating this any more because I would like to work on v2.  
In v2 i plan to:
- clean everything up
- use the steamcondenser gem instead of GameQ
- use angular / json to update the server list instead of page refreshes.


If you do not want this to run in a subdirectory, you will need to edit the following files:

>app/assets/javascripts/servers.js  
>change /servers/servers #servers to /servers #servers

>config/environments/production.rb  
>comment out or delete config.relative_url_root = "/servers"


### tasks ###

rake update:servers updates the server list  
rake update:protocols checks gameq for any new protocols and adds them. (for use after you git fetch/pull gameq)   
rake update:networks updates servers network info (for updating existing servers when a new network is added)  
rake update:seats updates seat information from quakecon.org  
rake cleanup:servers removes old servers from the database  

### Here are the quick and dirty instructions to get this up and running: ###

clone this repo. in this example i have it cloned in ~/Development/qcon_steam_browser

> git clone https://github.com/wdeasy/qcon_steam_browser.git ~/Development/qcon_steam_browser

install ruby on rails  
> \curl -sSL https://get.rvm.io | bash -s stable --rails

install postgres  
> sudo apt-get install postgresql libpq-dev

create the qconservers role in postgres
>sudo su postgres  
>psql  
>create role sb_user with createdb login password 'passgoeshere';  
>\q

edit postgres config to allow md5 auth for local
> sudo nano /etc/postgresql/9.X/main/pg_hba.conf

change
> \#"local" is for Unix domain socket connections only  
> local   all             all                                     peer

to
> \#"local" is for Unix domain socket connections only  
> local   all             all                                     md5

>sudo service postgresql restart

create default gemset and bundle install
> cd ~/Development/qcon_steam_browser  
> rvm gemset create qcon_steam_browser  
> rvm use ruby-2.2.1@qcon_steam_browser --default  
> bundle install

create environment variables
>sudo nano ~/.profile

>export STEAM_WEB_API_KEY="steam web api key"  
>export SECRET_KEY_BASE="passenger secret key base"     
>export GAMEQ_PATH="../GameQ/"  
>export HOSTNAME="localhost"  
>export DATABASE="database name"  
>export USERNAME="database user"  
>export PASSWORD="database pass"   
>export MAILGUN_URL="mailgun api url"  
>export MAILGUN_API="mailgun api key"  
>export MAILGUN_DOMAIN="mailgun domain"  
>export MAILGUN_EMAIL="mailgun email"  

install php (used by gameq)
> sudo apt-get install php5 php5-pgsql

clone gameq to the same directory that qcon_steam_browser is in
>cd ~/Development   
>git clone https://github.com/Austinb/GameQ.git ~/Development/GameQ


edit db/seeds.rb and add your admin users and groups  
the group_list array needs group steam id, group steam name and group url  
the user_list array needs user steamid, name, url, avatar, and admin bool  

setup the database
> cd ~/Development/qcon_steam_browser  
> rake db:setup

run rake update:servers to see if it works  
run rails server, navigate to localhost:3000 and see if it works

run whenever and copy into cron
>cd ~/Development/qcon_steam_browser  
>whenever  
>crontab -e

nginx / passenger instructions

install nginx
>sudo apt-get install nginx

install passenger
>sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 561F9B9CAC40B2F7    
>sudo nano /etc/apt/sources.list.d/passenger.list  

paste & save this:   
>deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main

>sudo chown root: /etc/apt/sources.list.d/passenger.list  
>sudo chmod 600 /etc/apt/sources.list.d/passenger.list

>sudo apt-get install nginx-extras passenger

>sudo nano /etc/nginx/nginx.conf

uncomment passenger_root and passenger_ruby  
passenger_root should be the output of passenger-config --root  
passenger_ruby should be the output of which ruby   

setup vhosts file
> nano /etc/nginx/sites-available/qcon_steam_browser

>server {  
>  listen 80;  
>  server_name qconsteambrowser-dev;  

>  location /servers {  
>    alias /home/user/Development/qcon_steam_browser/public$1;  
>    passenger_base_uri /servers;  
>    passenger_app_root /home/ray/Development/qcon_steam_browser;  
>    passenger_document_root /home/ray/Development/qcon_steam_browser/public;  
>    passenger_ruby /home/user/.rvm/wrappers/ruby-2.2.1@qcon_steam_browser/ruby;   
>    passenger_app_env development;   
>    passenger_enabled on;   
>  }  
>}  

>sudo ln -s /etc/nginx/sites-available/qcon_steam_browser /etc/nginx/sites-enabled/qcon_steam_browser  

edit /etc/hosts  
>sudo nano /etc/hosts  

add and save  
>127.0.0.1	qconsteambrowser-dev

>sudo service nginx restart  
>rake assets:clobber  
>rake assets:precompile  

once everything is set up, you should be able to sign in through steam with the user you put in the seeds.rb file and see athe admin bar.





