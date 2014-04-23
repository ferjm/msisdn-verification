# Obtaining the MSISDN
## Getting it from the SIM card
## MO SMS
## MO Call ?
## Asking the user

# Verification mechanisms
## Network based authentication
## SMS based authentication
### MT only
### MO and MT
## Telephony call based authentication

# API Endpoints
  * [POST /v1/msisdn/register](#post-v1msisdnregister)
  * [POST /v1/msisdn/unregister](#post-v1msisdnunregister)
  * [POST /v1/msisdn/network/verify](#post-v1msisdnnetworkverify)
  * [POST /v1/msisdn/telephony/verify](#post-v1msisdntelephonyverify)
  * [POST /v1/msisdn/sms/verify](#post-v1msisdnsmsverify)
  * [POST /v1/msisdn/sms/verify_code](#post-v1msisdnverify_code)
  * [POST /v1/msisdn/sms/resend_code](#post-v1msisdnresend_code)

## POST /v1/msisdn/register

Starts the registration of a MSISDN in E.164 format. The verification service checks the available verification mechanism according to the given network information (mcc, mnc and roaming) and replies back with a session token and a verification URL corresponding to the chosen verification mechanism.

___Parameters___
* msisdn - a MSISDN in E.164 format
* mcc - [Mobile Country Code](http://es.wikipedia.org/wiki/MCC/MNC)
* mnc - [Mobile Network Code](http://es.wikipedia.org/wiki/MCC/MNC)
* roaming - boolean that indicates if the device is on roaming or not

### Request

```sh
curl -v \
-X POST \
-H "Content-Type: application/json" \
"https://api.accounts.firefox.com/v1/msisdn/register" \
-d '{
  "msisdn": "+442071838750",
  "mcc": "214",
  "mnc": "07",
  "roaming": false
}'
```

### Response

Successful requests will produce a "200 OK" response with following format:

```json
{
  "msisdnSessionToken": "27cd4f4a4aa03d7d186a2ec81cbf19d5c8a604713362df9ee15c4f4a4aa03d7d",
  "verificationUrl": "https://api.accounts.firefox.com/v1/msisdn/sms/verify"
}
```
## POST /v1/msisdn/network/verify

### Request

:lock: HAWK-authenticated with a `msisdnSessionToken`.

```sh
curl -v \
-X POST \
-H "Content-Type: application/json" \
"https://api.accounts.firefox.com/v1/msisdn/network/verify" \
-H 'Authorization: Hawk id="d4c5b1e3f5791ef83896c27519979b93a45e6d0da34c7509c5632ac35b28b48d", ts="1373391043", nonce="ohQjqb", hash="vBODPWhDhiRWM4tmI9qp+np+3aoqEFzdGuGk0h7bh9w=", mac="LAnpP3P2PXelC6hUoUaHP72nCqY5Iibaa3eeiGBqIIU="' \
-d '{}'
```

### Response

Successful requests will produce a "200 OK" response with following format:

```json
{
  "cert": "eyJhbGciOiJEUzI1NiJ9.eyJwdWJsaWMta2V5Ijp7ImFsZ29yaXRobSI6IlJTIiwibiI6IjU3NjE1NTUwOTM3NjU1NDk2MDk4MjAyMjM2MDYyOTA3Mzg5ODMyMzI0MjUyMDY2Mzc4OTA0ODUyNDgyMjUzODg1MTA3MzQzMTY5MzI2OTEyNDkxNjY5NjQxNTQ3NzQ1OTM3NzAxNzYzMTk1NzQ3NDI1NTEyNjU5NjM2MDgwMzYzNjE3MTc1MzMzNjY5MzEyNTA2OTk1MzMyNDMiLCJlIjoiNjU1MzcifSwicHJpbmNpcGFsIjp7ImVtYWlsIjoiZm9vQGV4YW1wbGUuY29tIn0sImlhdCI6MTM3MzM5MjE4OTA5MywiZXhwIjoxMzczMzkyMjM5MDkzLCJpc3MiOiIxMjcuMC4wLjE6OTAwMCJ9.l5I6WSjsDIwCKIz_9d3juwHGlzVcvI90T2lv2maDlr8bvtMglUKFFWlN_JEzNyPBcMDrvNmu5hnhyN7vtwLu3Q"
}
```

The signed certificate includes these additional claims:

* fxa-verifiedMSISDN - the user's verified MSISDN
* fxa-lastVerifiedAt - time of last MSISDN verification (seconds since epoch)

## POST /v1/msisdn/telephony/verify

:lock: HAWK-authenticated with a `msisdnSessionToken`.

## POST /v1/msisdn/sms/verify

:lock: HAWK-authenticated with a `msisdnSessionToken`.

## POST /v1/msisdn/sms/verify_code

:lock: HAWK-authenticated with a `msisdnSessionToken`.

This verifies the SMS code sent to a MSISDN. The server is given a public key, and returns a signed certificate using the same JWT-like mechanism as a BrowserID primary IdP would (see the [browserid-certifier project](https://github.com/mozilla/browserid-certifier for details)). The signed certificate includes a `principal.email` property to indicate a "Firefox Account-like" identifier (a uuid at the account server's primary domain). TODO: add discussion about how this id will likely *not* be stable for repeated calls to this endpoint with the same MSISDN (alone), but probably stable for repeated calls with the same MSISDN+`msisdnSessionToken`.

___Parameters___
* msisdn - a MSISDN in E.164 format
* code - the SMS verification code sent to the MSISDN
* publicKey - the key to sign (run `bin/generate-keypair` from [jwcrypto](https://github.com/mozilla/jwcrypto))
    * algorithm - "RS" or "DS"
    * n - RS only
    * e - RS only
    * y - DS only
    * p - DS only
    * q - DS only
    * g - DS only
* duration - time interval from now when the certificate will expire in seconds

### Request

The request must include a Hawk header that authenticates the request (including payload) using a `msisdnSessionToken` received from `/v1/msisdn/register`.

```sh
curl -v \
-X POST \
-H "Content-Type: application/json" \
"https://api.accounts.firefox.com/v1/msisdn/verify_code" \
-H 'Authorization: Hawk id="d4c5b1e3f5791ef83896c27519979b93a45e6d0da34c7509c5632ac35b28b48d", ts="1373391043", nonce="ohQjqb", hash="vBODPWhDhiRWM4tmI9qp+np+3aoqEFzdGuGk0h7bh9w=", mac="LAnpP3P2PXelC6hUoUaHP72nCqY5Iibaa3eeiGBqIIU="' \
-d '{
  "msisdn": "+442071838750",
  "code": "e3c5b0e3f5391e134596c27519979b93a45e6d0da34c7509c5632ac35b28b48d",
  "publicKey": {
    "algorithm":"RS",
    "n":"4759385967235610503571494339196749614544606692567785790953934768202714280652973091341316862993582789079872007974809511698859885077002492642203267408776123",
    "e":"65537"
  },
  "duration": 86400000
}'
```

### Response

Successful requests will produce a "200 OK" response with following format:

```json
{
  "cert": "eyJhbGciOiJEUzI1NiJ9.eyJwdWJsaWMta2V5Ijp7ImFsZ29yaXRobSI6IlJTIiwibiI6IjU3NjE1NTUwOTM3NjU1NDk2MDk4MjAyMjM2MDYyOTA3Mzg5ODMyMzI0MjUyMDY2Mzc4OTA0ODUyNDgyMjUzODg1MTA3MzQzMTY5MzI2OTEyNDkxNjY5NjQxNTQ3NzQ1OTM3NzAxNzYzMTk1NzQ3NDI1NTEyNjU5NjM2MDgwMzYzNjE3MTc1MzMzNjY5MzEyNTA2OTk1MzMyNDMiLCJlIjoiNjU1MzcifSwicHJpbmNpcGFsIjp7ImVtYWlsIjoiZm9vQGV4YW1wbGUuY29tIn0sImlhdCI6MTM3MzM5MjE4OTA5MywiZXhwIjoxMzczMzkyMjM5MDkzLCJpc3MiOiIxMjcuMC4wLjE6OTAwMCJ9.l5I6WSjsDIwCKIz_9d3juwHGlzVcvI90T2lv2maDlr8bvtMglUKFFWlN_JEzNyPBcMDrvNmu5hnhyN7vtwLu3Q"
}
```

The signed certificate includes these additional claims:

* fxa-verifiedMSISDN - the user's verified MSISDN
* fxa-lastVerifiedAt - time of last MSISDN verification (seconds since epoch)

## POST /v1/msisdn/sms/resend_code

:lock: HAWK-authenticated with a `msisdnSessionToken`.

This triggers the sending of an SMS code the MSISDN registered in /v1/msisdn/register. 

___Parameters___
* msisdn - a MSISDN in E.164 format

### Request

The request must include a Hawk header that authenticates the request (including payload) using a `msisdnSessionToken` received from `/v1/msisdn/register`.

```sh
curl -v \
-X POST \
-H "Content-Type: application/json" \
"https://api.accounts.firefox.com/v1/msisdn/resend_code" \
-H 'Authorization: Hawk id="d4c5b1e3f5791ef83896c27519979b93a45e6d0da34c7509c5632ac35b28b48d", ts="1373391043", nonce="ohQjqb", hash="vBODPWhDhiRWM4tmI9qp+np+3aoqEFzdGuGk0h7bh9w=", mac="LAnpP3P2PXelC6hUoUaHP72nCqY5Iibaa3eeiGBqIIU="' \
-d '{
  "msisdn": "+442071838750"
}'
```

### Response

Successful requests will produce a "200 OK" response with following format:

```json
{}
```

## POST /v1/msisdn/unregister

:lock: HAWK-authenticated with a `msisdnSessionToken`.

This completely removes a previously registered MSISDN.

___Parameters___
* msisdn - a MSISDN in E.164 format

### Request

The request must include a Hawk header that authenticates the request (including payload) using a `msisdnSessionToken` received from `/v1/msisdn/register`.

```sh
curl -v \
-X POST \
-H "Content-Type: application/json" \
"https://api.accounts.firefox.com/v1/msisdn/unregister" \
-H 'Authorization: Hawk id="d4c5b1e3f5791ef83896c27519979b93a45e6d0da34c7509c5632ac35b28b48d", ts="1373391043", nonce="ohQjqb", hash="vBODPWhDhiRWM4tmI9qp+np+3aoqEFzdGuGk0h7bh9w=", mac="LAnpP3P2PXelC6hUoUaHP72nCqY5Iibaa3eeiGBqIIU="' \
-d '{
  "msisdn": "+442071838750"
}'
```

### Response

Successful requests will produce a "200 OK" response with following format:

```json
{}
```
