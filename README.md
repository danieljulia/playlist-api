# Spotify playlist API

Public collaboration of our upcoming (web-based) playlist API during
[Music Hack Day Amsterdam, Apr 24 2010](http://amsterdam.musichackday.org/).

Abilities:

- Retrieve the list of playlists for a users
- Modify the list of playlists for a users
  - Add a playlist (the left hand column in the desktop client)
  - Remove a playlist
  - Reorder the list
- Create a new playlist
- Query a specific playlist
- Query a special playlist (i.e. "Starred" or "Purchases")
- Modify a specific playlist

## Authentication

OAuth

## A playlist entity

    version       The version of the playlist in the Spotify system (integer).
    identifier    The Spotify URI of the playlist.
    title         Title of the playlist.
    creator       Username of the user who created the playlist.
    description   Free text description.
    image         Playlist image.
    trackList     List of tracks.
      identifier    The Spotify URI of the track.
      title         Name of the track.
      creator       Artist(s) performing the track.
      album         Album on which the track appears.
      trackNum      The track number on the album (integer).
      duration      Length of the track in milliseconds (integer).
      meta          Further metadata mapped by key prefixed by "spotify:meta:".
        starred       True if the track is "starred" by the authenticated caller.

## Ideas & requests under consideration

- Requesting older versions e.g. stateless undo.
- Store arbitrary values (key-value style) in a playlist
- Embedded statistics (e.g. track popularity, subscribers, etc)
- 

### XSPF example

    <?xml version="1.0" encoding="utf-8"?>
    <playlist xmlns="http://xspf.org/ns/0/" xmlns:sp="http://spotify.com"
        version="1" sp:version="123">
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
        },
        ...
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

As the API is restricted to HTTP we make use of the HTTP protocol when it comes
to error indication. The status of any HTTP response is represented by a
standard status code. The ranges are as follows:

See [Status Code Definitions, RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
for further details.

The response entity of an error (unless in the case of "bodyless" responses, as
defined by [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616.html))
contains a simple human-readable message, primarily aiding development.

XML response:

    HTTP/1.1 400 Bad Request
    
    <error message="The foo argument is empty" />

JSON response:

    HTTP/1.1 400 Bad Request
    
    {"message":"The foo argument is empty"}


# API methods

---

## Querying playlists

### GET /user/playlists/regular -> ordered list of {playlist}s

Retrieve the list of regular playlists for a `user`.

**Authorization:** required

    GET /smedjan/playlists.json

JSON response:

    {
      "playlists":[
        {
          "version": "4",
          "identifier": "spotify:user:smedjan:playlist:6welunS19b7RD9lodXrhuG",
          "title": "Spotify playlist",
          "creator": "smedjan"
        }
        ...
      ]
    }

XML response:

    <?xml version="1.0" encoding="utf-8"?>
    <playlist xmlns="http://xspf.org/ns/0/" xmlns:sp="http://spotify.com" 
        version="1" sp:version="4">
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


### GET /playlist/{id} -> {playlist}

Retrieve any playlist by identifier `{id}`.

**Authorization:** no


### GET /user/playlists/special/starred -> {playlist}

Retrieve `user`s special "Starred" playlist, containing all tracks a user have "starred".

**Authorization:** required, same owner

The response is the same as for any playlist.


### GET /user/playlists/special/purchases -> {playlist}

Retrieve `user`s special "Purchases" playlist, containing all tracks a user has purchased.

**Authorization:** required, same owner

The response is the same as for any playlist.

---

## Manipulating playlists

As we perform server-side merging of modifications, a modification request might
result in a few different status codes:

- `200 OK` -- Modifications applied and your version is the latest.

- `201 Created` -- The playlist was created.

- `205 Reset Content` -- The modification was accepted but resulted in a dirty state. See below for details.

You will *never* get a `409 Conflict`, since conflicts are resolved automagically.

**Modifications implying incorporation of remote edits:**

If a modification was accepted but resulted in a dirty state, you will get a
205 Reset Content response, indication your client should reload the playlist
from the server.

Example:

    > GET /playlists/xyz123
    # Some time passes and the playlist is modified by another client.
    > POST /playlists/xyz123
    < 205 Reset Content
    > GET /playlists/xyz123


### PUT,POST /playlist <- {playlist}

Create a new playlist as the authenticated user.

**Authorization:** required

**Request entitiy**:

A {playlist} object, except the `version` and `identifier` members should not be present.


### POST /playlist/{id}/add?index <- list of track URIs

Insert one or more tracks into playlist `{id}` at `{index}`.

**Authorization:** required, same owner or collaborator

- `index` The position in the list where to insert the tracks. Indices are 0-based.

**Request entitiy**:

List of tracks URIs

Example:

    POST /playlist/6welunS19b7RD9lodXrhuG/add?index=4
    Content-type: application/json
    
    ["spotify:track:5xft6jBZvMHRD1jyTDPQXx","spotify:track:1MD4tX2g5hx0D2WQ6JsC2m"]


### POST /playlist/{id}/remove?src-index&src-count&dst-index <- list of track URIs

Insert one or more tracks into playlist `{id}` at `{index}`.

**Authorization:** required, same owner or collaborator

- `src-index` The start position of the tracks to move.
- `src-count` The number of tracks, starting at `src-index`, to move (or "select").
- `dst-index` The position where to move (or "land") the tracks, expressed in your versions coordinates.

Example -- move the first two tracks down past the next two tracks:

List before the modification:

    0. Track A
    1. Track B
    2. Track C
    3. Track D

POSTing:

    POST /playlist/6welunS19b7RD9lodXrhuG/remove?src-index=0&src-count=2&dst-index=2

List after the modification:

    0. Track C
    1. Track D
    2. Track A
    3. Track B


**Request entitiy**:

List of tracks URIs

Example:

    POST /playlist/6welunS19b7RD9lodXrhuG/add?index=4
    Content-type: application/json
    
    ["spotify:track:5xft6jBZvMHRD1jyTDPQXx","spotify:track:1MD4tX2g5hx0D2WQ6JsC2m"]






