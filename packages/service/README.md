# Kafka-Replay-Service

A simple REST (+websockets) service to play (publish) a sequence of messages to a Kafka topic. The messages you can play are based on their location in a folder (following the convention over configuration principle). The folder is actively watched, and new files will be automatically available.

Replay logged messages that are stored inside a, potentially mounted, folder. The folder layout is using the following convention, so you can either write your own message, one message per file, or, alternatively, use a log file from Landoop's Kafka TOPICS UI:

ROOT

- logs
  - SESSION_NAME
    - TOPIC_NAME
      - TIMESTAMP_FILENAME
  - SESSION_NAME2
    - [Kafka TOPICS UI log file].json

Messages that belong to each other are added to the same session. The topic name is the name of the topic to publish the messages to. When a TIMESTAMP (a number in msec) is set, it is used to send the message after TIMESTAMP msec.

The REST API is documented using `OpenAPI/Swagger` at [http://[HOST]:[PORT]/api-docs/](http://localhost:8200/api-docs).

The GUI is available at [http://[HOST]:[PORT]/api-docs/](http://localhost:8200).

Via websockets, your client may receive a `session_update` notification that something has changed, after which you should GET all session data again.

## Filename convention for single messages

The message file's filename convention informs us when to send it:

- It is just a name: you can publish them step-by-step or all in one go
- It is in the format 12345... (only numbers, or, in Regex /d+): it represents the offset in milliseconds since you pressed play
- Optionally, the previous format may be augmented by a textual description, such as 0001_Init_msg, in which case the `Init msg`' is used as the label of the message in the GUI.

## Extension

Finally, the message file's extension informs us how to read it. We support the following inputs:

- .xml, for XML messages
- .json or .geojson for JSON and GeoJSON messages, respectively.

## Use cases (TO BE DONE)

### Playback step-by-step

The user can select a `session_name` from the GUI, and either send messages by selecting them, and pressing play, or by sending them one after another (batch mode).
In case we are dealing with multiple topics, the user can unselect specific topics in batch mode.

### Playback scenario-based

Finally, if there is a time message signal on the test-bed, playback messages based on the external clock. The scenario duration is used to determine when to send a message.

## Build instructions

From the command prompt, install all dependencies using `npm i`. Under Windows, using Node v9, you may run into an installation error when installing `node-expat`. In that case, you can try the following:

```bash
npm i -g node-gyp
cd node_modules\node-expat
node-gyp rebuild
```

In case the `node-expat` folder is not created, run `npm i` in the `packages\service` folder, and when `node-expat` is downloaded and starts to build, break it off using CTRL-C, and follow the instructions above. For whatever reason, it seems that a `node-gyp rebuild` in the actual folder performs better than from the `service` folder.

Optionally, you may also need to install the `npm` production tools, from an admin prompt: `npm install --global --production windows-build-tools`. Or, more specifically, install [Visual Studio C++ 2012](http://go.microsoft.com/?linkid=9816758) and run `npm` with the [`--msvs_version=2012` flag](http://stackoverflow.com/a/16854333/937891).

Next, run `npm start` to compile the TypeScript code to JavaScript.