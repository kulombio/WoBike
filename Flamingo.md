# Flamingo
[Flamingo](https://www.flamingoscooters.com/) is a E-Scooter sharing service that operates in New Zealand.

Here's a published Postman collection that has all the request in it: https://documenter.getpostman.com/view/11220018/TVKD2xdh

## Request OTP Code

**Method**: `POST`

**URL**: `https://www.googleapis.com/identitytoolkit/v3/relyingparty/sendVerificationCode`

**Headers**:

| Headers                  | Value                                 | Mandatory |
| ------------------------ | ------------------------------------- | :-------: |
| X-Ios-Bundle-Identifier  | com.flamingoscooters.ios              | X         |

**Params**:

| Params        | Value                                  | Mandatory |
| ------------- | -------------------------------------- | :-------: |
| key           | AIzaSyCrkmrEuGrn9tgWfCY2rcpd-LRAnsE84ik| X         |

**The Body**:

`{"iosReceipt":"AEFDNu-Htn7DOuqCRwPrpDA7lMKkUHJduCQPNpel6sVYukJIBM6g_ZoJxyKK__G41ND_-uEkoTQvulRec3wZXXuXxEgfpbcOF8Dftg1eLFHRiZWsCfzbezjYe1P7rx2RIBG7c2V_","iosSecret":"WlQ2n-YexIkrNlpZ","phoneNumber":"+64<YOUR-PHONE-NUMBER>"}`

As a result an OTP code should have been texted to and the response should include the session id:

```
{
    "sessionInfo": "<LONG-STRING>"
}
```

## Send back OTP code

**Method**: `POST`

**URL**: `https://www.googleapis.com/identitytoolkit/v3/relyingparty/verifyPhoneNumber`

**Headers**:

| Headers                  | Value                                 | Mandatory |
| ------------------------ | ------------------------------------- | :-------: |
| X-Ios-Bundle-Identifier  | com.flamingoscooters.ios              | X         |

**Params**:

| Params        | Value                                  | Mandatory |
| ------------- | -------------------------------------- | :-------: |
| key           | AIzaSyCrkmrEuGrn9tgWfCY2rcpd-LRAnsE84ik| X         |

**The Body**:

`{"code":"<OTP-CODE>","operation":"SIGN_UP_OR_IN","sessionInfo":"<SESSION-ID>"}`

This should use the OTP Code and grant you an Access and Refresh Token

```
{
    "idToken": "<ID-TOKEN>", --- Your temporary Access Token
    "refreshToken": "<REFRESH-TOKEN>",
    "expiresIn": "3600",
    "localId": "<LOCAL-ID>",
    "isNewUser": false,
    "phoneNumber": "+64<YOUR-PHONE-NUMBER>"
}
```

### Add User Info

:warning: This only applies to accounts that haven't already been created, but it is recommended that you setup your account in the Flamingo app and not sign up through their API. :warning:

**Method**: `POST`

**URL**: `https://api.flamingoscooters.com/sign-up`

**Headers**:

| Headers                  | Value                                 | Mandatory |
| ------------------------ | ------------------------------------- | :-------: |
| X-Ios-Bundle-Identifier  | com.flamingoscooters.ios              | X         |
| Authorization            | USER-ID                               | X         |

**Params**:

| Params        | Value                                  | Mandatory |
| ------------- | -------------------------------------- | :-------: |
| key           | AIzaSyCrkmrEuGrn9tgWfCY2rcpd-LRAnsE84ik| X         |

**The Body**:

`{"lastName":"<YOUR-LAST-NAME>","email":"<YOUR-EMAIL>","firstName":"<YOUR-FIRST-NAME>","dateOfBirth":"1990-09-20T19:32:00+12:00"}`


## Refresh Token

This will renew your access token since it will expire after an hour. The refresh token is basically an account password, anyone with your Refresh Token can request an access token and login with it. So if you want to intergrate the Flamingo API into a public app, use a burner account and not your personal one.

**Method**: `POST`

**URL**: `https://securetoken.googleapis.com/v1/token`

**Headers**:

| Headers                  | Value                                 | Mandatory |
| ------------------------ | ------------------------------------- | :-------: |
| X-Ios-Bundle-Identifier  | com.flamingoscooters.ios              | X         |

**Params**:

| Params        | Value                                  | Mandatory |
| ------------- | -------------------------------------- | :-------: |
| key           | AIzaSyCrkmrEuGrn9tgWfCY2rcpd-LRAnsE84ik| X         |

**The Body**:

`{"grantType":"refresh_token","refreshToken":"<YOUR-REFRESH-TOKEN>"}`

This should verify the code and grant you a Access and Refresh Token. Here's the output:

```
{
    "access_token": "<YOUR-ACCESS-TOKEN>",
    "expires_in": "3600", --- 1 HOUR
    "token_type": "Bearer",
    "refresh_token": "<YOUR-REFRESH-TOKEN>",
    "id_token": "<YOUR-ID-TOKEN>",
    "user_id": "<YOUR-USER-ID>",
    "project_id": "663814096530"
}
```

## Get Scooter Locations

After finally logging in, we can now retreive locations of scooters and zones

**URL**: `https://api.flamingoscooters.com/vehicle/area`

**Method**: `GET`

**Headers**:

| Headers       | Value                                 | Mandatory |
| ------------  | ------------------------------------- | :-------: |
| Authorization | YOUR-ACCESS-TOKEN                     | X         |

**Parameters**:

| Parameters   | Descriptions             | Mandatory |
| ------------ | ------------------------ | :-------: |
| neLatitude   | latitude                 | X         |
| neLongitude  | longitude                | X         |
| swLatitude   | latitude                 | X         |
| swLongitude  | longitude                | X         |
| zoom         | zoom integer             | X         |


The output should be something like this:

```
{
    "success": true,
    "data": [
        {
            "id": 110, --- ID to send commands to the scooter
            "type": "SCOOTER",
            "powerPercent": 86,
            "remainingRange": 1608, --- Meter Range
            "latitude": -43.524343,
            "longitude": 172.579243,
            "registration": "1961", --- Scooter ID that you can use to find the scooter on the street
            "regionId": 4,
            "updatedAt": "2020-09-20T07:53:38.000Z", --- Last Signal
            "lastActivity": "2020-09-19T03:45:52.000Z" --- Last Ride
        },
        ...
```

## Get Zones and area info

**URL**: `https://api.flamingoscooters.com/region/all`

**Method**: `GET`

**Headers**:

| Headers       | Value                                 | Mandatory |
| ------------  | ------------------------------------- | :-------: |
| Authorization | YOUR-ACCESS-TOKEN                     | X         |

The output should be something like this:

```
{
    "success": true,
    "data": [
        {
            "id": 2,
            "name": "Auckland",
            "country": "NZ",
            "centreLatitude": -36.848619,
            "centreLongitude": 174.76429,
            "message": "Flamingo Auckland - Tap to Learn More",
            "messageUrl": "https://support.flamingoscooters.com/webview/auckland",
            "timezone": "Pacific/Auckland",
            "startTime": "03:00",
            "endTime": "03:01",
            "serviceArea": [
                {
                    "latitude": -36.724571,
                    "longitude": 174.819065
                },
                {
                    "latitude": -36.872732,
                    "longitude": 174.896842
                },
                {
                    "latitude": -36.922322,
                    "longitude": 174.844494
                },
                {
                    "latitude": -36.932023,
                    "longitude": 174.79019
                },
                {
                    "latitude": -36.930775,
                    "longitude": 174.706669
                },
                {
                    "latitude": -36.789123,
                    "longitude": 174.666784
                }
            ],
            "fare": {
                "unlock": 1,
                "perMin": 0.38,
                "currency": "NZD",
                "wording": "$1 NZD to unlock, 38c per min"
            },
        ...
```

### Get Scooter Parking Photo

Use the `id` from the scooter output and not the `registration` id.

**URL**: `https://api.flamingoscooters.com/vehicle/<vehicle-id>/last-photo`

**Method**: `GET`

**Headers**:

| Headers       | Value                                 | Mandatory |
| ------------  | ------------------------------------- | :-------: |
| Authorization | YOUR-ACCESS-TOKEN                     | X         |

This should output the URL for the Parking Photo. Some scooters may not have one, here's an example from a scooter that does have one:

```
{
    "success": true,
    "data": {
        "photoUrl": "https://storage.googleapis.com/flamingo-ugc/JmpviKnyIwwXQQRRmyEJkyEaoasFZpgM.jpg"
    }
}
```

## GBFS

Use Flamingo's GBFS API to get scooter locations instead of having to authenticate with their main API.

**URL**: `https://api.flamingoscooters.com/gbfs/<city>/free_bike_status.json`

Replace the `<city>` in the URL with either: `auckland`, `christchurch` or `wellington`, depending on which city.

The output should be something like this:

```
{
                "bike_id": "1339",
                "lat": -43.532093,
                "lon": 172.628018,
                "current_range_meters": 11900,
                "last_reported": 1599977150
            },
```

### Meanings from output

`bike_id`: The code that lets riders identify the scooter and unlock it

`lat`: Latitude location of scooter

`lon`: Longitude location of scooter

`current_range_meters`: Battery level of scooter in meters

`last_reported`: The date and time for when the scooter last sent a GPS Ping, this would need to be decoded

## Implementations

https://openscootermap.netlify.app/
