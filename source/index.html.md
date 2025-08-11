---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - ruby
  - python
  - javascript
  - php

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

```php
function sign_headers($method, $endpoint, $date) {
    $access_id = '<ACCESS_ID>';
    $secret_key = '<SECRET_KEY>';

    $method = strtoupper($method);
    $content_type = "application/json";

    $canonical_string = "$method,$content_type,,$endpoint,$date";
    $signature = base64_encode(hash_hmac('sha256', $canonical_string, $secret_key, true));

    return "APIAuth-HMAC-SHA256 $access_id:$signature";
}

$date = gmdate('D, d M Y H:i:s T');
$endpoint = "/api/v1/distribution/artists";

$signed_auth_header = sign_headers('get', $endpoint, $date);
echo $signed_auth_header;
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'GET';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists/1";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'GET';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
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

- **Production**: `GET https://api.even.biz/api/v1/distribution/artists/:artist_id`
- **Stage**: `GET https://api-staging.even.biz/api/v1/distribution/artists/:artist_id`

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
            "name": "Snarky Puppy.",
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'POST';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Request body
$body = json_encode([
    "artists" => [
        [
            "name" => "Snarky Puppy",
            "email" => "Snarky@puppy.com",
            "links" => [
                "spotify" => "https://open.spotify.com/artist/7ENzCHnmJUr20nUjoZ0zZ1?si=KpNjtD0HTsiShOTM2ZuFdA",
                "instagram" => "https://www.instagram.com/snarkypuppy",
                "soundcloud" => "https://soundcloud.com/snarkypuppy"
            ],
        ],
        [
            "name" => "Unknown Artist",
            "email" => "unknown@artist.com",
            "links" => [
                "spotify" => "https://open.spotify.com/artist/unknown",
                "youtube" => "https://www.instagram.com/unknown"
            ]
        ]
    ]
]);

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
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

This endpoint creates artists in bulk.

### HTTP Request

- **Production**: `POST https://api.even.biz/api/v1/distribution/artists`
- **Stage**: `POST https://api-staging.even.biz/api/v1/distribution/artists`

### Body Parameters

Parameter | Description | Required
--------- | ----------- | --------
artists | An array of objects containing information about artists. | Yes
artists[].name | The name of the artist. | Yes
artists[].email | The email address of the artist. | No
artists[].links | An object containing the artist's social media links. | Yes
artists[].links.spotify | The Spotify link of the artist. | Yes
artists[].links.instagram | The Instagram link of the artist (if available). | No
artists[].links.soundcloud | The SoundCloud link of the artist (if available). | No
artists[].links.youtube | The YouTube link of the artist (if available). | No
artists[].complete | Marking an Artist as complete will create an Invitation and send an email to said Artist asking them to join EVEN. You can mark an Artist as complete later via an update with `PUT` if there's still data that needs to be filled in after creating an Artist. | No

## Import Artists with CSV File

This endpoint receives a CSV file that will import all artists similar to the Bulk Create Artist Endpoint.

**Note**: This will only work with CSV files. Other files will return an error.

```ruby
uri = URI("https://api-staging.even.biz/api/v1/distribution/artists/import")

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
  artists: {
    file: "your-csv-file.csv"
  }
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
url = "https://api-staging.even.biz/api/v1/distribution/artists/import"
endpoint = "/api/v1/distribution/artists/import"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='post', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Prepare request body
body = {
    "artists": {
        "file": "your-csv-file.csv"
    }
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
const url = 'http://api-staging.even.biz/api/v1/distribution/artists/import';
const method = 'POST';
const endpoint = '/api/v1/distribution/artists/import';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Prepare the request body
const body = {
    artists: {
      file: "your-csv-file.csv"
    }
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists/import";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'POST';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Request body
$body = json_encode([
    "artists" => {
      "file" => "your-csv-file.csv"
    }
]);

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
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

### HTTP Request

- **Production**: `POST https://api.even.biz/api/v1/distribution/artists/import`
- **Stage**: `POST https://api-staging.even.biz/api/v1/distribution/artists/import`

