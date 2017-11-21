# node-transmission

transmission-daemon wrapper script written in node.js

## Install

```sh
npm install transmission
```

## How to Use

```js
Transmission = require ('transmission')
transmission = new Transmission({
  host: 'localhost'  # default 'localhost'
  port: 9091         # default 9091
  username: 'username'   # default blank
  password: 'password'   # default blank
  ssl: true   # default false use https
  url: '/my/other/url'   # default '/transmission/rpc'
  });
  ```  

## Definition

### Status

RPC returned torrent status with integer `0-7`.

Using `transmission.status` for inspect status.

```
transmission.status =
  STOPPED       : 0  # Torrent is stopped
  CHECK_WAIT    : 1  # Queued to check files
  CHECK         : 2  # Checking files
  DOWNLOAD_WAIT : 3  # Queued to download
  DOWNLOAD      : 4  # Downloading
  SEED_WAIT     : 5  # Queued to seed
  SEED          : 6  # Seeding
  ISOLATED      : 7  # Torrent can't find peers
```

## Methods

### transmission.set([ids], callback)

Set torrent's properties.

```js
transmission.set(id, options, function(err, arg){});
```
You must provide one or more ids. According to the rpc-spec, transmission will not respond a success argument. Only error.

### transmission.addFile(filePath, callback)

Add torrents to transmission-daemon.

```js
transmission.addFile('path', function(err, arg){});
```

OR

The `options` object would be the arguments passed to transmission.
If you want to set the download directory of the torrent you would pass in `"download-dir":"/my/path"`

```js
transmission.addFile('path', options, function(err, arg){});
```

### transmission.addUrl(url, callback)

Add torrents to transmission-daemon.

```js
transmission.addUrl('url', function(err, arg){});
```
OR

The `options` object would be the arguments passed to transmission.
If you want to set the download directory of the torrent you would pass in `"download-dir":"/my/path"`

```js
transmission.addUrl('url', options, function(err, arg){});
```

### transmission.remove(ids, del, callback)

Remove torrents.

Remove also local data when `del` is `true`.

```js
transmission.remove(ids, function(err, arg){});
```

### transmission.active(callback)

List of active torrents. Callback is not needed and will fire the `active` event.

```js
transmission.active(function(err, arg){});
```

### transmission.get([ids], callback)

Get torrents info that optional `ids`.

If omit `ids`, get all torrents.

```js
transmission.get(ids, function(err, arg){
  if err
    console.error err
  else{
    for torrent in arg.torrents
      console.log arg.torrents
   }
});

# Get all torrents and remove it if status is stopped.
transmission.get(function(err, arg){
  if err
    console.error err
  else{
    for torrent in arg.torrents
      if torrent.status is transmission.status.STOPPED
        transmission.remove ([torrent.id], function (err){
          console.error err if err
      	});
  }
});
```

### transmission.waitForState(id, targetState, callback)

Polls the server and waits for the target state.
targetState = ['STOPPED', 'CHECK_WAIT', 'CHECK', 'DOWNLOAD_WAIT', 'DOWNLOAD', 'SEED_WAIT', 'SEED', 'ISOLATED'];

```js
transmission.waitForState(id, targetState, function(err, arg){});
```

### transmission.stop(ids, callback)

Stop working torrents.

```js
transmission.stop(ids, function(err, arg){});
```

### transmission.start(ids, callback)

Start working torrents.

```js
transmission.start(ids, function(err, arg){});
```

### transmission.startNow(ids, callback)

Bypass the download queue, start working torrents immediately.

```js
transmission.startNow(ids, function(err, arg){});
```

### transmission.verify(ids, callback)

Verify torrent data.

```js
transmission.verify(ids,function(err, arg){});
```

### transmission.reannounce(ids, callback)

Reannounce to the tracker, ask for more peers.

```js
transmission.reannounce(ids, function(err, arg){});
```

### transmission.session(callback)

Get client session infomation.

```js
transmission.session (function(err, arg){});
```

### transmission.session({}, callback)

