# MqttDlnaPLay

Small dlna player that can be controlled through mqtt in
order to play media to a media renderer (ex: smart TV).

It is currently tied to file naming as generated by the
[JavaPVR](https://github.com/mhdawson/JavaPVR) project and the
feature to automatically mark/hide media that you have
already watched.  This allows it to automatically
figure out the next episode in a series that was
recorded but has not yet been watched and play that
for you.

I current use it with [AlexaMqttBridge]
(https://github.com/mhdawson/AlexaMqttBridge) so
that we can simply say:

```text
Alexa ask Michael whats new on TV
Alexa ask Michael to play XXXX on TV
Alexa ask Michael to pause TV
```

were XXXX is the name of a recorded TV series and
it will play the next available episode that
we have not yet watched.

It then also support the regular stop/start/seek
type commands so that you can control the media
being played once it is started.

You can also ask what new shows are available with

```text
Alexa ask Michael what is new on TV
```

The list of topic/messaage combinations that can be used include:

* play a show
  * topic - topic from configuration file + /play
  * message - name of the show
* control a show that is playing
  * topic - topic from configuration file + /control
  * message - one of stop, pause, play, seek.  If seek is specified then
    the message should be 'seek:' followed by the minutes into the show
    you want to seek to
* find out what new shows are available
  * topic - topic from configuration file + /control
  * message - 'whatsnew'

For commands other than 'whatsnew', ',X' can be appended to specify a specfic
media renderer where X corresponds to the index for the render in the array
within the confirmation file.  For example:

```
stop, 1
```
this allows multiple renderers(and therefore TVs) to be supported.  I use this
with both the TV in the living room and one upstairs.

The response to a request or any errors will be published back to the
topic from the configuration file + '/response'.  For example, if you ask
whats new the response published would be (of course depends on the new shows):

```text
New episodes include: chicago fire,22 minutes,the mindy project,greys anatomy,
rick mercer report,the catch,survivor
```

In the case of errors, the response is the text explanation of the error that occured
for example:

```text
No current episodes for:no show
```

The micro-app provides a simple GUI which simply shows the requested commands
and responses with the most recent being displayed first:

![MqttDlnaPlay - GUI](https://raw.githubusercontent.com/mhdawson/MqttDlnaPlay/master/pictures/mqttdlnaplay.png)

# Installation

The easiest way to install is to run:

```
npm install MqttDlnaPlay
```

or

```
npm install https://github.com/mhdawson/MqttDlnaPlay.git
```

and then configure the default config.json file in the lib directory
as described in the configuration section below.

# Configuration

Configuration is done thorugh the config.json file in the lib directory.
Configuration options include:

* title(optional) - title for the indicator window.
* serverPort - port on which the server listens for connections.
* mqttServerUrl - object with serverUrl.
* topic - the topic on which the player will listen for requests.
* dnalServerName - the dlna server to serve media from.
* dlnaSearchRoot - the root in the dlan server to serve media from.
* mediaPlayer - array of objects representing the list of media players the
  media will be played on.  Each object has a "name" and optional "UDN" field.
  DLNA requests are made to search for media renderers and the responses
  are matched against the name and optionally the UDN.  The order of the
  entries is signficant in that the first one is the default. When commands
  are requested through mqtt, if the renderer is not specified it will default
  to the first entry in the list.  To submit requests to other renderers you   
  must include the index into the array. To do this ',X' is appended to the
  command where X is the index into the array.
* newDays - the number of days a show is considered 'new' after this number
  of days a show will not be included in the response to the 'whats new'
  command.
* ignore - list of show to exclude when responding to the 'whats new' command.

As a micro-app the bridge also supports other options like authentication and
tls for the GUI connection.  See the documentation for the
[micro-app-framework](https://github.com/mhdawson/micro-app-framework)
for additional details.

As an example a sample configuration file would be:

```json
{
  "serverPort": 5010,
  "MaxRecentActivity": 100,
  "topic": "house/dlnaplay",
  "mqttServerUrl": "tcp://10.1.1.186:1883",
  "dlnaServerName": "New - Michaels Media Server",
  "dlnaSearchRoot": "New(auto)",
  "mediaPlayer": [
                   { "name": "TV-46C610",
                     "UDN": "uuid:0c23e459-0793-6f51-45f9-5a4255bb8601" },
                   { "name": "ATV@12" }
                 ],
  "newDays": 14,
  "ignore": [ "marvels_agents_of_shield", "blindspot", "scorpion",
              "madam_secretary", "the_blacklist", "the_simpsons" ]
}
```
