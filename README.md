# docker-postfix
Simple Postfix SMTP TLS relay [docker](http://www.docker.com) alpine based image with no local authentication enabled (to be run in a secure LAN).

This image is available for the following architectures:

* 386
* amd64 (_latest_ and _alpine_ tags)
* armv6
* armv7
* arm64

_If you want to follow the development of this project check out [my blog](https://www.juanbaptiste.tech/category/postfx)._

### Build instructions

Clone this repo and then:

    cd docker-Postfix
    sudo docker build -t postfix:local .


### How to run it

The following env variables need to be passed to the container:

* `SMTP_SERVER` Server address of the SMTP server to use.
* `SMTP_PORT` (Optional, Default value: 587) Port address of the SMTP server to use.
* `SMTP_USERNAME` (Optional) Username to authenticate with.
* `SMTP_PASSWORD` (Mandatory if `SMTP_USERNAME` is set) Password of the SMTP user. If `SMTP_PASSWORD_FILE` is set, not needed.
* `SERVER_HOSTNAME` Server hostname for the Postfix container. Emails will appear to come from the hostname's domain.

The following env variable(s) are optional.
* `SMTP_HEADER_TAG` This will add a header for tracking messages upstream. Helpful for spam filters. Will appear as "RelayTag: ${SMTP_HEADER_TAG}" in the email headers.

* `SMTP_NETWORKS` Setting this to allow comma seperated subnets to use the relay. Used like
    -e SMTP_NETWORKS='xxx.xxx.xxx.xxx/xx,xxx.xxx.xxx.xxx/xx'

* `SMTP_PASSWORD_FILE` Setting this to a mounted file containing the password, to avoid passwords in env variables. Used like
    -e SMTP_PASSWORD_FILE=/secrets/smtp_password
    -v $(pwd)/secrets/:/secrets/

* `SMTP_USERNAME_FILE` Setting this to a mounted file containing the username, to avoid usernames in env variables. Used like
    -e SMTP_USERNAME_FILE=/secrets/smtp_username
    -v $(pwd)/secrets/:/secrets/

* `ALWAYS_ADD_MISSING_HEADERS` This is related to the [always\_add\_missing\_headers](http://www.postfix.org/postconf.5.html#always_add_missing_headers) Postfix option (default: `no`). If set to `yes`, Postfix will always add missing headers among `From:`, `To:`, `Date:` or `Message-ID:`.

* `OVERWRITE_FROM` This will rewrite the from address overwriting it with the specified address for all email being relayed. Example settings:
    OVERWRITE_FROM=email@company.com
    OVERWRITE_FROM="Your Name" <email@company.com>

* `DESTINATION` This will define a list of domains from which incoming messages will be accepted.

* `LOG_SUBJECT` This will output the subject line of messages in the log.

* `SMTPUTF8_ENABLE` This will enable (default) or disable support for SMTPUTF8. Valid values are `no` to disable and `yes` to enable. Not setting this variable will use the postfix default, which is `yes`.

* `MESSAGE_SIZE_LIMIT` This will change the default limit of 10240000 bytes (10MB).

To use this container from anywhere, the 25 port or the one specified by `SMTP_PORT` needs to be exposed to the docker host server:

    docker run -d --name postfix -p "25:25"  \
           -e SMTP_SERVER=smtp.bar.com \
           -e SMTP_USERNAME=foo@bar.com \
           -e SMTP_PASSWORD=XXXXXXXX \
           -e SERVER_HOSTNAME=helpdesk.mycompany.com \
           juanluisbaptiste/postfix

If you are going to use this container from other docker containers then it's better to just publish the port:

    docker run -d --name postfix -P \
           -e SMTP_SERVER=smtp.bar.com \
           -e SMTP_USERNAME=foo@bar.com \
           -e SMTP_PASSWORD=XXXXXXXX \
           -e SERVER_HOSTNAME=helpdesk.mycompany.com \           
           juanluisbaptiste/postfix

Or if you can start the service using the provided [docker-compose](https://github.com/juanluisbaptiste/docker-postfix/blob/master/docker-compose.yml) file for production use:

    sudo docker-compose up -d

To see the email logs in real time:

    docker logs -f postfix

### Debugging
If you need troubleshooting the container you can set the environment variable _DEBUG=yes_ for a more verbose output.
