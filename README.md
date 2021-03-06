# pdjr-skplugin-alarm-manager

Issue notification and other outputs in response to Signal K alarm
conditions.

__pdjr-skplugin-alarm-manager__ implements a centralised mechanism for
issuing alarm notifications contingent upon the alarm configuration
properties embedded in the meta values associated with monitored keys.

The design of the plugin acknowledges the Signal K specification
discussions on 
[Metadata](https://github.com/SignalK/specification/blob/master/gitbook-docs/data_model_metadata.md)
and
[Alarm, Alert, and Notification Handling](https://github.com/SignalK/specification/blob/master/gitbook-docs/notifications.md).

__pdjr-skplugin-alarm-manager__ generates three types of outputs in
response to an alarm condition.

Firstly, it reponds to the requirements of the Signal K specification
by issuing notifications: thus, an alarm triggered by a value on *key*
will raise a notification on 'notifications.*key*'.

Secondly, the plugin maintains a digest of active alarm notifications
at 'notifications.plugins.alarm.digest'.
The digest is simply an array of all currently active notifications and
provides a convenient data set for use by software annunciators or
other alarm consumers.

Thirdly, the plugin may operate one or more user-defined switch outputs
in response to the system's alarm state.
This allows, for example, the host system to operate arbitrary types of
physical annunciator.

[pdjr-skplugin-alarm-manager-widget](https://github.com/preeve9534/signalk-alarm-widget)
is a simple web component that can be included in any webapp to provide
a front-end annunciator for __pdjr-skplugin-alarm-manager__.

## Operating principle

The keys which __pdjr-skplugin-alarm-manager__ monitors are
automatically derived from the Signal K tree by examination of those
paths which are configured through their meta properties to support
alarm function.

The __ignorepaths__ configuration property allows sections of the
Signal K tree to be excluded wholesale from alarm processing.

Values appearing on an alarm *key* are checked against the associated
meta zones configuration and if their value falls within an alarm zone
then a notification will be issues on the path 'notifications.*key*'
using the rules defined in the meta.

The __outputs__ configuration property can be used to specify one or
more switch output paths which will be updated using PUT requests in
response to the presence or absence of active alarm notifications.
In this way the plugin can operate one or more output switches or
relays in response to the presence or absence of active alarm
notifications with particular state property values.
This feature allows external annunciators to be operated in direct
response to Signal K's internal alarm state.

The correct operation of __pdjr-skplugin-alarm-manager__ depends upon
the presence of meta information at the time a trigger key is
processed.
There is a short period following a server restart during which it is
possible that dynamically generated meta data is not yet in place.
To ensure that alarm conditions are not missed during this critical
phase it is possible to defer the start of alarm processing until a
true-ish condition appears on a trigger key defined by the
configuration __starton__ property.
True-ish means either numeric 1 or the presence of a notification.
If __starton__ is not defined then __pdjr-skplugin-alarm-manager__ will
begin execution immediately on server boot.

## Example configuration

The reference implementation on my boat looks like this.
```
{
  "enabled": true,
  "enableLogging": false,
  "enableDebug": false,
  "configuration": {
    "ignorepaths": [ "design.", "electrical.", "environment.", "network.", "notifications.", "sensors." ],
    "starton": "notifications.plugins.meta.status",
    "outputs": [
      { "switchpath": "electrical.switches.bank.usb0.1", "triggerstates": [ "warn", "alert" ] },
      { "switchpath": "electrical.switches.bank.usb0.2", "triggerstates": [ "alarm", "emergency" ] }
    ]
  }
}
```

__ignorepaths__ eliminates all the key trees I don't want to monitor.

__starton__ points to the location of the 'completed' notification
issued by the plugin I use to inject dynamic meta data.
See [__signalk-meta__](https://github.com/preeve9534/signalk-meta#readme).

I have a multi-channel alarm controller installed at the ship's helm
which supports high and low priority inputs (low priority inputs light
up an indicator whilst high priority inputs also sound a beeper).
The __outputs__ configuration controls a two-channel external relay
(usb0) that announces the presence of 'warn'/'alert' notifications
through a 'low priority' relay channel (usb0.1) and 'alarm'/'emergency'
notifications through a 'high priority' relay channel (usb0.2).
See [__signalk-devantech__](https://github.com/preeve9534/signalk-devantech#readme)
for details of the hardware interface.
