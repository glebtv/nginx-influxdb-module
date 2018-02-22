# NGINX InfluxDB Module :unicorn:

A module to monitor request passing trough an NGINX server by sending
every request to an InfluxDB backend exposing UDP.

## Exported Fields (per request)

| Metric                | Type    | Description                                                                                                                               |
|-----------------------|---------|-------------------------------------------------------------------------------------------------------------------------------------------|
| method                | string  | The HTTP request method that has been given as a reply to the caller                                                                      |
| status                | integer | The HTTP status code of the reply from the server (refer to [RFC 7231](https://tools.ietf.org/html/rfc7231#section-6.1) for more details) |
| connection_bytes_sent | integer | Bytes sent by the current connection considering all the buffers                                                                          |
| body_bytes_sent       | integer | Bytes sent for the body only                                                                                                              |
| header_bytes_sent     | integer | Bytes sent for the header only                                                                                                            |
| request_length        | integer | The length of the headers sent by the client                                                                                              |
| uri                   | string  | The called uri (e.g: /index.html)                                                                                                         |
| extension             | string  | The extension of the served file (e.g: js, html, php, png)                                                                                |
| content_type          | string  | The content type of the response (e.g: text/html)                                                                                         |


## Installation

## From pre-built dynamic modules

Pre-built dynamic modules are not available (yet)

## Build the module statically with NGINX

```
mkdir build
pushd build
git clone git@github.com:fntlnz/nginx-influxdb-module.git
wget -nv -O - http://nginx.org/download/nginx-1.13.7.tar.gz | tar zx
pushd nginx-1.13.7
./configure --add-module=../nginx-influxdb-module
make -j
popd
popd
```


## Build the module dinamically to be loaded within NGINX

Writing this in a snap!

## Configuration

To configure it you just need this one line configuration.

```
influxdb server_name=myserver host=127.0.0.1 port=8089 measurement=mymeasures enabled=true;
```


### Full Example

A full example config looks like this

```nginx
user  nginx;
worker_processes  auto;
daemon off;

#load_module path/to/the/module/ngx_http_influxdb_module.so; needed if a dynamic module
error_log  /var/log/nginx/error.log debug;
pid        /var/run/nginx/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /tmp/access.log main;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       8080;
        server_name  localhost;
        location / {
            root   /usr/share/nginx/html;
            influxdb server_name=myserver host=127.0.0.1 port=8089 measurement=mymeasures;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }

}

```


## Contributing

### Run Unit the tests

If you are willing to contribute to a patch, you're required to test what you did
whenever possible.

This project uses the [Google Test](https://github.com/google/googletest) testing framework.

The build of the module itself is done trough NGINX and its build model. However, for testing we use
`CMake` - so if you want to test this module you need the following dependencies:

- CMake >= 3.3
- Google Test
- Google Mock
- gcc
- gcc-c++


Now that you have the dependencies you can test your changes with:

```bash
cd test
cmake .
make
make test
```


### Run the NGINX Integration tests

The integration tests are actually using the official nginx tests patched
on the fly to use the module and write metrics to Influx.

To run the tests you actually need an influxdb instance to accept data:

```
docker run -d --name influxdb -p 8086:8086 -p 8089:8089/udp -v $PWD/integration/influxdb.conf:/etc/influxdb/influxdb.conf:ro influxdb
```

Then you can run the test script

```bash
./integration/nginx-tests.sh
```

After all this you will se all the points written to the influx instance
