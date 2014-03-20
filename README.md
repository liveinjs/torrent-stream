# torrent-stream

The streaming torrent engine that [peerflix](https://github.com/mafintosh/peerflix) uses

	npm install torrent-stream

## How can I help?

1. Open issues on things that are broken
2. Fix open issues by sending PRs
3. Add documentation

## Usage

torrent-stream is a node module that allows you to access files inside a torrent as node streams.

``` js
var torrentStream = require('torrent-stream');
var fs = require('fs');

var engine = torrentStream('magnet:my-magnet-link');

engine.on('ready', function() {
	engine.files.forEach(function(file) {
		console.log('filename:', file.name);
		var stream = file.createReadStream();
		// stream is readable stream to containing the file content
	});
});
```

You can pass `start` and `end` options to stream to slice the file

``` js
// get a stream containing bytes 10-100 inclusive.
var stream = file.createReadStream({
	start: 10,
	end: 100
});
```

Per default no files are downloaded unless you create a stream to them.
If you want to fetch a file anyway use the `file.select` and `file.deselect` method.

When you start torrent-stream it will connect to the torrent dht
and fetch pieces according to the streams you create.

## Full API

#### `engine = torrentStream(magnet_link_or_buffer, opts)`

Create a new engine instance. Options can contain the following

``` js
{
	connections: 100,     // Max amount of peers to be connected to.
	path: '/tmp/my-file', // Where to save the buffer data.
	verify: true,         // Verify previously stored data before starting
	dht: true             // Whether or to use the dht to find peers.
	                      // Defaults to true
}
```

#### `engine.on('ready', fn)`

Emitted when the engine is ready to be used.
The files array will be empty until this event is emitted

#### `engine.on('download', [piece-index])`

Emitted everytime a piece has been downloaded and verified.

#### `engine.on('upload', [piece-index, offset, length])`

Emitted everytime a piece is uploaded.

#### `engine.files[...]`

An array of all files in the torrent. See the file section for more info on what methods the file has

#### `engine.destroy()`

Destroy the engine. Destroys all connections to peers

#### `engine.connect('127.0.0.0:6881')`

Connect to a peer manually

#### `engine.disconnect('127.0.0.1:6881')`

Disconnect from a peer manually

#### `engine.remove(cb)`

Completely remove all saved data for this torrent

#### `engine.listen([port], cb)`

Listen for incoming peers on the specified port. Port defaults to `6881`

#### `engine.swarm`

The attached [peer-wire-swarm](https://github.com/mafintosh/peer-wire-swarm) instance

#### `file = engine.files[...]`

A file in the torrent. They contains the following data

``` js
{
	name: 'my-filename.txt',
	path: 'my-folder/my-filename.txt',
	length: 424242
}
```

#### `file.select()`

Selects the file to be downloaded, but at a lower priority than streams.
Useful if you know you need the file at a later stage.

#### `file.deselect()`

Deselects the file which means it won't be downloaded unless someone creates a stream to it

#### `stream = file.createReadStream(opts)`

Create a readable stream to the file. Pieces needed by the stream will be prioritized highly.
Options can contain the following

``` js
{
	start: startByte,
	end: endByte
}
```

Both `start` and `end` are inclusive

## License

MIT