### Body Parameters

Parameter | Description | Required
--------- | ----------- | --------
artists | outermost parameter required to parse the file with our API. | Yes
file | CSV File containing the artists to be imported. | Yes

> You will find an example CSV file below.

```csv
name,email,spotify_url,youtube_url,instagram_url
Snarky Puppy,Snarky@puppy.com,https://open.spotify.com/artist/7142321893?si=8888,https://music.youtube.com/Snarky,https://www.instagram.com/Snarky/
```

## Claim Artist

This endpoint receives an `artist_id` to claim the Artist associated with the specified email fields.

It will then create an Invitation and send an email to the Artist, inviting them to join EVEN.


**Note**: Only one Artist can be claimed per HTTP Request.


```ruby
uri = URI("https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/claim")

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
  artist: {
    email: "artist@music.com"
  }
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
url = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/claim"
endpoint = "/api/v1/distribution/artists/:artist_id/claim"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='post', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Prepare request body
body = {
    "artist": {
      "name": "updated name",
      "country": "US",
      "email": "artist@music.com"
    }
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
const url = 'http://api-staging.even.biz/api/v1/distribution/artists/:artist_id/claim';
const method = 'POST';
const endpoint = '/api/v1/distribution/artists/:artist_id/claim';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Prepare the request body
const body = {
  artist: {
    email: "artist@music.com"
  }
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/claim";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'POST';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Request body
$body = json_encode([
    "artist" => {
        "email" => "artist@email.com"
    }
]);

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "id": "1",
    "type": "artist",
    "attributes": {
      "id": 1,
      "name": "updated name",
      "slug": "updated-name",
      "username": "artist-53f58fa7-4cc4-4925-b261-0ecd16e85c0b",
      "external_link": null,
      "bio": "some cool bio",
      "about": null,
      "links": null,
      "published": true,
      "country": "US",
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

### HTTP Request

- **Production**: `POST https://api.even.biz/api/v1/distribution/artists/:artist_id/claim`
- **Stage**: `POST https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/claim`

### Body Parameters

Parameter | Description | Required
--------- | ----------- | --------
artist | Artist object containing the fields to be edited. | Yes
artists[].email | The email address of the artist. | Yes

## Update Artist

This endpoint receives an `artist_id` and updates the Artist with the specified fields.

**Note**: Only one Artist can be edited per HTTP Request.


```ruby
uri = URI("https://api-staging.even.biz/api/v1/distribution/artists/:artist_id")

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
  artist: {
    name: "updated name",
    country: "US",
    bio: "some cool bio"
  }
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
url = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id"
endpoint = "/api/v1/distribution/artists/:artist_id"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='post', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Prepare request body
body = {
    "artist": {
      "name": "updated name",
      "country": "US",
      "bio": "some cool bio"
    }
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
const url = 'http://api-staging.even.biz/api/v1/distribution/artists/:artist_id';
const method = 'POST';
const endpoint = '/api/v1/distribution/artists/:artist_id';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Prepare the request body
const body = {
  artist: {
    name: "updated name",
    country: "US",
    bio: "some cool bio"
  }
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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'POST';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Request body
$body = json_encode([
    "artist" => {
        "name" => "updated name",
        "country" => "US",
        "bio" => "some cool bio"
    }
]);

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "id": "1",
    "type": "artist",
    "attributes": {
      "id": 1,
      "name": "updated name",
      "slug": "updated-name",
      "username": "artist-53f58fa7-4cc4-4925-b261-0ecd16e85c0b",
      "external_link": null,
      "bio": "some cool bio",
      "about": null,
      "links": null,
      "published": true,
      "country": "US",
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

### HTTP Request

- **Production**: `PUT https://api.even.biz/api/v1/distribution/artists/:artist_id`
- **Stage**: `PUT https://api-staging.even.biz/api/v1/distribution/artists/:artist_id`

