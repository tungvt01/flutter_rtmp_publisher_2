# rtmp_publisher

RTMP streaming and camera plugin.

## Getting Started

This plugin is a fork of [flutter_rtmp_publisher](https://github.com/nhancv/flutter_rtmp_publisher)

This plugin is an extension of the flutter 
[camera plugin](https://pub.dev/packages/camera) to add in
rtmp streaming as part of the system.  It works on android and iOS
(but not web).

This means the API Is exactly the same as the camera and 
installation requirements are the same.  The different is there
is an extra API that is startStreaming(url) that takes an rtmp
url and starts streaming to that specific url.

For android I use [rtmp-rtsp-stream-client-java](https://github.com/pedroSG94/rtmp-rtsp-stream-client-java) 
and for iOS I use 
[HaishinKit.swift](https://github.com/shogo4405/HaishinKit.swift)
   
## Features:

* Display live camera preview in a widget.
* Snapshots can be captured and saved to a file.
* Record video.
* Add access to the image stream from Dart.

## Installation

First, add `camera` as a [dependency in your pubspec.yaml file](https://flutter.io/using-packages/).

### iOS

Add two rows to the `ios/Runner/Info.plist`:

* one with the key `Privacy - Camera Usage Description` and a usage description.
* and one with the key `Privacy - Microphone Usage Description` and a usage description.

Or in text format add the key:

```
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
<key>NSPhotoLibraryUsageDescription</key>
<string></string>
<key>UIBackgroundModes</key>
<array>
    <string>processing</string>
</array>
<key>NSCameraUsageDescription</key>
<string>App requires access to the camera for live streaming feature.</string>
<key>NSMicrophoneUsageDescription</key>
<string>App requires access to the microphone for live streaming feature.</string>
```

### Android

Change the minimum Android sdk version to 21 (or higher) in your `android/app/build.gradle` file.

```
minSdkVersion 21
```

Need to add in a section to the packaging options to exclude a file, or gradle will error on building.

```
packagingOptions {
   exclude 'project.clj'
}
```

### Example

[example code](https://github.com/nhancv/flutter_rtmp_publisher/tree/master/example)

-----
# Install RTMP Server on Ubuntu 18.04

https://sites.google.com/view/facebook-rtmp-to-rtmps/home
https://www.scaleway.com/en/docs/setup-rtmp-streaming-server/
https://topdev.vn/blog/streaming-media-voi-nginx-va-nginx-rtmp-module/

- RTMP publisher: https://obsproject.com/
- RTMP client player: https://www.videolan.org/vlc/index.html
- RTMP server: https://github.com/arut/nginx-rtmp-module
```
sudo apt update
sudo apt install build-essential libpcre3 libpcre3-dev libssl-dev nginx libnginx-mod-rtmp ffmpeg -y

sudo nano /etc/nginx/nginx.conf

rtmp {
	server {
		listen 1935;
		chunk_size 4096;
		notify_method get;

		application live {
			live on;

			##Set this to "record off" if you don't want to save a copy of your broadcasts

			record off;
			#record all;

			## The directory in which the recordings will be stored
			## mkdir -p /var/www/html/recordings
			## chown -R www-data:www-data /var/www/html/recordings/

			#record_path /var/www/html/recordings;
			#record_unique on;

			on_publish http://127.0.0.1/auth;
			#on_publish http://127.0.0.1/public;
		}
	}
}

sudo nano /etc/nginx/sites-enabled/default


	location /auth {
		if ($arg_pwd = 'a_secret_password') {
			return 200;
		}
		return 401;
	}

	location /public {
		return 200;
	}

    location /stat {
        rtmp_stat all;
        rtmp_stat_stylesheet stat.xsl;
        # Allow access from any visitor
        allow all;
        # Live updates for the stat page
        add_header Refresh "3; $request_uri";
    }

    location /stat.xsl {
        root /var/www/;
    }

    location /control {
        rtmp_control all;

        # Enable CORS
        add_header Access-Control-Allow-Origin * always;
    }

sudo systemctl restart nginx

cd /var/www/
sudo wget https://raw.githubusercontent.com/arut/nginx-rtmp-module/master/stat.xsl
```

## Test tools

OBS >> Ngnix >> VLC Player

- OBS Publisher configuration: 
```
In the Controls section of the Interface, click on Settings to enter the OBS configuration interface. Enter the Stream tab and enter the Information about your streaming instance.

Service: Custom
Server: rtmp://<instance_ip>/live
Stream Key: your_stream_key?pwd=a_secret_password 
```

- VLC Stream Viewer Configuration
```
Start VLC and click on Open Media. Click on the Network tab and enter the URL of your Stream.

Please enter a network URL: rtmp://<instance_ip>/live/your_stream_key
```

# Flutter

- RTMP Publisher: https://github.com/nhancv/flutter_rtmp_publisher
- Stream Viewer: https://pub.dev/packages/fijkplayer

# Note for MUX Server
```
- RTMP Url (for publisher): rtmp://live.mux.com/app/${streamkey}
- Stream Url (for client): https://stream.mux.com/${playbackId}.m3u8
- Poster Url (for preview): https://image.mux.com/${playbackId}/thumbnail.jpg
```
