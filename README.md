# ns8-paperless ngx

This is a ns8-paperless-ngx module for [NethServer 8](https://github.com/NethServer/ns8-core).
This Modules uses: [PAperless NGX](https://github.com/paperless-ngx/paperless-ngx). 

1. Rename some references inside the repo:
   ```
   modulename=$(basename $(pwd) | sed 's/^ns8-//')
   git mv imageroot/systemd/user/ns8-paperless-ngx.service imageroot/systemd/user/${modulename}.service
   git mv tests/ns8-paperless-ngx.robot tests/${modulename}.robot
   sed -i "s/ns8-paperless-ngx/${modulename}/g" $(find .github/ * -type f)
   git commit -a -m "Repository initialization"
   ```

1. Edit this `README.md` file, by replacing this section with your module
   description

1. Adjust `.github/workflows` to your needs. `clean-registry.yml` might
   need the proper list of image names to work correctly. Unused workflows
   can be disabled from the GitHub Actions interface.

1. Commit and push your local changes

## Install

Instantiate the module with:

    add-module ghcr.io/compgeniuses/ns8-paperless-ngx:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "ns8-paperless-ngx1", "image_name": "ns8-paperless-ngx", "image_url": "ghcr.io/compgeniuses/ns8-paperless-ngx:latest"}

## Configure

Let's assume that the paperless instance is named `ns8-paperless-ngx1`.

Launch `configure-module`, by setting the following parameters:
- `paperless_name`: the name given to the instance that wil also appear as the name on the dachboard
- `PAPERLESS_SECRET_KEY`: a generated secret key that will be used as the project security key
  `host`: the traefik host url for the project
- `PAPERLESS_TIME_ZONE`: the timezone for the project, a config that can be modified
- `PAPERLESS_OCR_LANGUAGE`: the default OCR language for when files are scanned
- ...

Example:

    api-cli run module/paperlessngx1/configure-module --data '{"host": "paperlessngx.domain.com"}'

The above command will:
- start and configure the ns8-paperless-ngx instance
- (describe configuration process)
- ...

Send a test HTTP request to the ns8-paperless-ngx backend service:

    curl http://127.0.0.1/ns8-paperless-ngx/

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time
kickstart starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when ns8-paperless-ngx is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/ns8-paperless-ngx.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve ns8-paperless-ngx1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/ns8-paperless-ngx:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