Set session infomation.

```js
transmission.session({'download-dir':'/my/path'}, function(err, arg){});
```

### transmission.sessionStats(callback)

Get client session stats.

```js
transmission.sessionStats(function(err, arg){});
```

### transmission.freeSpace(path, callback)

Get free space available on the server for the specified directory.

```js
transmission.freeSpace(path, function(err, arg){});
```

### All together.

```js
'use strict';

var Transmission = require('transmission');
var transmission = new Transmission({
	port: 9091,			// DEFAULT : 9091
	host: 192.168.1.34,			// DEAFULT : 127.0.0.1
	username: 'username',	// DEFAULT : BLANK
	password: 'password'	// DEFAULT : BLANK
});

// Get details of all torrents currently queued in transmission app
function getTransmissionStats(){
	transmission.sessionStats(function(err, result){
		if(err){
			console.log(err);
		} else {
			console.log(result);
		}
	});
}

// Add a torrent by passing a URL to .torrent file or a magnet link
function addTorrent(url){
	transmission.addUrl(url, {
	    "download-dir" : "~/transmission/torrents"
	}, function(err, result) {
	    if (err) {
	        return console.log(err);
	    }
	    var id = result.id;
	    console.log('Just added a new torrent.');
	    console.log('Torrent ID: ' + id);
	});
}

// Get various stats about a torrent in the queue
function getTorrentDetails(id) {
    transmission.get(id, function(err, result) {
        if (err) {
            throw err;
        }
        if(result.torrents.length > 0){
        	// console.log(result.torrents[0]);			// Gets all details
        	console.log("Name = "+ result.torrents[0].name);
        	console.log("Download Rate = "+ result.torrents[0].rateDownload/1000);
        	console.log("Upload Rate = "+ result.torrents[0].rateUpload/1000);
        	console.log("Completed = "+ result.torrents[0].percentDone*100);
        	console.log("ETA = "+ result.torrents[0].eta/3600);
        	console.log("Status = "+ getStatusType(result.torrents[0].status));
        }
    });
}

function deleteTorrent(id){
	transmission.remove(id, true, function(err, result){
		if (err){
			console.log(err);
		} else {
			transmission.get(id, function(err, result) {
		       //err no torrent found...
		    });
		}
	});
}

// To start a paused / stopped torrent which is still in queue
function startTorrent(id){
	transmission.start(id, function(err, result){});
}

// To get list of all torrents currently in queue (downloading + paused)
// NOTE : This may return null if all torrents are in paused state 
function getAllActiveTorrents(){
	transmission.active(function(err, result){
	if (err){
		console.log(err);
	}
	else {
		for (var i=0; i< result.torrents.length; i++){
			console.log(result.torrents[i].id);
			console.log(result.torrents[i].name);
		}
	}
	});
}

// Pause / Stop a torrent
function stopTorrent(id){
	transmission.stop(id, function(err, result){});
}

// Pause / Stop all torrent
function stopAllActiveTorrents(){
	transmission.active(function(err, result){
	if (err){
		console.log(err);
	}
	else {
		for (var i=0; i< result.torrents.length; i++){
			stopTorrents(result.torrents[i].id);
		}
	}
	});
}

// Remove a torrent from download queue
// NOTE : This does not trash torrent data i.e. does not remove it from disk
function removeTorrent(id) {
    transmission.remove(id, function(err) {
        if (err) {
            throw err;
        }
        console.log('torrent was removed');
    });
}

// Get torrent state
function getStatusType(type){
	return transmission.statusArray[type]
	if(type === 0){
		return 'STOPPED';
	} else if(type === 1){
		return 'CHECK_WAIT';
	} else if(type === 2){
		return 'CHECK';
	} else if(type === 3){
		return 'DOWNLOAD_WAIT';
	} else if(type === 4){
		return 'DOWNLOAD';
	} else if(type === 5){
		return 'SEED_WAIT';
	} else if(type === 6){
		return 'SEED';
	} else if(type === 7){
		return 'ISOLATED';
	}
}

```


