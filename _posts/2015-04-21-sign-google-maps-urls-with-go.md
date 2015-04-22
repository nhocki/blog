---
layout: post
title: Sign Google Maps URLs with Go
excerpt: A simple way to sign your Google Maps for Work URLs with Go so you don't have to worry about it everywhere in your codebase.
date: 2015-04-21
---

[Google Maps API][api] is very powerful. You can easily create static images
with routes, calculate distance, time, turn-by-turn directions between two points,
and a lot more. While it's free for most cases, but if your product depends on
it you're probably using [Maps for Work][mfw] to be able to do more requests you
hit the [rate limit][rate].

As a Maps for Work user, you need to _authenticate_ your requests. This is kind
of a pain since it's not adding a `token` as a header, or adding a `key` param,
but instead you need to **sign** the request with the `CLIENT` and `SECRET`
pair from Google's console.

## Creating the Signature

To create a valid signature you'll need to get the full path (`path` and `query`
parts from the URL - make sure the query has your `CLIENT`) and then using the
`SECRET` you sign the full path using the HMAC-SHA1 algorithm.

It gets a little bit tricky since you get the `SECRET` in a modified base64
encoding and you need to use an unmodified version to create the signature, and
then encode the signature in the modified encoding. Luckily, the `base64`
package has the `base64.URLEncoding` scheme to handle this for you.

At [Ride][ride] we extracted all this in a simple function that takes an
`url.URL` object and *adds both the client and signature* to the query params. 
Notice that it will **use the free, public URL if there are no credentials**,
making it great for testing and development environments (so you don't consume
your rate quota on tests).

Here's it is:

```go
// SignURL adds the signature to a Google URL.
// Needed to make authenticated responses.
// https://developers.google.com/maps/documentation/business/webservices/auth#generating_valid_signatures
func SignURL(reqURL *url.URL) (*url.URL, error) {
  googleClientID := os.Getenv("GOOGLE_MAPS_API_CLIENT_ID")
  googleKey := os.Getenv("GOOGLE_MAPS_API_KEY")

  if googleKey == "" || googleClientID == "" {
    return reqURL, nil
  }

  googleBinaryKey, err := base64.URLEncoding.DecodeString(googleKey)
  if err != nil {
    log.Println("[ERROR] Can't decode Google key.", err)
    return nil, err
  }

  queryString := reqURL.Query()
  queryString.Set("client", googleClientID)
  reqURL.RawQuery = queryString.Encode()

  uri := reqURL.Path + "?" + reqURL.RawQuery
  mac := hmac.New(sha1.New, googleBinaryKey)
  mac.Write([]byte(uri))

  signature := base64.URLEncoding.EncodeToString(mac.Sum(nil))
  reqURL.RawQuery = reqURL.RawQuery + "&signature=" + signature

  return reqURL, nil
}
```

## Signing URLs

It's pretty easy to sign every maps API URL, we just create the URL and call the
function with it.

```go
// No error handling shown here.
mapsForWork, err := url.Parse("some-google-maps-url")
mapsForWork, err = maps.SignURL(mapsForWork)
```

And that's it. We'll have a signed URL in production and a free on in
development and tests.

## Testing it

Talking about tests, this function is really simple to test using [Google's
example keys][gek]. We simple set the environment to have those keys and sign
the example URL:

```go
func init() {
  // Setup Google's example test keys
  os.Setenv("GOOGLE_MAPS_API_CLIENT_ID", "clientID")
  os.Setenv("GOOGLE_MAPS_API_KEY", "vNIXE0xscrmjlyV-12Nj_BvUPaw=")
}

// https://developers.google.com/maps/documentation/business/webservices/auth#signature_examples
func TestSignURI(t *testing.T) {
  tests := []struct {
    uri       string
    signedURI string
  }{
    {
      "/maps/api/geocode/json?address=new+york",
      "/maps/api/geocode/json?address=new+york&client=clientid&signature=charf2htjkoscpr-rqcehzbszie=",
    },
  }

  for _, tt := range tests {
    req, err := url.parse(tt.uri)
    testing.ok(t, err)

    requrl, err := signurl(req)
    testing.ok(t, err)

    testing.equals(t, tt.signedURI, reqURL.String())
  }
}
```

As a closing note, I have to admit I have been enjoying Go way too much. I find
it really simple and easy to learn. There are amazing [open][docker]
[source][coreos] [projects][influxdb] using it making them great learning
resources.

I'm looking forward to attend [Gophercon](http://www.gophercon.com/) later this
year to learn even more about it. If you're looking into learning a new
language, give Go a try. You won't regret it.

[influxdb]: http://influxdb.com/
[coreos]: https://coreos.com/
[docker]: https://www.docker.com/
[api]: https://developers.google.com/maps/documentation/webservices/
[mfw]: https://www.google.com/intx/en/work/mapsearth/
[ride]: https://www.ride.com
[gek]: https://developers.google.com/maps/documentation/business/webservices/auth#signature_examples
[rate]: https://developers.google.com/maps/documentation/business/articles/usage_limits
