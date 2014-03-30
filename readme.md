
# HOTDOGSEAGULL #
This is a server built in node.js that lets you browse your media using a browser and push it up to a chromecast. Should be suitable for running on a home server. 

This is intended as a starting point for UNIX inclined people to set up local media servers for the chromecast. The UI intentionally looks bad because I want to leave the UI to others while I focus on the technical aspects. If you make any improvements to this code, feel free to send a pull request. See below for more details on future development 

Tested on OS X but should work on anything that can run node and ffmpeg.

Verified to work even on a Raspberry Pi when files are fully compatible (i.e. no transcoding needs to be done).

## Features ##
 * Uses the default media player app on the Chromecast - no need to get a developer ID or publish an app.
 * Detects which of your local files are fully compatible with the chromecast using ffmpeg
 * Transcoding that only re-encodes the streams that need it. (I.e. audio/video that is already compatible will not be transcoded.).
 * External and Internal subtitles (ass,srt) and audio/video track selection when using transcoding.
 * Extremely basic UI. Uses templates so adding skins and customizing should be easier.
 * API so you can use transcoding and compatibility checking in your own app! (See below)
 * Lets you try files anyways just in case detection got it wrong. (Let me know when detection is wrong).
 * Lots of debugging output
 * Ridiculous name

## Installation ##
Depends on ffmpeg and ffprobe (provided by ffmpeg). You can install this using Homebrew or whatever package manager you have on your system. If you have issues with transcoding, check the ffmpeg configuration section for the flags you need.

Depends on node packages: express, dot, node-ffprobe, fluent-ffmpeg

```
git clone https://github.com/craiig/HOTDOGSEAGULL
cd HOTDOGSEAGULL
brew install ffmpeg --with-libass --enable-libx264
npm install
cp config.sample.json config.json
edit 'media_folder' variable in config.json to point to your media directory
node server.js
Visit http://<local_ip>:3000 in a web browser.
```
NOTE: You need to use an ip that the chromecast can access. I.e. use 192.168.1.X and don't use localhost.

### FFmpeg configuration ###
I've tested this with ffmpeg 1.2.4. If you are having issues with transcoding, particularly subs, you should ensure your ffmpeg version has been compiled with the following flags: *--enable-libass --enable-libx264*. Run *ffmpeg -version* to see what options it has been built with. 

**OS X Users**: Homebrew does not include the subtitle library by default. Install ffmpeg with the following command: *brew install ffmpeg --with-libass --enable-libx264*

Leave me a note if you have to recompile ffmpeg for your system and I'll make a note here for others.

## Known Issues ##
As of Feb 24 2014:
 * The Chromecast default media player doesn't seem to update the sender on the progress through the video. Manual status changes, such as pausing, muting, etc will cause the progress to get updated. This is also an issue in Google's CastHelloVideo, so I'm not sure if it's just not currently possible or I'm not adding a listener in the right places.
 * Skipping while transcoding is not supported. I'm looking into ways to make this work.

## Warranty & Support ##
This server might be very insecure and may leak all your files. Security audits/fixes are more than welcome. For the love of god put this behind a firewall.

Make posts in the github issue tracker if you have problems.

## Upcoming features & Contributing ##
To contribute, just send a pull request on github and I'll look at it. Bugs, new features, better UI, it's all welcome but I don't guarantee I'll accept everything.

Here are some features that would be good to add:
 * Better UI layout
 * Read thumbnail from the video using ffmpeg
 * Skin & config system so you can switch between skins, etc.
 * UI & support for mobile browsers
 * Packaging this up for less technical users on other systems.

Here's what I'm working on:
 * Automatic extraction of in-stream subs
 * Improving the player interaction with the chromecast
 * Play whole directory (gimme dat binge)
 * DLNA support (if possible)
 * Offline transcoding support

## API ##
This module supports a basic API so that you can add compatibility checking and transcoding to your own node programs. Please feel free to give me some feedback on what you'd like to see from an API. Ideally many people can use this to get the server-side media features for the chromecast. I'm also looking into ways of supporting the javascript needed for the chromecast as well.

```
npm install https://github.com/craiig/HOTDOGSEAGULL/archive/master.tar.gz
```

```javascript
chromecast = require('HOTDOGSEAGULL')
filename = ...

//retrieve chromecast compatibility information
chromecast.get_file_data(filename,
	function(compatible, compat_data){
		if(compatible){
			console.log("File is fully compatible");
		} else {
			console.log("File needs some transcoding");
		}
		console.log(compat_data);	
	});

//set up a route that transcodes the files
//note: make sure to set up a static route so you can serve files that are fully compatible
app.get('/transcode', function(req, res) {
	filename = ...
	options = {}; //list of options for transcoding, see the source for more details
	ffmpeg_flags = ""; //pass flags directly to ffmpeg
	chromecast.transcode_stream(filename, res, options, "", 
	  function(err, ffmpeg_error_code, ffmpeg_output){
		if(err){
			console.log("transcode error:");
			console.log(ffmpeg_output);
		} else {
			console.log("transcoding finished ffmpeg_output: ");
			console.log(ffmpeg_output);
		}
	});
 
});
```

## License ##
I used some code from Google's CastHelloVideo-chrome which are under the Apache 2.0 License. https://github.com/googlecast/CastHelloVideo-chrome

You should consider my code to be under the Apache 2.0 license but with an ADDED REQUIREMENT THAT YOU MUST EXPLICITLY MENTION THIS RIDICULOUS PROJECT NAME (HOTDOGSEAGULL) IN YOUR ACCOMPANYING DOCUMENTATION.