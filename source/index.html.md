---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Contact us</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the EVEN API
---

# Introduction

Welcome to the integration guide for onboarding artists, album and releases onto the EVEN platform.

This document provides detailed instructions on you can deliver artist data to EVEN.

This integration adheres to DDEX standards, ensuring compliance with industry best practices.

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help or get a Developer Access Key.


# Authentication

> To generate a valid signed authorization header, use this code:

```ruby
require 'time'
require 'openssl'
require 'base64'

def sign_headers(method:, endpoint:, date:)
  access_id = `<ACCESS_ID>`
  secret_key =  `<SECRET_KEY>`

  method = method.to_s.upcase
  content_type = "application/json"
  digest = OpenSSL::Digest.new("SHA256")

  canonical_string = "#{method},#{content_type},,#{endpoint},#{date}"
  signature = Base64.strict_encode64(OpenSSL::HMAC.digest(digest, secret_key, canonical_string))

  "APIAuth-HMAC-SHA256 #{access_id}:#{signature}"
end

date = Time.now.utc.httpdate
endpoint = "/api/v1/distribution/artists"

signed_auth_header = sign_headers(method: :get, endpoint: endpoint, date: date)
puts signed_auth_header
```

```python
import hashlib
import hmac
import base64
from datetime import datetime

def sign_headers(method, endpoint, date):
    access_id = '<ACCESS_ID>'
    secret_key = '<SECRET_KEY>'
    
    method = method.upper()
    content_type = "application/json"
    digest = hashlib.sha256

    canonical_string = f"{method},{content_type},,{endpoint},{date}"
    signature = base64.b64encode(hmac.new(secret_key.encode(), canonical_string.encode(), digest).digest()).decode()

    return f"APIAuth-HMAC-SHA256 {access_id}:{signature}"

date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
endpoint = "/api/v1/distribution/artists"

signed_auth_header = sign_headers(method='get', endpoint=endpoint, date=date)
print(signed_auth_header)
```

```javascript
const crypto = require('crypto');

function signHeaders(method, endpoint, date) {
    const accessId = '<ACCESS_ID>';
    const secretKey = '<SECRET_KEY>'; 
    
    method = method.toUpperCase();
    const contentType = 'application/json';
    const canonicalString = `${method},${contentType},,${endpoint},${date}`;
    
    const hmac = crypto.createHmac('sha256', secretKey);
    hmac.update(canonicalString);
    const signature = hmac.digest('base64');

    return `APIAuth-HMAC-SHA256 ${accessId}:${signature}`;
}

const date = new Date().toUTCString();
const endpoint = '/api/v1/distribution/artists';

const signedAuthHeader = signHeaders('GET', endpoint, date);
console.log(signedAuthHeader);
```

> Make sure to replace `<ACCESS_ID>` and `<SECRET_KEY>` with your API access key.

EVEN uses `<ACCESS_ID>` and `<SECRET_KEY>` to allow access to the API. 
You can contact <a href= "mailto: support@even.biz"> support@even.biz</a> for get a Developer Access Key.

EVEN expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: <SIGNED_AUTHORIZATION_HEADER>`

<aside class="notice">
You must replace <code>SIGNED_AUTHORIZATION_HEADER</code>.
</aside>

# Distribution API

## Get All Artists

```ruby
require 'net/http'
require 'json'

uri = URI("https://api-staging.even.biz/api/v1/distribution/artists")

# Prepare HTTP request
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Get.new(uri.path, 'Content-Type' => 'application/json')

# Generate required headers
date = Time.now.utc.httpdate
signed_auth_header = sign_headers(method: :get, endpoint: uri.path, date: date)

# Set custom headers
request['Authorization'] = signed_auth_header
request['Date'] = date

# Send request
response = http.request(request)

# Handle response
if response.is_a?(Net::HTTPSuccess)
  puts "Success: #{response.body}"
else
  puts "Error: #{response.code} #{response.message}"
end
```

```python
import requests

# Prepare URL and headers
url = "https://api-staging.even.biz/api/v1/distribution/artists"
endpoint = "/api/v1/distribution/artists"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='get', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Send GET request
response = requests.get(url, headers=headers)

