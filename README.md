# Spotify playlist API

Public colaboration of our upcoming (web-based) playlist API announced at [Music Hack Day Amsterdam, Apr 24 2010](http://amsterdam.musichackday.org/).

## Authentication

OAuth

## A playlist entity

    version       The version of the playlist in the Spotify system (integer).
    identifier    The Spotify URI of the playlist.
    title         Title of the playlist.
    creator       Username of the user who created the playlist.
    trackList     List of tracks.
      identifier    The Spotify URI of the track.
      title         Name of the track
      creator       Artist(s) performing the track.
      album         Album on which the track appears.
      trackNum      The track number on the album (integer).
      duration      Length of the track in milliseconds (integer).
      meta          Further metadata mapped by key prefixed by "spotify:meta:".
        starred       True if the track is "starred" by the authenticated caller.


### XSPF example

    <?xml version="1.0" encoding="utf-8"?>
    <playlist xmlns="http://xspf.org/ns/0/" xmlns:sp="http://spotify.com" version="1" sp:version="123">
      <identifier>spotify:user:smedjan:playlist:6welunS19b7RD9lodXrhuG</identifier>
      <title>Spotify playlist</title>
      <creator>smedjan</creator>
      <trackList>
        <track>
          <identifier>spotify:track:0v7T2Xu3kpHj1JiaOvt7gm</identifier>
          <title>Ghost Trail</title>
          <creator>Cult Of Luna</creator>
          <album>Eternal Kingdom</album>
          <meta rel="spotify:meta:starred">true</meta>
        </track>
        ...
      </trackList>
    </playlist>

### JSON example

    {
      "version": 123,
      "identifier": "spotify:user:smedjan:playlist:6welunS19b7RD9lodXrhuG",
      "title": "Spotify playlist",
      "creator": "smedjan",
      "trackList": [
        {
          "identifier": "spotify:track:0v7T2Xu3kpHj1JiaOvt7gm",
          "title": "Ghost Trail",
          "creator": {
            "title": "Cult Of Luna",
            "identifier": "spotify:artist:7E7fJJpdVgr1F3pfAfRtHe"
          },
          "album": {
            "title": "Eternal Kingdom",
            "identifier": "spotify:album:51fF4JNsyx99YAWixRgmVh"
          },
          "trackNum": 3,
          "duration" 710000,
          "meta": {"starred": true}
        }
      ]
    }

Further called `{playlist}`.

## Outline

    GET /user/playlists -> ordered list of {playlist}s
      List a users regular playlists
    
    GET /user/playlists/starred -> ordered list of {playlist}s
      List the contents of a users "Starred" playlist
      @auth required
    
    GET /user/playlists/purchases -> ordered list of {playlist}s
      List the contents of a users "Starred" playlist
      @auth required
    
    GET /user/playlists/purchases -> ordered list of {playlist}s
      List the contents of a users "Starred" playlist
      @auth required

## Error responses

As the API is restricted to HTTP we make use of the HTTP protocol when it comes to error indication. The status of any HTTP response is represented by a standard status code. The ranges are as follows:

See [Status Code Definitions, RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) for further details.

The response entity of an error (unless in the case of "bodyless" responses, as defined by [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616.html)) contains a simple human-readable message, primarily aiding development.

XML response:

    HTTP/1.1 400 Bad Request
    
    <error message="The foo argument is empty" />

JSON response:

    HTTP/1.1 400 Bad Request
    
    {"message":"The foo argument is empty"}


## API methods

### GET /user/playlists -> ordered list of {playlist}s

    GET /smedjan/playlists.json

JSON response:

    {
      "playlists":[
        {
          "identifier": "spotify:user:smedjan:playlist:6welunS19b7RD9lodXrhuG",
          "title": "Spotify playlist",
          "creator": "smedjan"
        }
        ...
      ]
    }

XML response:

    <?xml version="1.0" encoding="utf-8"?>
    <playlist xmlns="http://xspf.org/ns/0/" xmlns:sp="http://spotify.com" version="1">
      <trackList>
        <track>
          <extension application="http://spotify.com">
            <sp:playlist collaborative="true" />
          </extension>
          <identifier>spotify:user:smedjan:playlist:6welunS19b7RD9lodXrhuG</identifier>
          <title>Spotify playlist</title>
          <creator>smedjan</creator>
        </track>
        ...
      </trackList>
    </playlist>


### GET /user/playlists/starred -> {playlist}

Retrieve `user`s special "Starred" playlist, containing all tracks a user have "starred".

The response is the same as for any playlist.


### GET /user/playlists/purchases -> {playlist}

Retrieve `user`s special "Purchases" playlist, containing all tracks a user has purchased.

The response is the same as for any playlist.