### Body Parameters

Parameter | Description | Required
--------- | ----------- | --------
artist | Artist object containing the fields to be edited. | Yes
artists[].name | The name of the artist. | No
artists[].email | The email address of the artist. | No
artists[].links | An object containing the artist's social media links. | No
artists[].links.spotify | The Spotify link of the artist. | No
artists[].links.instagram | The Instagram link of the artist (if available). | No
artists[].links.soundcloud | The SoundCloud link of the artist (if available). | No
artists[].links.youtube | The YouTube link of the artist (if available). | No
artists[].username | The EVEN username of the artist. | No
artists[].external_link | Any additional profile link (singluar) for the Artist. | No
artists[].bio | Artist Bio | No
artists[].about | Artist About page | No
artists[].country | Artist's country | No
artists[].city | Artist's city | No
artists[].complete | Marking an Artist as complete will create an Invitation and send an email to said Artist asking them to join EVEN. You can mark an Artist as complete later via an update with `PUT` if there's still data that needs to be filled in after creating an Artist. | No

## Remove Artist

This endpoint receives an `:id` and removes the Artist from the Partner's Catalog.

**Note**: This endpoint will **not** remove the Artist from our database. It will only remove the associations between the Partner and the Artist. For full data deletion, please contact us.

You can verify the request work from the HTTP status `200` response, and by using the `api/v1/distribution/artists` or `api/v1/distribution/artists/:id` endpoints to make sure the artist is no longer available for that Partner.

### HTTP Request

- **Production**: `POST https://api.even.biz/api/v1/distribution/artists/:artist_id/remove`
- **Stage**: `POST https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/remove`

### URL Parameters

Parameter | Description | Required
--------- | ----------- | -----------
ID | The ID of the artist to retrieve| Yes

```ruby
uri = URI("https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/remove")

# Prepare HTTP request.
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Post.new(uri.path, 'Content-Type' => 'application/json')

# Generate required headers
date = Time.now.utc.httpdate
signed_auth_header = sign_headers(method: :post, endpoint: uri.path, date: date)

# Set custom headers
request['Authorization'] = signed_auth_header
request['Date'] = date

body = {}
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
url = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/remove"
endpoint = "/api/v1/distribution/artists/:artist_id/remove"
date = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
signed_auth_header = sign_headers(method='post', endpoint=endpoint, date=date)

headers = {
    'Content-Type': 'application/json',
    'Authorization': signed_auth_header,
    'Date': date
}

# Prepare request body
body = {}

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
const url = 'http://api-staging.even.biz/api/v1/distribution/artists/:artist_id/remove';
const method = 'POST';
const endpoint = '/api/v1/distribution/artists/:artist_id/remove';

// Generate date and authorization header
const date = new Date().toUTCString();
const authorization = signHeaders(method, endpoint, date);

// Prepare the request body
const body = {};

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

```php
$uri = "https://api-staging.even.biz/api/v1/distribution/artists/:artist_id/remove";

// Prepare HTTP request
$ch = curl_init($uri);

$method = 'POST';
$endpoint = parse_url($uri, PHP_URL_PATH);
$date = gmdate('D, d M Y H:i:s T');

// Generate required headers
$signed_auth_header = sign_headers($method, $endpoint, $date);

// Set custom headers
$headers = [
    'Content-Type: application/json',
    'Authorization: ' . $signed_auth_header,
    'Date: ' . $date,
];

// Request body
$body = json_encode([]);

// Set cURL options
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
curl_setopt($ch, CURLOPT_POSTFIELDS, $body);

// Send request and get response
$response = curl_exec($ch);
$httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Handle response
if ($httpcode >= 200 && $httpcode < 300) {
    echo "Success: " . $response;
} else {
    echo "Error: " . $httpcode . " " . curl_error($ch);
}

