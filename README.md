# wsc-api-examples-ruby

## Wowza Streaming Cloud REST API - Example Application for the Live Stream Workflow Written in Ruby

This API example application demonstrates the live stream workflow using the `/live_streams` endpoint of the Wowza Streaming Cloud™ service REST API version 1.

If you just want to know how to use the Wowza Streaming Cloud API using the Ruby Net::HTTP library, [look at this section](#how_to_ruby_net_http).

## Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Run the application](#run)
- [How to use the example application](#how_to_use)
- [How to use the Wowza Streaming Cloud REST API with the Ruby Net::HTTP library](#how_to_ruby_net_http)
- [References](#references)
- [Contact](#contact)
- [License](#license)

<a name="prerequisites"></a>
## Prerequisites

To install the example application make sure you have [Ruby](https://www.ruby-lang.org/en/documentation/installation/), [RubyGems](https://rubygems.org/), and [Bundler](http://bundler.io/) ready to use on your system.

<a name="installation"></a>
## Installation

1. In Terminal, make sure you're in your application directory:

```bash
$ cd /wsc-api-example-ruby
```

2. Run bundle install:

```bash
$ bundle install
```

You're done!

<a name="configuration"></a>
## Configuration

To use the application, you need a valid [Wowza Streaming Cloud account](https://cloud.wowza.com/) and access to the API.

### Put the 'API Key' and the 'API Access Key' into the configuration file

```yml
# config/keys.yml
api_key: "- PASTE YOUR API KEY HERE -"
api_access_key: "- PASTE YOUR API ACCESS KEY HERE -"
```

### (Optional) Change the hostname of the Wowza Streaming Cloud environment

You have two options. To use our sandbox environment and be safe, use this hostname:

```yml
# config/settings.yml
api_base_url: "https://api-sandbox.cloud.wowza.com"
...
```

> **Note:** This is the default. If you don't change anything, you'll use the free sandbox environment.

If you know what to do and want to use your live account (**and accrue charges**), use this hostname:

```yml
# config/settings.yml
api_base_url: "https://api.cloud.wowza.com"
...
```

### (Optional) Enable debug output

If you want to see header response data and response codes for each API call, enable it in the settings:

```yml
# config/settings.yml
...
debug: true
```

This is disabled by default.

<a name="run"></a>
## Run the application

In Terminal, make sure you're in your application directory and execute the Ruby file:

```bash
$ ./live_stream_api_example.rb
```

> **Note:** Make sure the file is executable: ```$ chmod +x live_stream_api_example.rb```

That's all.

<a name="how_to_use"></a>
## How to use the example application

After launching the application, you'll see the main menu with several action items. Just enter a number and press Return to be guided through the application. It's easy.

*The screen after launching the application:*
```
"Main Menu"

1. Show the number of live streams in your account
2. List all live streams of your account
3. Create a live stream with pre-configured settings         => data/live_stream/encoder_types/*.json
4. Show the details of an existing live stream
5. Update a live stream with pre-configured settings         => data/live_stream/update_example.json
6. Start a live stream                                       => only for live streams with the 'stopped' state
7. Reset a live stream                                       => only for live streams with the 'started' state
8. Stop a live stream                                        => only for live streams with the 'started' state
9. Show the current state of a live stream
10. Show the thumbnail URL of a live stream                  => only for live streams with the 'started' state
11. Delete a live stream
12. Run the pre-configured live stream workflow
13. Quit :(

Enter the number for the action you want to execute.
```

**Actions 1 through 11** use single API calls to your account's data. This should give you an overview what you can do with the `/live_streams` endpoint on Wowza Streaming Cloud.

**Action 12** ('Run the pre-configured live stream workflow') is meant to give you an idea of the complete live stream workflow. It uses the same code as actions 1-11. It's separated into the following steps:

1. Create a live stream
2. Start the live stream
3. Poll the status of the live stream (to know when the live stream in your requested location is ready to use)
4. Poll the status of the player that was created with your live stream (to know when the player and the hosted page are successfully provisioned and ready to use)
5. Receive the hosted page URL (where you can watch the stream)
6. Stop the live stream
7. Delete the live stream

You will be guided through each step.

> **Note:** The live stream settings used by default in this example application are based on the **Other RTSP Pull** encoder setting described in our user interface on [https://cloud.wowza.com/](https://cloud.wowza.com/). The data, along with lots of other examples of different live stream configurations, are stored in JSON files in the [data/live_stream/](https://github.com/WowzaMediaSystems/wsc-api-examples-ruby/tree/main/data/live_stream) directory in this repository:

```json
# data/live_stream/encoder_types/other_rtsp_pull.json
{
  "live_stream": {
    "name": "My Awesome live stream",
    "transcoder_type": "transcoded",
    "billing_mode": "pay_as_you_go",
    "broadcast_location": "us_west_california",
    "encoder": "other_rtsp",
    "delivery_method": "pull",
    "source_url": "1.2.3.4/axis-media/media.amp",
    "aspect_ratio_width": 1920,
    "aspect_ratio_height": 1080
  }
}
```

<a name="how_to_ruby_net_http"></a>
## How to use the Wowza Streaming Cloud REST API with the Ruby Net::HTTP library

### Get a list of all live streams of your account (GET)

```ruby
require 'net/http'

# build the URI to access the /live_streams endpoint
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/")

# initialize Net::HTTP using SSL
Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|

  # build the HTTP request
  request = Net::HTTP::Get.new uri.path
  # add authentication keys to the request header
  request['wsc-api-key'] = "API_KEY"
  request['wsc-access-key'] = "API_ACCESS_KEY"
  # add the right Content-Type to the header (always required when pushing data)
  request["Content-Type"] = "application/json"

  response = http.request(request)

  # get the response as JSON object
  puts JSON.parse(response.body)
  # get the response code
  puts response.code
  # loop through response headers:
  response.each_header do |key, value|
    puts "#{key} => #{value}"
  end

end

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' https://api.cloud.wowza.com/api/v1/live_streams/
```

Example Response:
```json
# 200 OK
{
    "live_streams": [
        {
              "id": "kdgbsvth",
            "name": "My Awesome Live Stream",
             "...": "..."
        },
        {
              "id": "ftklwjvz",
            "name": "My Second Live Stream",
             "...": "..."
        }
    ]
}
```

<a name="how_to_ruby_net_http_more_examples"></a>
## Additional examples

> **Note:** The following sections contain examples that show only the differences from the preceding example.

### Create a live stream (POST)

> **Note:** Check [/data/live_stream/encoder_types/*.json](https://github.com/WowzaMediaSystems/wsc-api-examples-ruby/tree/main/data/live_stream/encoder_types) for real-world JSON examples.

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/")
...
  request = Net::HTTP::Post.new uri.path
  # add the body to the request and parse the hash to JSON (or load it from a JSON file)
  request.body = {
    "live_stream": {
      LIVE_STREAM_SETTINGS
    }
  }.to_json
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -H 'Content-Type: application/json' -X POST -d '{
#   "live_stream": {
#     LIVE_STREAM_SETTINGS
#   }
# }' https://api.cloud.wowza.com/api/v1/live_streams/
```

Example Response:
```json
# 201 CREATED
{
    "live_stream": {
          "id": "kdgbsvth",
        "name": "My Awesome Live Stream",
         "...": "..."
    }
}
```

### Details of a live stream (GET)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/")
...
  request = Net::HTTP::Get.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
                                   "id": "kdgbsvth",
                                 "name": "My Awesome Live Stream",
                      "transcoder_type": "transcoded",
                         "billing_mode": "pay_as_you_go",
                   "broadcast_location": "us_west_california",
                            "recording": false,
                  "closed_caption_type": "none",
                              "encoder": "other_rtsp",
                      "delivery_method": "pull",
                    "use_stream_source": false,
                   "aspect_ratio_width": 1920,
                  "aspect_ratio_height": 1080,
        "source_connection_information": { "source_url": "rtsp://..." },
                            "player_id": "g7kpkj1n",
                    "player_responsive": false,
                         "player_width": 640,
                     "player_countdown": false,
                    "player_embed_code": "in_progress",
              "player_hds_playback_url": "http://...",
              "player_hls_playback_url": "http://...",
                          "hosted_page": true,
                    "hosted_page_title": "My Awesome Event",
                      "hosted_page_url": "https://...",
            "hosted_page_sharing_icons": true,
                           "created_at": "2016-02-29T17:59:25.251Z",
                           "updated_at": "2016-02-29T17:59:25.604Z"
    }
}
```

### Update a live stream (PATCH)

> **Note:** Check [/data/live_stream/update_example.json](https://github.com/WowzaMediaSystems/wsc-api-examples-ruby/tree/main/data/live_stream/update_example.json) for a JSON example.

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/")
...
  request = Net::HTTP::Patch.new uri.path
  # add the body to the request and parse the hash to JSON (or load it from a JSON file)
  request.body = {
    "live_stream": {
      LIVE_STREAM_SETTINGS
    }
  }.to_json
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -H 'Content-Type: application/json' -X PATCH -d '{
#   "live_stream": {
#     LIVE_STREAM_SETTINGS
#   }
# }' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
                                   "id": "kdgbsvth",
                                 "name": "My Updated Awesome Live Stream",
                                  "...": "..."
    }
}
```

### Start a live stream (PUT)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/start/")
...
  request = Net::HTTP::Put.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -X PUT -d '' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/start/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
        "state": "starting"
    }
}
```

### Reset a live stream (PUT)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/reset/")
...
  request = Net::HTTP::Put.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -X PUT -d '' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/reset/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
        "state": "resetting"
    }
}
```

### Stop a live stream (PUT)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/stop/")
...
  request = Net::HTTP::Put.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -X PUT -d '' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/stop/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
        "state": "stopped"
    }
}
```

### Poll the state of a live stream (GET)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/state/")
...
  request = Net::HTTP::Get.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/state/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
        "state": "started"
    }
}
```

### Poll the state of a player that was created with a live stream (GET)

> **Note:** The player ID can be found in the property 'player_id' in a live stream JSON response.

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/players/[PLAYER_ID]/state/")
...
  request = Net::HTTP::Get.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' https://api.cloud.wowza.com/api/v1/players/[PLAYER_ID]/state/
```

Example Response:
```json
# 200 OK
{
    "player": {
        "state": "activated"
    }
}
```

### Show thumbnail URL (preview image) of a live stream (GET)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/thumbnail_url/")
...
  request = Net::HTTP::Get.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/thumbnail_url/
```

Example Response:
```json
# 200 OK
{
    "live_stream": {
        "thumbnail_url": "http://..."
    }
}
```

### Delete a live stream (DELETE)

```ruby
uri = URI("https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/")
...
  request = Net::HTTP::Delete.new uri.path
...

# Equivalent cURL command:
#
# curl -H 'wsc-api-key: KEY' -H 'wsc-access-key: KEY' -X DELETE https://api.cloud.wowza.com/api/v1/live_streams/[LIVE_STREAM_ID]/
```

Example Response:
```json
# 204 NO CONTENT
```

<a name="references"></a>
## References

[Wowza Streaming Cloud REST API Reference](https://www.wowza.com/docs/wowza-streaming-cloud-rest-api-reference-documentation)

<a name="contact"></a>
## Contact

Wowza Media Systems™, LLC

Wowza Media Systems provides developers with a platform to create streaming applications and solutions. See [Wowza Developer Tools](https://www.wowza.com/developer) to learn more about our APIs and SDK.

<a name="license"></a>
## License

This code is distributed under the [BSD 3-Clause License](/LICENSE.txt).
