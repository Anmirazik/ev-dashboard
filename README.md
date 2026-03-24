# Open e-Mobility Angular Dashboard App

## Summary

The Angular dashboard connects to the [Open e-Mobility NodeJs Server](https://github.com/sap-labs-france/ev-server) to display the charging stations in real time.

The application features:

* Charging Stations details and real-time statuses
* Charging sessions curves in real time
* Charging stations remote control (Reboot, Clear Cache, Stop Transaction, Unlock Connector)
* Charging Station Template management: Zero configuration
* User management
* Badge management
* Role management (ABAC)
* Static Energy Management: Manually limit the charging station
* Smart Charging with Assets, Fair Sharing, Peak Shaving, Cost Management and Phase Balancing
* Realtime Asset Management (Building, Battery, Solar Panel) 
* Billing with Stripe
* Complex Pricing
* Roaming integration (Gire, Hubject)
* Refunding (SAP Concur)
* Simple Statistics + Advanced Analytics (SAP Analytics)
* Car Connector Management (Get the car's data to optimize the charging session)

**Contact the author** <a href="https://www.linkedin.com/in/serge-fabiano-a420a218/" target="_blank">Serge FABIANO</a>

## Installation

* Install NodeJS: https://nodejs.org/ (install the LTS version)
* Clone this GitHub project
* Go into the **ev-dashboard** directory and run **npm install** or **yarn install** (use sudo in Linux)

**NOTE**:

* On Windows with **chocolatey** (https://chocolatey.org/), do as an administrator:

```powershell
choco install -y nodejs-lts
```

* On Mac OSX with **Homebrew** (https://brew.sh/), do:

```shell
brew install node
```

* Follow the rest of the setup below

## The Dashboard

#### Configuration

**`src/assets/config.json` is already committed** and pre-configured for local Docker development — no manual copy or edit needed to get started.

For reference, a template is also provided at `src/assets/config-template.json`. Use it as a base when configuring a custom or production environment.

The committed `config.json` is set up as follows:

```json
{
  "CentralSystemServer": {
    "protocol": "http",
    "host": "localhost",
    "port": 8081
  },
  "User": {
    "captchaSiteKey": "6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI"
  }
}
```

- **Port 8081** matches the ev-server Docker REST port
- **captchaSiteKey** is Google's public test reCAPTCHA key, paired with the test secret key already set in `ev-server/docker/config.json`

#### Connect to the Central Service REST Server (CSRS)

The dashboard calls the ev-server REST API. If you need to point to a different backend, update `CentralSystemServer` in `src/assets/config.json`:

```json
  "CentralSystemServer": {
    "protocol": "http",
    "host": "localhost",
    "port": 8081
  }
```

### Create and set a Google Maps API key
Ev-dashboard requires you to setup a Google API key: https://developers.google.com/maps/documentation/javascript/get-api-key#restrict_key.
Once the key is created it must be enabled (from the Google Console) and the value must replace the one present in /src/index.html, in Google Maps section:

	src="https://maps.googleapis.com/maps/api/js?key=<YOUR_KEY_HERE>&libraries=places&language=en"></script>

### Setup the reCaptcha API key
The committed `config.json` already includes Google's public test reCAPTCHA site key which works out of the box with the local Docker ev-server setup.

For production, replace it with your own key from https://www.google.com/recaptcha/admin/create and set the paired server key in ev-server's `CentralSystemRestService.captchaSecretKey`:

```json
  "User": {
    "captchaSiteKey": "<YOUR_GOOGLE_RECAPTCHA_SITE_KEY>"
  }
```

## Start the Dashboard Server

### Development Mode

```shell
npm start
```

### Production Mode

First build the sources with:
```shell
npm run build:prod
```

Next, start the server with:
```shell
npm run start:prod
```

### Secured Production Mode (SSL)

Build the sources as above and run it with:
```shell
npm run start:prod:ssl
```

## Integration tests

To run integration tests, you first need to start the UI and run the below command:
```shell
npm run test
```

This will run all integraiton tests written with **Jest** framework.

## License

This file and all other files in this repository are licensed under the Apache Software License, v.2 and copyrighted under the copyright in [NOTICE](NOTICE) file, except as noted otherwise in the [LICENSE](LICENSE) file.

Please note that Docker images can contain other software which may be licensed under different licenses. This LICENSE and NOTICE files are also included in the Docker image. For any usage of built Docker images please make sure to check the licenses of the artifacts contained in the images.
