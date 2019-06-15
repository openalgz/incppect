[![Build Status](https://travis-ci.org/ggerganov/incppect.svg?branch=master)](https://travis-ci.org/ggerganov/incppect?branch=master)

# incppect

Inspect C++ memory in the browser

## Description

This is a small library that allows a C++ native application to stream memory bits to one or more websocket clients. This functionality can be used to conveniently inspect the internal state of the native application from a browser.

Incppect starts a simple HTTP/WebSocket server in your application that accepts external connections. When a client connects, incppect serves the static content (HTML/JS) from a user-specified location, as well as the built-in incppect JS client [incppect.js](https://github.com/ggerganov/incppect/blob/master/src/incppect.js). The client opens a websocket connection back to the application and starts requesting custom data. The data is streamed back to the client over the websocket. The usage/visualization of the received data is entirely up to the client.

The HTTP/WebSocket server is implemented via the [uWebSocket](https://github.com/uNetworking/uWebSockets) library (included in the project as a submodule).

## Example:

<a href="https://i.imgur.com/8hJSbzQ.gif" target="_blank">![incppect-balls2d](https://i.imgur.com/8hJSbzQ.gif)</a>

Checkout the [examples](https://github.com/ggerganov/incppect/tree/master/examples) folder for more samples.

## Sample usage:

In your C++ program add something along these lines:

```
#include "incppect/incppect.h"

// start the web server in a dedicated thread
auto & incppect = Incppect::getInstance();
incppect.runAsync(...);

int32_t some_var;
float some_arr[10];
    
// define variables that can be requested from the web clients
incppect.var("path0", [&](auto ) { return Incppect::view(some_var); });
incppect.var("path1[%d]", [&](auto idxs) { return Incppect::view(some_arr[idxs[0]]); });

```

In your web client:

```
<script src="incppect.js"></script>

<script>
    incppect.render = function() {
        // request C++ data
        var some_var = this.get_int32('path0');
        var some_arr_element = this.get_int32_arr('path1[%d]', 5);
        
        // do something with it
        ...
    }
    
    incppect.init();
</script>

```

## Build instructions

**Linux and Mac OS**

    git clone https://github.com/ggerganov/incppect
    cd incppect
    git submodule update --init
    mkdir build && cd build
    cmake ..
    make