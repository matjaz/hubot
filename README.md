# Hubot

This is a version of GitHub's Campfire bot, hubot. He's pretty cool.

**You'll probably never have to hack on this repo directly.**

Instead this repo provides a library that's distributed by `npm` that you
simply require in your project. Follow the instructions below and get your own
hubot ready to deploy.

## Getting Your Own

Make sure you have [node.js][nodejs] and [npm][npmjs] (npm comes with node
v0.6.3+) installed.

Download the [latest version of hubot][hubot-latest].

Then follow the instructions in the [README][readme] in the extracted
`hubot/src/templates` directory. The `templates` directory is an example
runnable hubot.

[nodejs]: http://nodejs.org
[npmjs]: http://npmjs.org
[hubot-latest]: https://github.com/github/hubot/archive/master.zip
[readme]: https://github.com/github/hubot/blob/master/src/templates/README.md

## Adapters

Adapters are the interface to the service you want your hubot to run on. This
can be something like Campfire or IRC. There are a number of third party
adapters that the community have contributed. Check the
[hubot wiki][hubot-wiki] for the available ones and how to create your own.

Please submit issues and pull requests for third party adapters to the adapter
repo, not this repo (unless it's the Campfire or Shell adapter).

[hubot-wiki]: https://github.com/github/hubot/wiki

## Hubot Scripts

Hubot ships with a number of default scripts, but there's a growing number of
extras in the [hubot-scripts][hubot-scripts] repository. `hubot-scripts` is a
way to share scripts with the entire community.

Check out the [README][hubot-scripts-readme] for more help on installing
individual scripts.

[hubot-scripts]: https://github.com/github/hubot-scripts
[hubot-scripts-readme]: https://github.com/github/hubot-scripts#readme

## External Scripts

This functionality allows users to enable scripts from `npm` packages which
don't have to be included in the `hubot-scripts` repository.

To enable to functionality you can follow the following steps.

1. Add the packages as dependencies into your `package.json`
2. `npm install` to make sure those packages are installed

To enable third-party scripts that you've added you will need to add the package
name as a double quoted string to the `external-scripts.json` file for your
hubot.

### Creating A Script Package

Creating a script package for hubot is very simple. Start by creating a normal
`npm` package. Make sure you add a main file for the entry point (e.g.
`index.js` or `index.coffee`).

In this entry point file you're going to have to export a function that hubot
will use to load the scripts in your package. Below is a simple example for
loading each script in a `./scripts` directory in your package.

```coffeescript
Fs   = require 'fs'
Path = require 'path'

module.exports = (robot) ->
  path = Path.resolve __dirname, 'scripts'
  Fs.exists path, (exists) ->
    if exists
      robot.loadFile path, file for file in Fs.readdirSync(path)
```

After you've built your `npm` package you can publish it to [npmjs][npmjs].

## HTTP Listener

Hubot has a HTTP listener which listens on the port specified by the `PORT`
environment variable. If PORT is not specified, the default port will be 8080.

You can specify routes to listen on in your scripts by using the `router`
property on `robot`.

```coffeescript
module.exports = (robot) ->
  robot.router.get "/hubot/version", (req, res) ->
    res.end robot.version
```

There are functions for GET, POST, PUT and DELETE, which all take a route and
callback function that accepts a request and a response.

In addition, if you set `EXPRESS_STATIC`, the HTTP listener will serve static
files from this directory.

Also, set `EXPRESS_SOCKETS` to true to setup sokect.io; you can configure it in your scripts using the `io` property on `robot`.

```coffeescript
module.exports = (robot) ->
  io = robot.router.io
  io.sockets.on 'connection', (socket) ->
    socket.emit 'news', hello: 'world'
    socket.on 'event', (data) -> console.log data
```

## Events

Hubot has also an node.js [EventEmitter][event-emitter] attached. It can be used
for data exchange between scripts.

```coffeescript
# src/scripts/github-commits.coffee
module.exports = (robot) ->
  robot.router.post "/hubot/gh-commits", (req, res) ->
  	#code goes here
    robot.emit "commit", {
        user    : {}, #hubot user object
        repo    : 'https://github.com/github/hubot',
        hash  : '2e1951c089bd865839328592ff673d2f08153643'
    }
```
```coffeescript
# src/scripts/heroku.coffee
module.exports = (robot) ->
  robot.on "commit", (commit) ->
    robot.send commit.user, "Will now deploy #{commit.hash} from #{commit.repo}!"
    #deploy code goes here
```

If you'll provide an event, it's very recommended to include a hubot user object
in data. In case of other reacting scripts want to respond to chat.

[event-emitter]: http://nodejs.org/api/events.html#events_class_events_eventemitter
