# docker-logentries

Forward all your logs to [LogEntries](https://logentries.com), like a breeze.

![logentries dashboard](https://raw.githubusercontent.com/nearform/docker-logentries/master/dashboard.png)

See the Logentries community pack at [http://revelops.com/community/packs/docker/](http://revelops.com/community/packs/docker/).

## Usage as a Container

To use with Docker listening on TCP port:

```sh
docker run --rm  polyverse/docker-logentries -t 2ab680b7-1492-4233-ba61-48c76fd0e699 -H 192.168.99.100 -P 2376 --protocol tcp -j -a host=`uname -n`
```

The simplest way to forward all your container's log to LogEntries is to
run this repository as a container, with:

```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock logentries/docker-logentries -t <TOKEN> -j -a host=`uname -n`
```

You can also use two different tokens for logging and stats:
```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock logentries/docker-logentries -l <LOGSTOKEN> -k <STATSTOKEN> -j -a host=`uname -n`
```

You can also pass the `--no-stats` flag if you do not want stats to be
published to logentries every second. You __need this flag for Docker
version < 1.5__.

You can also pass the `--no-logs` flag if you do not want logs to be
published to logentries.

You can also filter filter the containers for whitch the logs/stats are
forwarded with:

* `--matchByName REGEXP`: forward logs/stats only for the containers whose name matches the given REGEXP.
* `--matchByImage REGEXP`: forward logs/stats only for the containers whose image matches the given REGEXP.
* `--skipByName REGEXP`: do not forward logs/stats for the containers whose name matches the given REGEXP.
* `--skipByImage REGEXP`: do not forward logs/stats for the containers whose image matches the given REGEXP.

### Running container in a restricted environment.
Some environments(such as Google Compute Engine) does not allow to access the docker socket without special privileges. You will get EACCES(`Error: read EACCES`) error if you try to run the container.
To run the container in such environments add --privileged to the `docker run` command.

Example:
```sh
docker run --privileged -v /var/run/docker.sock:/var/run/docker.sock logentries/docker-logentries -t <TOKEN> -j -a host=`uname -n`
```

## Usage as a CLI

1. `npm install docker-logentries -g`
2. `docker-logentries -t TOKEN -a host=\`uname -n\``
3. ..there is no step 3

You can also pass the `-j` switch if you log in JSON format, like
[bunyan](http://npm.im/bunyan).
You can also pass the `--no-stats` flag if you do not want stats to be
published to logentries every second.
The `-a/--add` flag allows to add fixed values to the data being
published. This follows the format 'name=value'.

You can also filter filter the containers for whitch the logs/stats are
forwarded with:

* `--matchByName REGEXP`: forward logs/stats only for the containers whose name matches the given REGEXP.
* `--matchByImage REGEXP`: forward logs/stats only for the containers whose image matches the given REGEXP.
* `--skipByName REGEXP`: do not forward logs/stats for the containers whose name matches the given REGEXP.
* `--skipByImage REGEXP`: do not forward logs/stats for the containers whose image matches the given REGEXP.

## Embedded usage

Install it with: `npm install docker-logentries --save`

Then, in your JS file:

```
var logentries = require('docker-logentries')({
  json: false, // or true to parse lines as JSON
  secure: false, // or true to connect securely
  token: process.env.TOKEN, // logentries TOKEN
  stats: true, // disable stats if false
  add: null, // an object whose properties will be added

  // the following options limit the containers being matched
  // so we can avoid catching logs for unwanted containers
  matchByName: /hello/, // optional
  matchByImage: /matteocollina/, //optional
  skipByName: /.*pasteur.*/, //optional
  skipByImage: /.*dockerfile.*/ //optional
})

// logentries is the source stream with all the
// log lines

setTimeout(function() {
  logentries.destroy()
}, 5000)
```

## Building a docker repo from this repository

First clone this repository, then:

```bash
docker build -t logentries .
docker run -v /var/run/docker.sock:/var/run/docker.sock logentries -t <TOKEN> -j -a host=`uname -n`
```

## How it works

This module wraps four [Docker
APIs](https://docs.docker.com/reference/api/docker_remote_api_v1.17/):

* `POST /containers/{id}/attach`, to fetch the logs
* `GET /containers/{id}/stats`, to fetch the stats of the container
* `GET /containers/json`, to detect the containers that are running when
  this module starts
* `GET /events`, to detect new containers that will start after the
  module has started

This module wraps
[docker-loghose](https://github.com/mcollina/docker-loghose) and
[docker-stats](https://github.com/pelger/docker-stats) to fetch the logs
and the stats as a never ending stream of data.

All the originating requests are wrapped in a
[never-ending-stream](https://github.com/mcollina/never-ending-stream).

## License

MIT