# Handle response
if response.status_code == 200:
    print(f"Success: {response.text}")
else:
    print(f"Error: {response.status_code} {response.reason}")
```

```javascript
const axios = require('axios');

// Set the endpoint and HTTP method
const url = 'https://api-staging.even.biz/api/v1/distribution/artists';
const method = 'GET';
const endpoint = '/api/v1/distribution/artists';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Send the request using axios
axios.get(url, {
    headers: {
        'Content-Type': 'application/json',
        'Authorization': authorization,
        'Date': date
    }
})
    .then(response => {
        console.log('Success:', response.data);
    })
    .catch(error => {
        if (error.response) {
            console.error(`Error: ${error.response.status} ${error.response.statusText}`);
        } else {
            console.error(error.message);
        }
    });
```

> The above command returns JSON structured like this:

```json
{
  "data": [
    {
      "id": "1",
      "type": "artist",
      "attributes": {
        "id": 1,
        "name": "Jane Doe",
        "slug": "jane-doe",
        "hidden": true
      },
      "relationships": {}
    },
    {
      "id": "2",
      "type": "artist",
      "attributes": {
        "id": 2,
        "name": "John Doe",
        "slug": "john-doe",
        "hidden": true
      },
      "relationships": {}
    }
  ],
  "meta": {
    "page_url": "/api/v1/distribution/artists?page=1",
    "prev_url": "/api/v1/distribution/artists?page=",
    "next_url": "/api/v1/distribution/artists?page=",
    "page": 1,
    "prev": null,
    "next": null,
    "last": 1,
    "pages": 1,
    "in": 2,
    "count": 2
  }
}

```

This endpoint retrieves all artists.

### HTTP Request

- **Production**: `GET https://api.even.biz/api/v1/distribution/artists`
- **Stage**: `GET https://api-staging.even.biz/api/v1/distribution/artists`


### Query Parameters

Parameter | Default | Description
--------- |---------| -----------
page | null    | If set to a integer value, the result will be paginated.

## Get Specific Artist

```ruby
require 'net/http'
require 'json'

uri = URI("https://api-staging.even.biz/api/v1/distribution/artists/1")

# Prepare HTTP request
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Get.new(uri.path, 'Content-Type' => 'application/json')

# Generate required headers
date = Time.now.utc.httpdate
signed_auth_header = sign_headers(method: :get, endpoint: uri.path, date: date)

# Set custom headers
request['Authorization'] = signed_auth_header
request['Date'] = date

# Send request
response = http.request(request)

# Handle response
if response.is_a?(Net::HTTPSuccess)
  puts "Success: #{response.body}"
else
  puts "Error: #{response.code} #{response.message}"
end
```

```python
import requests

# Prepare URL and headers
url = "https://api-staging.even.biz/api/v1/distribution/artists/1"
endpoint = "/api/v1/distribution/artists/1"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='get', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Send GET request
response = requests.get(url, headers=headers)

# Handle response
if response.status_code == 200:
    print(f"Success: {response.text}")
else:
    print(f"Error: {response.status_code} {response.reason}")
```

```javascript
const axios = require('axios');

// Set the endpoint and HTTP method
const url = 'https://api-staging.even.biz/api/v1/distribution/artists/1';
const method = 'GET';
const endpoint = '/api/v1/distribution/artists/1';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Send the request using axios
axios.get(url, {
    headers: {
        'Content-Type': 'application/json',
        'Authorization': authorization,
        'Date': date
    }
})
    .then(response => {
        console.log('Success:', response.data);
    })
    .catch(error => {
        if (error.response) {
            console.error(`Error: ${error.response.status} ${error.response.statusText}`);
        } else {
            console.error(error.message);
        }
    });
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "id": "1",
    "type": "artist",
    "attributes": {
      "id": 1,
      "name": "Jane Doe",
      "slug": "jane-doe",
      "username": "artist-53f58fa7-4cc4-4925-b261-0ecd16e85c0b",
      "external_link": null,
      "bio": null,
      "about": null,
      "links": null,
      "published": true,
      "country": null,
      "city": null,
      "private": true,
      "hidden": true,
      "verified": false,
      "onboarded": false,
      "created_at": "2024-10-07T05:59:46.957Z",
      "updated_at": "2024-10-07T05:59:46.957Z",
      "upload_status": "approved"
    },
    "relationships": {
      "releases": {
        "data": []
      }
    }
  },
  "included": []
}
```

