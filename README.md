## This is __NOT__ the source code repo. It contains only binary releases for arm32 and arm64. This doc is about deployment
## deployment binaries
- aarch32: https://github.com/lzbgt/evsuits_release/releases/tag/aarch32-latest
- aarch64: https://github.com/lzbgt/evsuits_release/releases/tag/aarch64-latest

## steps
- download aarch[32|64]_[bin|libs].tar for your board
- extract the bin and libs tar to a dedicated path, e.g. /root/work/opencv-pocs/opencv-motion-detect/
  ```
  mkdir -p /root/work/opencv-pocs/opencv-motion-detect
  tar xf aarch[x]_bin.tar --strip-components=1 -C /root/work/opencv-pocs/opencv-motion-detect
  mkdir -p /root/work/opencv-pocs/opencv-motion-detect/vendor
  tar xf aarch[x]_libs.tar --strip-components=1 -C /root/work/opencv-pocs/opencv-motion-detect/vendor
  ```
- touch new /etc/systemd/system/evdaemon.service with below content:
  ```
    [Unit]
    Description=evdaemon manages edge evsuits
    After=network.target

    [Service]
    Type=simple
    Environment="LD_LIBRARY_PATH=/root/work/opencv-pocs/opencv-motion-detect/vendor/lib" "CLOUD_ADDR=tcp://evcloud.ilabservice.cloud:5548"
    ExecStart=/root/work/opencv-pocs/opencv-motion-detect/evdaemon
    WorkingDirectory=/root/work/opencv-pocs/opencv-motion-detect
    Restart=always
    StandardOutput=syslog
    StandardError=syslog
    SyslogIdentifier=evsuits

    [Install]
    WantedBy=multi-user.target
  ```
- enable and start the service
  ```
  systemctl enable evdaemon.service; systemctl start evdaemon
  ```
- check logs to get the device SN for configuration
  ```
  tail -10000 /var/log/syslog|grep evsuit

  Output: Oct 23 00:01:22 raspberrypi evsuits[6232]: [2019-10-23 00:01:22.002] [info] evdaemon 0017SRTC received ...
  ```
  something linke "0017SRTC" is the device SN.
- configure the cluster by posting on evcloud.ilabservice.cloud:8089/config or by control center platform.

