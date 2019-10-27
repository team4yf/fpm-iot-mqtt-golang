# fpm-iot-mqtt-golang

mqtt 服务器的 golang 版本

## Features

- pub/sub Mqtt Message
- Save message into influxDB(Optionally)
- Login required
- Debug Console(Restful or Webui)

## MQTT Servers

- yfsoftcom/mqtt-server []()

    - Deploy

    ```
    docker run -d \
      -e MQTT_AUTH='{"foo":"bar"}' \
      -p 1884:1883 \
      --name fpm-mqtt-server \
      yfsoftcom/fpm-mqtt-server:latest
    ```
- emqx/emqx [github](https://github.com/emqx/emqx)

    ** WARNNING: 955 clients max **

    Run the command `$ ulimit -n 2048` can fix the issue (Reset after reboot)


    - Conf : [https://github.com/emqx/emqx/blob/master/etc/emqx.conf](https://github.com/emqx/emqx/blob/master/etc/emqx.conf)

    - Deploy 

    ```
    docker run -d --name emqx \
      -e EMQX_ALLOW_ANONYMOUS=false \
      -e EMQX_LISTENER__TCP__EXTERNAL=1883 \
      -e EMQX_LISTENER__TCP__EXTERNAL__ACCEPTORS=64 \
      -e EMQX_LISTENER__TCP__EXTERNAL__MAX___CONNECTIONS=1024000 \
      -e EMQX_NODE__PROCESS___LIMIT=2097152 \
      -e EMQX_NODE__MAX___PORTS=1048576 \
      -e EMQX_LOADED_PLUGINS="emqx_management,emqx_auth_username,emqx_recon,emqx_retainer,emqx_rule_engine,emqx_dashboard" \
      -p 18083:18083 -p 1883:1883 -p 8080:8080 \
      emqx/emqx:latest
    ```
    - Login Dashboard
    [http://localhost:18083](http://localhost:18083) with user/pass: `admin/public`

    - Change Password

    - Create App

    - Add MQTT User

      - list usernames
      ```
      curl -v --basic -u 7289cb89f9b54:MjkwMDA4NzAzNzI4NDc4NTE5NjMxNTMxNjE4ODYyMzY2NzC -H "Content-Type: application/json" \-k http://localhost:8080/api/v3/auth_username
      ```

      - add username
      ```
      curl -v --basic -u 7289cb89f9b54:MjkwMDA4NzAzNzI4NDc4NTE5NjMxNTMxNjE4ODYyMzY2NzC -H "Content-Type: application/json" -d \
        '{"username":"foo", "password": "bar"}' \-k http://localhost:8080/api/v3/auth_username
      ```

      - get username
      ```
      curl -v --basic -u 7289cb89f9b54:MjkwMDA4NzAzNzI4NDc4NTE5NjMxNTMxNjE4ODYyMzY2NzC -H "Content-Type: application/json" \-k http://localhost:8080/api/v3/auth_username/admin
      ```

    
      - pub a message
      ```
      curl -v --basic -u 7289cb89f9b54:MjkwMDA4NzAzNzI4NDc4NTE5NjMxNTMxNjE4ODYyMzY2NzC -H "Content-Type: application/json" -d \
        '{"qos":1, "retain": false, "topic":"world", "payload":"test" , "client_id": "C_1492145414740"}' \-k http://localhost:8080/api/v3/mqtt/publish
      ```

- VolantMQ/volantmq [github](https://github.com/VolantMQ/volantmq)

    Best option is to run prebuilt docker image
    ```bash
    docker run --rm -p 1883:1883 -p 8080:8080 -v $(pwd)/examples/config.yaml:/etc/volantmq/config.yaml --env VOLANTMQ_CONFIG=/etc/volantmq/config.yaml volantmq/volantmq
    ```

## MQTT Benchmark Tools

- emqx/emqtt-bench [github](https://github.com/emqx/emqtt-bench)

- krylovsk/mqtt-benchmark [github](https://github.com/krylovsk/mqtt-benchmark)

    `$ go get github.com/krylovsk/mqtt-benchmark`

    `$ mqtt-benchmark --broker tcp://localhost:1883 --count 10 --size 100 --clients 100 --qos 2 --format json --quiet`

    `$ mqtt-benchmark --broker tcp://localhost:1883 --count 10 --size 100 --clients 500 --qos 2 --format text`

    - test mosca
    `$ mqtt-benchmark --broker tcp://localhost:1884 --username foo --password bar --count 20 --size 50 --clients 50 --qos 2 --format text --quiet`

    - test emqx
    `$ mqtt-benchmark --broker tcp://localhost:1883 --username foo --password bar --count 1000 --size 1000 --clients 900 --qos 2 --format text --quiet`

    ```
    ========= TOTAL (1900) =========
    Total Ratio:                 1.000 (1900000/1900000)
    Total Runtime (sec):         119.363
    Average Runtime (sec):       118.472
    Msg time min (ms):           0.188
    Msg time max (ms):           422.708
    Msg time mean mean (ms):     117.632
    Msg time mean std (ms):      0.967
    Average Bandwidth (msg/sec): 8.441
    Total Bandwidth (msg/sec):   16038.289
    ```

- inovex/mqtt-stresser [github](https://github.com/inovex/mqtt-stresser)

    ```bash
    $ docker run inovex/mqtt-stresser -broker tcp://localhost:1883 -num-clients 100 -num-messages 10 -rampup-delay 1s -rampup-size 10 -global-timeout 180s -timeout 20s
    ```