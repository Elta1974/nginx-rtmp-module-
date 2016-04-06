Setup Nginx-RTMP on Ubuntu 14.04
Modified on: Mon, Jun 29, 2015 at 11:16 am EST
Linux Guides Ubuntu Web Servers
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
Append the following.

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
After you've added the above, you can customize settings such a video bitrate, audio bitrate and resolution. These changes will only be applied to the lower quality stream. To add more qualities, copy and paste the exec ffmpeg line and change the settings. You'll also need to create a new application. You can do this by copying and pasting the live360 example that has been included. Don't forget to update the exec ffmpeg line with the address of the new application. You can do this by changing the final RTMP address in the exec ffmpeg line.

Note: Changing the value after -b:v will change the video bitrate. This is measured in kilobits per second. Changing the value after -b:a will change the audio bitrate. This is measured in kilobits per second. Changing the value after -s will change the resolution.
Save the file by pressing Control and X together. Restart Nginx.

sudo service nginx restart
Note: For best performance, each stream being converted should have its own CPU core. For example two qualities, 360P and 480P are being created from a 720P stream. A Vultr instance with at least two CPU cores should be used.
Security Note

If you're using a firewall, you'll need to make sure TCP 1935 is allowed.

The current configuartion allows anyone to stream to your server. We can fix this by only allowing certain IP addresses the publish permission. Open the Nginx configuration.

sudo nano /usr/local/nginx/conf/nginx.conf
Look for the following lines.

                live on;
                record off;
Add the following to each set of the above lines. Change 0.0.0.0 to your IP address.

                allow publish 127.0.0.1;
                allow publish 0.0.0.0;
                deny publish all;
The configuration should now look something like this.

rtmp {
    server {
            listen 1935;
            chunk_size 4096;

            application live {
                    live on;
                    record off;
                    allow publish 127.0.0.1;
                    allow publish 0.0.0.0;
                    deny publish all;
                    exec ffmpeg -i rtmp://localhost/live/$name -threads 1 -c:v libx264 -profile:v baseline -b:v 350K -s 640x360 -f flv -c:a aac -ac 1 -strict -2 -b:a 56k rtmp://localhost/live360p/$name;
            }
            application live360p {
                    live on;
                    record off;
                    allow publish 127.0.0.1;
                    allow publish 0.0.0.0;
                    deny publish all;
        }
    }
}
Save the file by pressing Control and X together. Restart Nginx.

sudo service nginx restart
Configuring Software to Work with Nginx-RTMP

Streaming applications typically have two fields for connection information. The first field is usually for the server information and the second field is usually for the stream name or key. The information that you should place into each field is listed. The stream name or key can be set to anything.

Field 1: rtmp://your.vultr.ip/live/
Field 2: stream-key-you-set
To view streams open the following links in a player supporting RTMP.

rtmp://your.vultr.ip/live/stream-key-you-set
rtmp://your.vultr.ip/live360p/stream-key-you-set
Setting up a player to display live video on a website is beyond the scope of this guide. Searching for the term 'RTMP web player' might assist you.