This endpoint retrieves a specific artist.

### HTTP Request

- **Production**: `GET https://api.even.biz/api/v1/distribution/artists/<ID>`
- **Stage**: `GET https://api-staging.even.biz/api/v1/distribution/artists/<ID>`

### URL Parameters

Parameter | Description | Required
--------- | ----------- | ----------- 
ID | The ID of the artist to retrieve| Yes

## Create Bulk Artists

```ruby
uri = URI("https://api-staging.even.biz/api/v1/distribution/artists")

# Prepare HTTP request
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true 

request = Net::HTTP::Post.new(uri.path, 'Content-Type' => 'application/json')

# Generate required headers
date = Time.now.utc.httpdate
signed_auth_header = sign_headers(method: :post, endpoint: uri.path, date: date)

# Set custom headers
request['Authorization'] = signed_auth_header
request['Date'] = date

body = {
  artists: [
    {
      name: "Snarky Puppy",
      email: "Snarky@puppy.com",
      links: {
        "spotify": "https://open.spotify.com/artist/7ENzCHnmJUr20nUjoZ0zZ1?si=KpNjtD0HTsiShOTM2ZuFdA",
        "instagram": "https://www.instagram.com/snarkypuppy",
        "soundcloud": "https://soundcloud.com/snarkypuppy"
      },
    },
    {
      name: "Unknown Artist",
      email: "unknown@artist.com",
      links: {
        "spotify": "https://open.spotify.com/artist/unknown",
        "youtube": "https://www.instagram.com/unknown"
      }
    }
  ]
}
request.body = body.to_json

# Send request
response = http.request(request)

# Handle response
if response.is_a?(Net::HTTPSuccess)
  puts "Success: #{response.body}"
else
  puts "Error: #{response.code} #{response.message}"
end
```

```python
# Prepare URL and headers
url = "https://api-staging.even.biz/api/v1/distribution/artists"
endpoint = "/api/v1/distribution/artists"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='post', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Prepare request body
body = {
    "artists": [
        {
            "name": "Snarky Puppy",
            "email": "Snarky@puppy.com",
            "links": {
                "spotify": "https://open.spotify.com/artist/7ENzCHnmJUr20nUjoZ0zZ1?si=KpNjtD0HTsiShOTM2ZuFdA",
                "instagram": "https://www.instagram.com/snarkypuppy",
                "soundcloud": "https://soundcloud.com/snarkypuppy"
            },
        },
        {
            "name": "Unknown Artist",
            "email": "unknown@artist.com",
            "links": {
                "spotify": "https://open.spotify.com/artist/unknown",
                "youtube": "https://www.instagram.com/unknown"
            }
        }
    ]
}

# Send POST request
response = requests.post(url, headers=headers, json=body)

# Handle response
if response.status_code == 200:
    print(f"Success: {response.text}")
else:
    print(f"Error: {response.status_code} {response.reason}")
```

```javascript
const axios = require('axios');

// Set the endpoint and HTTP method
const url = 'http://api-staging.even.biz/api/v1/distribution/artists';
const method = 'POST';
const endpoint = '/api/v1/distribution/artists';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Prepare the request body
const body = {
    artists: [
        {
            name: 'Snarky Puppy',
            email: 'Snarky@puppy.com',
            links: {
                spotify: 'https://open.spotify.com/artist/7ENzCHnmJUr20nUjoZ0zZ1?si=KpNjtD0HTsiShOTM2ZuFdA',
                instagram: 'https://www.instagram.com/snarkypuppy'
            }
        },
        {
            name: 'Unknow Artist',
            email: 'unknow@artist.com',
            links: {
                spotify: 'https://open.spotify.com/artist/unknow',
                youtube: 'https://www.instagram.com/unknow'
            }
        }
    ]
};

// Send the request using axios
axios.post(url, body, {
    headers: {
        'Content-Type': 'application/json',
        'Authorization': authorization,
        'Date': date
    }
})
    .then(response => {
        console.log('Success:', response.data);
    })
    .catch(error => {
        if (error.response) {
            console.error(`Error: ${error.response.status} ${error.response.statusText}`);
        } else {
            console.error(error.message);
        }
    });

```

