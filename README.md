# Node-inotify - monitoring file system events in Gnu/Linux with [NodeJS][nodejs_home]
The inotify API provides a mechanism for monitoring file system events.
Inotify can be used to monitor individual files, or to monitor directories.
When a directory is monitored, inotify will return events for the directory
itself, and for files inside the directory. [(ref: GNU/Linux Manual)][inotify.7]

## Installation
You must have [NodeJS][nodejs_dev] already installed to be able to build node-inotify.

    $ git clone git://github.com/c4milo/node-inotify.git
    $ cd node-inotify
    $ node-waf configure build

### Easy way
    $ npm install inotify

## API
  * `var inotify = new Inotify()`: Create a new instance of Inotify. By default it's in persistent mode.
  You can specify `false` in `var inotify = new Inotify(false)` to use the non persistent mode.

  * `var wd = inotify.addWatch(arg)`:  Add a watch for files or directories. This will then return a watch descriptor. The argument is an object as follows
        var arg = { path: 'path to be monitorized',
                    watch_for: an optional OR'ed set of events to watch for.
                               If they're not specified, it will use
                               Inotify.IN_ALL_EVENTS by default,
                    callback: Callback function that will receive each event.
        }
You can call this function as many times as you want to monitorize different paths.
**Inotify monitoring of directories is not recursive**: to monitor subdirectories
under a directory, additional watches must be created.

  * `inotify.removeWatch(watch_descriptor)`: Remove a watch associated with the watch_descriptor param and returns `true` if the action was succesful or `false` in the opposite case. Removing a watch cause an `Inotify.IN_IGNORED` event to be generated for this watch descriptor.

  * `inotify.close()`: Remove all the watches and close the inotify's file descriptor. Returns `true` if the action was succesful or false in the opposite case.

### Event object structure
    var event = {   watch: Watch descriptor,
                    mask: Mask of events,
                    cookie: Cookie that permits to associate events,
                    name: Optional name of the object being watched
                }

The `event.name` property is only present when an event is returned for a file inside a
watched directory; it identifies the file pathname relative to the watched
directory.


## Example of use
    sys     = require('sys');
    fs      = require('fs');
    Inotify = require('inotify').Inotify;

    //You can use new Inotify(false) for a non persistent fashion.
    var inotify = new Inotify(); //persistent by default

    var callback = function(event) {
        var mask = event.mask;
        var type = mask & Inotify.IN_ISDIR ? 'Directory ' : 'File ';
        event.name ? type += ' ' + event.name + ' ': ' ';

        if(mask & Inotify.IN_ACCESS) {
            sys.puts(type + 'was accessed ');
        } else if(mask & Inotify.IN_MODIFY) {
            sys.puts(type + 'was modified ');
        } else if(mask & Inotify.IN_OPEN) {
            sys.puts(type + 'was opened ');
        } else if(mask & Inotify.IN_CLOSE_NOWRITE) {
            sys.puts(type + ' opened for reading was closed ');
        } else if(mask & Inotify.IN_CLOSE_WRITE) {
            sys.puts(type + ' opened for writing was closed ');
        } else if(mask & Inotify.IN_MOVE) {
            sys.puts(type + 'was moved ');
        } else if(mask & Inotify.IN_ATTRIB) {
            sys.puts(type + 'metadata changed ');
        } else if(mask & Inotify.IN_CREATE) {
            sys.puts(type + 'created');
        } else if(mask & Inotify.IN_DELETE) {
            sys.puts(type + 'deleted');
        } else if(mask & Inotify.IN_DELETE_SELF) {
            sys.puts(type + 'watched deleted ');
        } else if(mask & Inotify.IN_MOVE_SELF) {
            sys.puts(type + 'watched moved');
        } else if(mask & Inotify.IN_IGNORED) {
            sys.puts(type + 'watch was removed');
        }
        //sys.puts(sys.inspect(event));
    }
    var home_dir = { path:      '/home/camilo',
                     watch_for: Inotify.IN_OPEN | Inotify.IN_CLOSE,
                     callback:  callback
                  };

    var home_watch_descriptor = inotify.addWatch(home_dir);

    var home2_dir = { path:      '/home/bob',
                      callback:  callback
                  };

    var home2_wd = inotify.addWatch(home2_dir);

## Inotify Events

### Watch for:
 * **Inotify.IN_ACCESS:** File was accessed (read)
 * **Inotify.IN_ATTRIB:** Metadata changed, e.g., permissions, timestamps, extended attributes, link count (since Linux 2.6.25), UID, GID, etc.
 * **Inotify.IN_CLOSE_WRITE:** File opened for writing was closed
 * **Inotify.IN_CLOSE_NOWRITE:** File not opened for writing was closed
 * **Inotify.IN_CREATE:** File/directory created in the watched directory
 * **Inotify.IN_DELETE:** File/directory deleted from the watched directory
 * **Inotify.IN_DELETE_SELF:** Watched file/directory was deleted
 * **Inotify.IN_MODIFY:** File was modified
 * **Inotify.IN_MOVE_SELF:** Watched file/directory was moved
 * **Inotify.IN_MOVED_FROM:** File moved out of the watched directory
 * **Inotify.IN_MOVED_TO:** File moved into watched directory
 * **Inotify.IN_OPEN:** File was opened
 * **Inotify.IN_ALL_EVENTS:** Watch for all kind of events
 * **Inotify.IN_CLOSE:**  (IN_CLOSE_WRITE | IN_CLOSE_NOWRITE)  Close
 * **Inotify.IN_MOVE:**  (IN_MOVED_FROM | IN_MOVED_TO)  Moves

### Additional Flags:
 * **Inotify.IN_ONLYDIR:** Only watch the path if it is a directory.
 * **Inotify.IN_DONT_FOLLOW:** Do not follow symbolics links
 * **Inotify.IN_ONESHOT:** Only send events once
 * **Inotify.IN_MASK_ADD:** Add (OR) events to watch mask for this pathname if it already exists (instead of replacing the mask).

### The following bits may be set in the `event.mask` property returned in the callback
 * **Inotify.IN_IGNORED:** Watch was removed explicitly with inotify.removeWatch(watch_descriptor) or automatically (the file was deleted, or the file system was unmounted)
 * **Inotify.IN_ISDIR:** Subject of this event is a directory
 * **Inotify.IN_Q_OVERFLOW:** Event queue overflowed (wd is -1 for this event)
 * **Inotify.IN_UNMOUNT:** File system containing the watched object was unmounted



## License
(The MIT License)

Copyright 2010 Camilo Aguilar. All rights reserved.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to
deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE SOFTWARE.


[inotify.7]: http://www.kernel.org/doc/man-pages/online/pages/man7/inotify.7.html "http://www.kernel.org/doc/man-pages/online/pages/man7/inotify.7.html"
[nodejs_home]: http://www.nodejs.org
[nodejs_dev]: http://github.com/ry/node
[code_example]: http://gist.github.com/476119