curl_close($ch);
```

> The above command returns an HTTP status `200`.

# Data Delivery Method

## AWS

EVEN offers these methods for data delivery using AWS.

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help.

### How Data Delivery works with AWS

After getting in contact with support and taking a look at your specific case, we'll generate a set of AWS S3 Bucket credentials. 

You'll be able to upload your DDEX ERN-compliant message (we support version 3.8 and 4.1).

### File Structure

| **Parameter**        | **Description**                                                                                                                                              | **Required** |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
| **Batch Id**         | A batch Id represents the date and time of its creation using the format `YYYYMMDDhhmmssnnn`. The batch Id is unique and used to name the batch directory.   | Yes          |
| **Root Directory**   | The root release directory contains two items: the release notification XML file and a "resources" directory.                                                | Yes          |
| **Release Folder**   | Every release within the batch must have a separate folder using the ReleaseId of the release as its name. Conventionally, this is the album UPC or EAN.     | Yes          |
| **Resources Folder** | The "resources" directory contains all the track MP3, FLAC or WAV files specified within the release and an image resource file for the release's album art. | Yes          |


```shell
20210131310922000/                         // BatchId (YYYYMMDDhhmmssnnn)
    A12345678910111213/                    // ReleaseId (UPC|EAN)
        A12345678910111213.xml             // ReleaseId.xml (NewReleaseMessage)
        resources/                         // Resources folder
            A12345678910111213_T1_001.mp3  // TechnicalSoundRecordingDetails.{mp3,wav}
            A12345678910111213_T2.jpg      // TechnicalSoundRecordingDetails.{jpg,png}
    A12345678910111214/                    // Another ReleaseId (UPC|EAN)
        A12345678910111214.xml             // ReleaseId.xml (NewReleaseMessage)
        resources/                         // Resources folder
            A12345678910111214_T1_001.mp4  // TechnicalSoundRecordingDetails.{mp4}
            A12345678910111214_T2_002.mp3  // TechnicalSoundRecordingDetails.{mp3}

```

### What happens after I upload my ERN XML File?

Every 24 hours we'll download the newly-uploaded XML files from our partner buckets to be processed. While processing we'll:

* Create the Release on EVEN.
* Create the Release's tracks on EVEN.
* Create the associated Artists on EVEN.
* Create the associated images and attach them to the release on EVEN.
* If the Artists inside the XML already exist, we'll notify them a Release was created for them and add them to the Partner's Artist Catalog if they're not there yet.
* Any existing Releases will be updated.

We'll provide an example XML document with all the basic information your ERN needs to have.

### What happens when an Artist I created via the API is included in one of my ERN?

When processing a partner's ERN we'll look for an Artist with the same NAME inside the Partner's Artist Catalog. This way they can be successfully linked together instead of creating repeat Artists.

### What happens when I create Artists via an ERN messsage but not the API?

Artists inside the ERN ReleasesList will be created with the following fields:

* `name`: from the ERN
* `upload_status`: approved by default
* `published`: true by default
* `username`: autogenerated

Alternatively, a partner can manually assign additional fields (like email) and generate invitations for the Artists from Bakcstage, doing a `PATCH` to our API with the additional fields, or just letting support know about the changes.

### What happens if I created Artists via an ERN message without an EMAIL, but now I want my Artists to join EVEN?

There are two useful endpoints to make this work:

Firstly, we want to get the Distributor's Artists:

* `api/v1/distribution/artists`

This will return all the Artists in a Distributor's catalog. After acquiring the Artist in question, we want to use the second endpoint:

* `api/v1/distribution/artists/:artist_id`

To assign an EMAIL to the artist, and mark them as complete. Signaling our Backend to send anotification informing the Artist in question that they should log in to EVEN.

NOTE: This is only for cases where the Artist email was not provided in the initial ERN message.

For further questions and support:

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help.

## SFTP

EVEN offers these methods for data delivery using SFTP.

Contact <a href= "mailto: support@even.biz"> support@even.biz</a> for help.
