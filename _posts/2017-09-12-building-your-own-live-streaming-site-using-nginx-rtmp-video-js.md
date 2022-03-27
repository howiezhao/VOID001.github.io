---
title: Building your own live streaming site using Nginx RTMP & video.js
---
=================================================================



#####  [September 12, 2017](https://web.archive.org/web/20210514123546/https://void-shana.moe/linux/building-your-own-live-streaming-site-using-nginx-rtmp-video-js.html "10:27 pm") 
[VOID001](https://web.archive.org/web/20210514123546/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [1 comment](https://web.archive.org/web/20210514123546/https://void-shana.moe/linux/building-your-own-live-streaming-site-using-nginx-rtmp-video-js.html#comments)





As I said in [twitter](https://web.archive.org/web/20210514123546/https://twitter.com/VOID_133/status/906175711211167745) I will update my blog at least once a week, so now I am writing this week’s blog (Although this article doesn’t contain too much technical detail) I just built my personal live server, for the trail version on bilibili is expired. And I don’t want to send sensitive personal data to that platform, so I decided to build one on my own.


Previously I built a live stream service using my raspberry pi, and only use the most simple configuration of nginx, and it does not play very well. Now I have bought a shiny new VPS from CAT.NET with my partner onion, it’s awesomely fast and fluent, so I use this server to build my live stream service, including a frontend to play the stream.


The tutorial is here: [https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/](https://web.archive.org/web/20210514123546/https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/)


The following guide will show how to build one stream server on Archlinux (Yes, archilnux ONLY, but compatible with many other distro), Just follow the basic steps:


### Setting up the RTMP Streaming Server with HLS


1. Install `nginx-rtmp` from AUR (nginx-mainline + nginx-rtmp-module may also works, but I have problems when compiling the module using makepkg)
2. If you have previous nginx configuration, install nginx-rtmp will conflict with nginx, just remove it, no worry about the configuration file you wrote, it will be stored at /etc/nginx/nginx.conf.pacsave
3. If you have a previous installation of nginx, after install `nginx-rtmp`  exec the command `mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.old` and `mv /etc/nginx/nginx.conf.pacsave /etc/nginx/nginx.conf`
4. Then restart the nginx server and reload the daemon `systemctl daemon-reload`  && `systemctl restart nginx.conf`
5. This restart shouldn’t generate any error except you have error in your previous nginx config
6. Then **create the rtmp.conf file** (or whatever you name it)  example configuration can be found [here](https://web.archive.org/web/20210514123546/https://github.com/arut/nginx-rtmp-module/wiki/Examples)


Here is my rtmp.conf, remember to include it in nginx.conf OUTSIDE the http block like this:



```
http {
 ...
}

include rtmp.conf
```


```
rtmp {
        server {
                

                max\_connections 100;
                chunk\_size 4096;
                ping 30s;
                notify\_method get;

                application my\_live {
                        live on;
                        hls on;
                        hls\_path /tmp/hls;
                        hls\_fragment 3s;
                        hls\_playlist\_length 60s;

                }
        }
}

```

Then you can use [OBS](https://web.archive.org/web/20210514123546/https://obsproject.com/) or anything else to push to the livestream, in this example, you should push to



```
rtmp://example.com/my\_live
```

In the key field just write something e.g: test


Remember to `mkdir /tmp/hls` before using the live stream


 


### Setting up the Live Stream data service


We need hls.js or videojs with hls supported, I choose the latter one.


create a config in /etc/nginx/sites-available


e.g: live.void-shana.moe.conf


put these lines in the config file



```
server {
    listen 80;

    location /stream {
        # Disable cache
        add\_header Cache-Control no-cache;

        # CORS setup
        add\_header 'Access-Control-Allow-Origin' '*' always;
        add\_header 'Access-Control-Expose-Headers' 'Content-Length';

        # allow CORS preflight requests
        if ($request\_method = 'OPTIONS') {
            add\_header 'Access-Control-Allow-Origin' '*';
            add\_header 'Access-Control-Max-Age' 1728000;
            add\_header 'Content-Type' 'text/plain charset=UTF-8';
            add\_header 'Content-Length' 0;
            return 204;
        }

        types {
            application/vnd.apple.mpegurl m3u8;
            video/mp2t ts;
        }

        alias /tmp/hls;
    }
}
```

Then when you visit https://example.com/stream/test.m3u8 you will get the live stream playlist of live named “test”.


This *.m3u8 file is a text file that describes every segment to play (segments are named in <name>-<id>.ts format), here is a sample file



```
#EXTM3U         
#EXT-X-VERSION:3                 
#EXT-X-MEDIA-SEQUENCE:0          
#EXT-X-TARGETDURATION:8          
#EXT-X-DISCONTINUITY             
#EXTINF:8.333,  
test-0.ts       
#EXTINF:8.333,  
test-1.ts       
#EXTINF:8.334,  
test-2.ts
```

The videojs / hls.js will recognize format and parse it, fetch *.ts segments from server then play it one by one, making it looks like a live stream.


### Create a webpage to show it


I use videojs with hls support for this, just take a look at view-source://live.void-shana.moe/ you can get a video player that works.


See https://github.com/videojs/videojs-contrib-hls#getting-started for more details


 


### Screenshot


![](https://web.archive.org/web/20210514123546im_/https://void-shana.moe/wp-content/uploads/2017/09/Spectacle.E22359-1024x640.png)


### TODO


I didn’t find any credential configuration when setting up rtmp stream, this will make it dangerous when someone know my rtmp URI. Bad guy can push nasty live stream / video to my site, currently the URI is complex, and cannot be fetched from frontend (I only expose the hls interface). Future will try to support auth


 


If you have problems when setting up the live stream service & frontend, just feel free to comment below, I am glad to help






---


[archlinux](https://web.archive.org/web/20210514123546/https://void-shana.moe/category/linux/archlinux), [Linux](https://web.archive.org/web/20210514123546/https://void-shana.moe/category/linux) [rtmp](https://web.archive.org/web/20210514123546/https://void-shana.moe/tag/rtmp), [video](https://web.archive.org/web/20210514123546/https://void-shana.moe/tag/video) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210514123546im_/https://secure.gravatar.com/avatar/fe0faba4dbcdeaec3e701acf14cd47fc?s=50&d=identicon&r=g) **Mohammed** says: 
[April 26, 2018 at 8:00 pm](https://web.archive.org/web/20210514123546/https://void-shana.moe/linux/building-your-own-live-streaming-site-using-nginx-rtmp-video-js.html#comment-388)
thanks for the article .. i can help in auth


1. ![](https://web.archive.org/web/20210514123546im_/https://secure.gravatar.com/avatar/fe0faba4dbcdeaec3e701acf14cd47fc?s=50&d=identicon&r=g) **Mohammed** says: 
[April 26, 2018 at 8:00 pm](https://web.archive.org/web/20210514123546/https://void-shana.moe/linux/building-your-own-live-streaming-site-using-nginx-rtmp-video-js.html#comment-388)
thanks for the article .. i can help in auth

            