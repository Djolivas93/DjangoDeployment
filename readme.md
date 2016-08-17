# Django Deployment V1 
# Step 1
----
##  Build a project!
## > django-admin startproject projectName
## > cd projectname
## > touch .gitignore or nul> .gitignore (PCs)
### In your .gitignore: add *.pyc and venv/
## > git init
## > git add .
## > git commit -m "my first commit"
### Build the rest of your project, commit as necessary.
### NOTE WHERE THE git init occurred: INSIDE THE OUTER FOLDER, AT THE SAME LEVEL AS manage.py
# Step 2
----
### Go to github
### DO NOT CREATE A README but do:
### create repository on github
# Step 3
----
### return to terminal
## paste the code generated by github to your terminal (last two lines, adding a remote repository to your project)
## these might look like this:
> git remote add origin https://github.com/MikeHannon/myProject.git
> git push -u origin master

# Step 4
----
### Keep working on your project and adding and committing as necessary.
## When finished (and after git add . and git commit -m "yay project complete"):
## > pip freeze > requirements.txt
## > git add .
## > git commit -m "add python dependencies"
### merge any branches and all that usual stuff.
## > git push origin master
# Step 5
----
### If you go to a new directory and clone this project
### e.g.
## > git clone https://github.com/MikeHannon/myProject.git
### you will end up with an outer folder named myProject and if you cd into that project you'd see manage.py and whatever we named our project (in this example projectName)
# Step 6
----
### On to AWS!
### Make an AWS account!
## Its free for a year, so long as you don't have more than 1 instance up at a time! <-- and keep your instances in the free tier!
## Login to AWS
## launch a new instance (From: EC2 Dashboard) (midsized blue? button in the middle of the page)
## Select a linux ubuntu server (its like the 4th one down on most launch page)
## click the review and launch button on the following page
### you aren't done here yet!
## Set up your security; http port 80 should be anywhere 0.0.0.8 (you can set https too if you want)
## ssh should be myIP address (this has to be updated everytime you change IP addresses, it adds a level of security, but can be annoying)
## click launch, it will ask you to create a key.  Do so and download this to a folder(or move it to a folder), called keys(or whatever you want to call it) THIS FOLDER SHOULD NOT BE IN YOUR PROJECT NOR ANYWHERE THAT GETS PUSHED TO GITHUB, but could be anywhere else.  I have mine at Documents/Keys!
## Back in AWS finish clicking launch and it should give you a EC2 launching instance.
## Click on your EC2 instance.  A blue button at the top saying connect will show up, Click that, it has some helpful stuff
# Step 7
----
### Back to your terminal:
### navigate to your keys folder
For me:
## > cd
## > cd Documents/keys
### Follow those directions given by the blue button on aws (chmod the key file you downloaded before)
### Run the connect to ubuntu example code (which is generally the actual bash command )
# PCs: use a bash terminal or putty to do this!
# If all goes well, you will now be in a linux box in the cloud!! Yay you.  basically the only chnage you will see though is:
## ubunutu14>  // this is your command prompt in terminal, but you are working in a linux box, trust me... no more PC/MAC confusion!
# Step 8
----
# Now we are going to set up our linux box for deployment.
## > sudo apt-get update
## > sudo apt-get install python-pip python-dev libpq-dev postgresql postgresql-contrib nginx git
### These ones above install pip and some other python libraries a postGRES database (which we'll use, ngnix, we'll talk about that shortly and git)
## > sudo pip install virtualenv
## Git Clone your project from github: e.g.
## > git clone https://github.com/MikeHannon/myProject.git
### Navigate into this project and
## >ls
# If you don't see manage.py as one of the files, stop. and check out the setting up github/git pieces from before.
### Ok if everything looks good: (venv is just short, and is just a virtualenvironment like the ones we've been using!)
## > virtualenv venv
## > source venv/bin/activate
## (venv)> pip install -r requirements.txt
## (venv)> pip install gunicorn
## (venv)> pip install psycopg2
# Step 9
---
### Navigate into your mainproject file (where settings.py is) for example:
## > cd projectName
## > sudo nano settings.py
### Change the following things in settings:
``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myProject', # you can put whatever you want here
        'USER': 'mikehannon' # you can put whatever you want here,
        'PASSWORD': 'passwordYO', # you can put whatever you want here
        'HOST': 'localhost',
        'PORT': '',
    }
}
and ADD:
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```
## control-X and save
### What we just did is set up our settings.py for postgres daabase and added a place for all of our static files.
# Step 10
---
### Now we are going to set up our postgres DB
## > sudo su - postgres
## > psql
## > newPrompt# CREATE DATABASE myProject; (matches the name in the DATABASES)
## > newPrompt# CREATE USER mikehannon WITH PASSWORD 'passwordYO'; (username is not in quotes, but password is!, and both should match what you put into settings.
## > newPrompt# GRANT ALL PRIVILEGES ON DATABASE myProject TO mikehannon;
## > newPrompt# \q # quits this prompt
## > exit
# Step 11
---
# Great we are really close to  being done!
### You might have to navigate back to the folder that holds manage.py (cd ..)
## (venv)> python manage.py collectstatic (say yes)
## (venv)> python manage.py makemigrations
## (venv)> python manage.py migrate
### Take a look at your prokect in github, next to settings.py there is a wsgi.py file that's what this next line is referring to, so it should be foldername.wsgi
## (venv)> gunicorn --bind 0.0.0.0:8000 projectName.wsgi:application
### Did gunicorn start? (If not, check the names!!!)
## ctrl-c
## (venv)> deactivate
# Step 12
---
### Next we are basically going to tell this green unicorn (gunicorn) service to start a virtualenv and navigate to our proect and start our project for us all behind the scenes.
## > sudo nano /etc/init/gunicorn.conf
```
description "Gunicorn application server handling our project"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
setuid ubuntu
setgid www-data
chdir /home/ubuntu/myProject
exec venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/myProject/projectName.sock projectName.wsgi:application
```
### Breaking this down: the runlevels are system configuration bytes... just use 2,3,4,5 as stated on the start, stop
### respawn this says if the project stops restart it.
### setuid just says ubuntu can use this project
### setgid establishes a group
### chdir says go to the /home/ubuntu/ #### YOUR PROJECT FOLDER
### exec venv/bin/gunicorn .. this says execute gunicorn in your virtualenv where you pip installed it!
### and futhermore  bind some workers to it (in your project folder and activate the wsgi file in your main project folder, look at these names carefully!!!! )
## To turn on or off this process: > sudo service gunicorn start or stop
## turn it on for now!
# Step 13
---
## > sudo nano /etc/nginx/sites-available/myProject
```
server {
    listen 80;
    server_name yourEC2.public.ip.here; # This should be juts the digits from AWS public ip!
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/myProject;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/myProject/proojectName.sock;
    }
}
```
ctrl-x and save
## > sudo ln -s /etc/nginx/sites-available/myProject /etc/nginx/sites-enabled
## > sudo nginx -t
### Take a really careful look at everything that is in that file, compare these names to the gunicorn names, and what you actual are using!
## > sudo service nginx restart
### If you get an ok, hopefully you are rockin and rollin' and your app is deployed, go to the public domain! from aws and see if you see your app!
