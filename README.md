# How to setup and host fastapi project on ubuntu server

* connect to server via ssh
*  run updates & install pip and venv

```bash
sudo apt-get update
sudo apt-get upgrade

sudo apt-get -y install python3-pip
sudo apt-get -y install python3-venv
```

* clone repo, create / activate virtual environment & install reqs

```bash
git clone https://github.com/username/repo_name
cd repo_name
python3 -m venv venv
source venv/bin/activate
sudo chmod -R a+rwx venv
```

pip install gunicorn uvicorn

/var/tiles/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000

Install all packages as requested
```bash
pip install fastapi
pip install numpy 
pip install pillow
pip install httpx
pip install diskcache
pip install sqlalchemy
pip install shapely
pip install geoalchemy2
pip install asyncpg
pip install requests
pip install scipy
pip install mercantile
```

* Install nginx

```bash
sudo apt-get -y install nginx
```

* update nginx configuration to proxy to localhost

```bash
sudo vi /etc/nginx/sites-available/default

```

* look for this block in the file

```
location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}
```

* update it to look like the following,update the port to match the one uvicorn or gunicorn is running on.

```
location / {
    include proxy_params;                         
    proxy_pass http://127.0.0.1:8000;
}
```

* Restart nginx

```bash
sudo systemctl restart nginx

uvicorn main:app

```
* confirm you can open the root of the site via the IP address
  

* create service for project
note - remove {} and change the contents in {} to your values

```bash
sudo vi /etc/systemd/system/tiles.service
```

* paste the following
* NB! Make sure username is correct

```
[Unit]
Description=Gunicorn instance to serve MyApp

[Service]
User=root
WorkingDirectory=/var/tiles
Environment="PATH=var/tiles/venv/bin"
ExecStart=/var/tiles/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

* start the service 
```bash
sudo systemctl daemon-reload
sudo systemctl start tiles.service
sudo systemctl enable tiles.service


sudo apt install python3-certbot-nginx

sudo certbot --nginx -d {domain}
sudo certbot --nginx -d {subdomain}.{domain}.fr

# to stop the server
sudo systemctl stop tiles.service
```

* Done.
