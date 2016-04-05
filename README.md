# nginx-rtmp-module-

RTMP is great for serving live content. When RTMP is paired with FFmpeg, streams can be converted into various qualities. Vultr is great for these applications as they provide fast and dedicated CPU performance. Vultr’s global network also ensures that you can deliver high quality live content with minimal delay. Lets get started!

Installing Nginx and Nginx-RTMP

Install the tools required to compile Nginx and Nginx-RTMP from source.

sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev
Make a working directory and switch to it.

mkdir ~/working
cd ~/working
Download the Nginx and Nginx-RTMP source.

wget http://nginx.org/download/nginx-1.7.5.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
Install the Unzip package.

sudo apt-get install unzip
Extract the Nginx and Nginx-RTMP source.

tar -zxvf nginx-1.7.5.tar.gz
unzip master.zip
Switch to the Nginx directory.

cd nginx-1.7.5
Add modules that Nginx will be compiled with. Nginx-RTMP is included.

./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master
Compile and install Nginx with Nginx-RTMP.

make
sudo make install
Install the Nginx init scripts.

sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
sudo update-rc.d nginx defaults
Start and stop Nginx to generate configuration files.

sudo service nginx start
sudo service nginx stop
Installing FFmpeg

Add the FFmpeg PPA.

sudo add-apt-repository ppa:kirillshkrogalev/ffmpeg-next
Update the package lists.

sudo apt-get update
Install FFmpeg.

sudo apt-get install ffmpeg
Note: The apt-add-repository command may not be installed in some cases. To install it run sudo apt-get install software-properties-common.
Configuring Nginx-RTMP and FFmpeg

Open the Nginx configuration file.

sudo nano /usr/local/nginx/conf/nginx.conf
rtmp {
    server {
            listen 1935;
            chunk_size 4096;

            application live {
                    live on;
                    record off;
                    exec ffmpeg -i rtmp://localhost/live/$name -threads 1 -c:v libx264 -profile:v baseline -b:v 350K -s 640x360 -f flv -c:a aac -ac 1 -strict -2 -b:a 56k rtmp://localhost/live360p/$name;
            }
            application live360p {
                    live on;
                    record off;
        }
    }
}
sudo service nginx restart