> The above command returns JSON structured like this:

```json
{
  "success": {
    "created": [
      {
        "id": 27,
        "name": "Snarky Puppy",
        "slug": "snarky-puppy",
        "bio": "An acclaimed fusion-influenced jam band, Snarky Puppy have built a loyal following with their adventurous blend of jazz, rock, and funk. Led by bassist <a href=\"spotify:artist:0YzrofLyUlORKLkH8AAMm9\">Michael League</a>, the Texas group emerged in the mid-2000s and garnered buzz with albums like 2006's The Only Constant, 2010's Tell Your Friends, and 2013's Family Dinner, Vol. 1, which took home the Grammy for Best R&B Performance for their cover of the <a href=\"spotify:artist:2O8VlquQPITO4tT3SWs95W\">Brenda Russell</a> song \"Something\" featuring singer <a href=\"spotify:artist:0uNEy4544VZq2KOl7BsLuo\">Lalah Hathaway</a>. More accolades followed, including further Grammy Awards for 2015's Sylva and 2016's Culcha Vulcha, the latter of which also topped Billboard's Jazz Albums chart. The band again reached the Top Ten of the jazz charts with 2019's Immigrance and picked up their fourth Grammy Award for 2020's Live at the Royal Albert Hall. In 2022, they paid tribute to their Dallas-area roots with Empire Central.\n\nFormed in 2004 in Denton, Texas (where the band members were studying jazz at the University of North Texas), Snarky Puppy features a wide-ranging assemblage of musicians known affectionately as \"the Fam,\" centered around bassist and leader <a href=\"spotify:artist:0YzrofLyUlORKLkH8AAMm9\">Michael League</a>. They debuted with the concert album Live at Uncommon Ground in 2005. Over the next few years, the band worked to build their fan base, touring heavily and issuing a handful of well-received albums including 2006's The Only Constant, 2007's The World Is Getting Smaller, 2008's Bring Us the Bright, 2010's Tell Your Friends, and 2012's groundUP. In 2013, Snarky Puppy released the album Family Dinner, Vol. 1, which included a cover of <a href=\"spotify:artist:2O8VlquQPITO4tT3SWs95W\">Brenda Russell</a>'s \"Something\" featuring singer <a href=\"spotify:artist:0uNEy4544VZq2KOl7BsLuo\">Lalah Hathaway</a>. The song proved a hit and propelled the album up various digital download charts. In 2014, \"Something\" earned Snarky Puppy and <a href=\"spotify:artist:0uNEy4544VZq2KOl7BsLuo\">Hathaway</a> the Grammy Award for Best R&B Performance. Also that year, they issued the concert album We Like It Here on <a href=\"spotify:search:label%3A%22Ropeadope%22\">Ropeadope</a>.\n\nSnarky Puppy next signed to <a href=\"spotify:search:label%3A%22Impulse%21%22\">Impulse!</a> and released Sylva, a collaboration with the Netherlands-based <a href=\"spotify:artist:7JYdpWAsiqzrmMB3qxkEbI\">Metropole Orkest</a>, in 2015. The album was greeted enthusiastically by the international press and won the band their second Grammy, this time for Best Contemporary Instrumental Album, at the 2016 Grammy Awards. They followed it with two live documents. The first was World Tour, a 32-disc box featuring their 16 favorite concerts; the deluxe package was sold exclusively through their website. In early 2016, the group also issued the audio-video set Family Dinner, Vol. 2, a documentary follow-up to the first Family Dinner that was recorded the previous year. Vol. 2 showcased the band during a concert (and included guests <a href=\"spotify:artist:0si9BxvM2C33fAIkr1pgUc\">Charlie Hunter</a>, <a href=\"spotify:artist:1DiaZsjdOzFCdk7Dw9KIs0\">Susana Baca</a>, <a href=\"spotify:artist:0VVnWF3KNaa5O7ESohKhAx\">Salif Keita</a>, and <a href=\"spotify:artist:59zdhVoWxSoHMc74n098Re\">David Crosby</a>), in interviews, and in backstage sequences.\n\nIn April 2016, Snarky Puppy struck again with Culcha Vulcha, their 11th studio album and first solely studio-based production in eight years. It topped the jazz album charts the week of its release and took home the prize for Best Contemporary Instrumental Album at the 2017 Grammy Awards. In 2019, they issued the <a href=\"spotify:artist:0YzrofLyUlORKLkH8AAMm9\">League</a>-produced Immigrance, which included the single \"Xavi.\" It debuted at number two on Billboard's Jazz Albums charts. While on tour for the album, they recorded their November 14, 2019 performance in London. Released the following year as Live at the Royal Albert Hall, the record took home the Grammy Award for Best Contemporary Instrumental Album.\n\nWith 2022's Empire Central, Snarky Puppy paid tribute to their hometown of Dallas. Recorded over eight nights at the city's Deep Ellum Art Company with an audience in attendance, the album found them drawing upon their jazz and funk roots going back to their time as students at the University of North Texas. The set also marked one of the final recorded appearances of '80s funk icon <a href=\"spotify:artist:2g8J4iI5qyYFMVHcpMcSsS\">Bernard Wright</a> (a major influence on the group), who died soon after finishing the sessions. Empire Central was named Best Contemporary Instrumental Album at the 65th Annual Grammy Awards in 2023. ~ Matt Collar, Rovi",
        "about": null,
        "links": {
          "spotify": "https://open.spotify.com/artist/7ENzCHnmJUr20nUjoZ0zZ1?si=KpNjtD0HTsiShOTM2ZuFdA",
          "instagram": "https://www.instagram.com/snarkypuppy", 
          "soundcloud": "https://soundcloud.com/snarkypuppy"
        },
        "city": "Brooklyn",
        "private": true,
        "hidden": true,
        "user_id": null,
        "created_at": "2024-10-21T20:01:03.822Z",
        "updated_at": "2024-10-21T20:01:03.822Z",
        "verified": false,
        "onboarded": false,
        "username": "artist-ca6a1e27-3c90-429d-91ad-2d70e0cfc906",
        "external_link": null,
        "created_by": null,
        "info": {
          "email": "Snarky@puppy.com",
          "image_url": "https://i.scdn.co/image/ab6761610000e5ebae9e6dd153b94058ebf99f6e"
        },
        "published": true,
        "record_label_id": 1,
        "country": "us",
        "upload_status": "approved",
        "notes": null,
        "party_identifier_id": 1,
        "referred_by_type": null,
        "referred_by_id": null,
        "referral_code": "ceb782e951093f556229"
      }
    ]
  },
  "errors": {
    "failed": [
      {
        "artist": {
          "email": "unknown@artist.com",
          "name": "Unknown Artist",
          "links": {
            "spotify": "https://open.spotify.com/artist/unknown",
            "youtube": "https://music.youtube.com/unknown"
          }
        },
        "errors": "Unable to found artist"
      }
    ]
  }
}

```

This endpoint create a bulk artists.

### HTTP Request

- **Production**: `POST https://api.even.biz/api/v1/distribution/artists`
- **Stage**: `POST https://api-staging.even.biz/api/v1/distribution/artists`

### Body Parameters

### Body Parameters

Parameter | Description | Required
--------- | ----------- | --------
artists | An array of objects containing information about artists. | Yes
artists[].name | The name of the artist. | Yes
artists[].email | The email address of the artist. | Yes
artists[].links | An object containing the artist's social media links. | Yes
artists[].links.spotify | The Spotify link of the artist. | Yes
artists[].links.instagram | The Instagram link of the artist (if available). | No
artists[].links.soundcloud | The SoundCloud link of the artist (if available). | No
artists[].links.youtube | The YouTube link of the artist (if available). | No


# Data Delivery Method

## AWS

EVEN offers this methods for data delivery using AWS.

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help.

## SFTP

EVEN offers this methods for data delivery using SFTP.

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help.