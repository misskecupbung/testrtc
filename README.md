## README
This repository is modified from https://github.com/KaptenJansson/testrtc with some custom for personal local use. Why?
* My own turn is dedicated locally -> on-premise (not on Google Cloud Platform)
* This guide is for testing whether the client can connect to my local application with a separate turn server,
* This doesn't use apprtc at https://github.com/webrtc/apprtc
* This guide doesn't use `API_KEY` parameter

## Requirement
* Install turn-server , remember :
    * IP/domain, eg: turn.example.com:5349
    * username, eg: USERNAME
    * password, eg: PASSWORD
* This guide is tested on [Ubuntu 18.04](https://ubuntu.com/).

## Deployment

### Setup app :
```
cd /opt
git clone https://github.com/misskecupbung/testrtc
cd testrtc
npm update
npm install --unsafe-perm=true --allow-root
npm install -g npm bower grunt
bower update --allow-root
grunt
grunt build
```

## Install google-cloud-sdk :
```
apt install python-pip -y
pip install webapp2 webob bootstrapping
cd /opt
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-292.0.0-linux-x86_64.tar.gz
tar zxvf google-cloud-sdk-292.0.0-linux-x86_64.tar.gz google-cloud-sdk
./google-cloud-sdk/install.sh
vim /etc/profile
...
#this next line enables pythonpath cloudsdk and appengine homedir
export PYTHONPATH=${PYTHONPATH}:$(gcloud info --format="value(installation.sdk_root)")/platform/google_appengine
export CLOUDSDK_ROOT_DIR="/opt/google-cloud-sdk"
export APPENGINE_HOME="${CLOUDSDK_ROOT_DIR}/platform/appengine-java-sdk"
export GAE_SDK_ROOT="${CLOUDSDK_ROOT_DIR}/platform/google_appengine"

# The next line enables Java libraries for Google Cloud SDK
export CLASSPATH="${APPENGINE_HOME}/lib":${CLASSPATH}

# The next line enables Python libraries for Google Cloud SDK
#export PYTHONPATH=${GAE_SDK_ROOT}:${PYTHONPATH}
...
source /etc/profile
gcloud components update
gcloud components install app-engine-python app-engine-python-extras appctl
```

## Custom file Gruntfile.js :
```
vim /opt/testrtc/Gruntfile.js :
...
    //   'API_KEY': process.env.API_KEY,
         'TURN_URL': 'turn.example.com:5349'
    //   'TURN_URL': 'https://networktraversal.googleapis.com/v1alpha/iceconfig?key='
...
```
## Custom file .eslintrc :
```
vim /opt/testrtc/.eslintrc
...
    "API_KEY": false,

```
* Custom file call.js :
```
vim /opt/testrtc/src/js/call.js
...
########### Before ##############

// Get a TURN config, either from settings or from network traversal server.
Call.asyncCreateTurnConfig = function(onSuccess, onError) {
  var settings = currentTest.settings;
  if (typeof(settings.turnURI) === 'string' && settings.turnURI !== '') {
    var iceServer = {
      'username': settings.turnUsername || '',
      'credential': settings.turnCredential || '',
      'urls': settings.turnURI.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('turn-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  } else {
    Call.fetchTurnConfig_(function(response) {
      var config = {'iceServers': response.iceServers};
      report.traceEventInstant('turn-config', config);
      onSuccess(config);
    }, onError);
  }
};

// Get a STUN config, either from settings or from network traversal server.
Call.asyncCreateStunConfig = function(onSuccess, onError) {
  var settings = currentTest.settings;
  if (typeof(settings.stunURI) === 'string' && settings.stunURI !== '') {
    var iceServer = {
      'urls': settings.stunURI.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('stun-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  } else {
    Call.fetchTurnConfig_(function(response) {
      var config = {'iceServers': response.iceServers.urls};
      report.traceEventInstant('stun-config', config);
      onSuccess(config);
    }, onError);
  }
};

############## After #############

// Get a TURN config, either from settings or from network traversal server.
Call.asyncCreateTurnConfig = function(onSuccess, onError) {
  var settings = currentTest.settings;
   console.log("test-turn")
   console.log(settings)
  if (typeof(settings.turnURI) === 'string' && settings.turnURI !== '') {
    var iceServer = {
      'username': settings.turnUsername || '',
      'credential': settings.turnCredential || '',
      'urls': settings.turnURI.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('turn-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  } else {
    var iceServer = {
      'username': 'USERNAME',
      'credential': 'PASSWORD',
      'urls': 'turn:turn.example.com:5349'.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('turn-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  }
};
// Get a STUN config, either from settings or from network traversal server.
Call.asyncCreateStunConfig = function(onSuccess, onError) {
  var settings = currentTest.settings;
  if (typeof(settings.stunURI) === 'string' && settings.stunURI !== '') {
    var iceServer = {
      'urls': settings.stunURI.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('stun-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  } else {
    var iceServer = {
      'urls': 'stun:turn.example.com:5349'.split(',')
    };
    var config = {'iceServers': [iceServer]};
    report.traceEventInstant('stun-config', config);
    setTimeout(onSuccess.bind(null, config), 0);
  }
};


###### Bottom, custome like this #######

xhr.open('POST', TURN_URL + API_KEY, false);
...

grunt
grunt build
## Testing with custom port
```
python /opt/google-cloud-sdk/bin/dev_appserver.py /opt/testrtc/out/app.yaml --enable_host_checking=False
```
## Setup service :
```
# This setup is for can run even reboot

vim /etc/systemd/system/testrtc.service 
...
[Unit]
Description=Test RTC

[Service]
User=root
ExecStart=/usr/bin/python /opt/google-cloud-sdk/bin/dev_appserver.py /opt/testrtc/out/app.yaml --enable_host_checking=False --port 8081 --admin_port 8082

[Install]
WantedBy=default.target
...

systemctl enable testrtc.service
systemctl start testrtc.service
netstat -tulpn
```