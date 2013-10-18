# Chronometer

THis module is a basic chron imitator. It creates a manager, Chron_Manager,
that fires off a time every minute (though not necessarily every 60 seconds)
and added events are executed with given intervals.

More complex intervals can be expressed as  a function that recievs the time as a moment instance.

The cronomter mixin fires off at -1000... (very negative number) so other mixins can use it.

## Installing Chronometer in hive-mvc

put the chronometer file in any resources/mixins folder.
you should not have more than one chronometer in a hive-mvc project;
it is written to only install itself
if there is not a chronometer in the apiary's config already.

## Using Chronomter outside of hive-mvc.

the export of chronometer is a function for use in hive-mvc; however it does have a Chronometer constructor as a property so
you can just call;

``` javascript
    var my_sandbox = {};
    var my_chron = new (require('./chronometer.js'))Chronometer(my_sandbox);

```

The single argument to Chronometer(Chron_Manager) isn't used any way by Chronometer.

## Example mixin using Chronometer:

``` javascript

    var _ = require('underscore');
    var util = require('util');
    var path = require('path');
    var fs = require('fs');

    /* ------------ CLOSURE --------------- */
    var EVERY_SIX_HOURS = 60 * 12;

    /** ********************
     * Purpose: poll local events every 12 hours
     * @return void
     */

    /* -------------- EXPORT --------------- */

    module.exports = function (apiary, cb) {

        function poll() {
            var tmsapi_model = apiary.model('tmsapi');
            var location_model = apiary.model('locations');

            location_model.locations.forEach(function(loc){

                tmsapi_model.poll_api(loc.zip, _.identity);
            })

        }

        cb(null, {
            name: 'poll_events',
            weight: 10000,
            respond: function (done) {
                var chronometer = apiary.get_config('chronometer');
                chronometer.add_time_listener('poll data', poll, EVERY_SIX_HOURS);
                poll();
                done();
            }
        }); // end callback
    }


```

note that the action is called with two arguments:

time {moment}
manager {Chron_Manager}

so an action can ass other listeners, cancel listeners, uncancel listeners, etc.

Each listener has an active field. If its active is set to false a listener will never trigger.
If that's not good enough the ChronEvent's _respond (NOT respond) listener can be removed from the manager.

## More custom responses

If for some reason you need a more custom handler for time events, you can manually listen for the time (time) event
that the manager emits.

## The last of the timelords

The manager's clock can be started and stopped; you can call `manager.stop_clock()` or `manager.start_clock`.
This is useful for unit tests or other scenarios where you want to control the execution of long tasks.

You could, for instnace, stop the clock and fire off times at whatever interval you please -- every five minutes,
every 15 minutes, every hour....

## Why this doesn't use/emulate CHRON

I find the notation of Chron overly arcane. I figured its best to just handle the "Extreme" scenarios:

1. You want to do something on a regular interval (pretty common)
2. You want to do something on a strange interval (for which you can write a function).

Giving you a function that interprets an absolute time really frees you up. You can execute on a given set of
absolute dates, or even pull a set of times from a data feed and execute on those times.

If the Chron system of notation works for you, using node-chron is pretty easy, and chronometer is not the module for you.