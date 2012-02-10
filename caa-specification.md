# Cover Art Archive specification

## coverartarchive.org API

For developers, the coverartarchive.org is the only thing they need to
know about. Any requests to fetch cover art MUST go through this
service. There are 3 classes of end points, text surrounded in braces
indicates a variable.

### /release/{mbid}/

#### Summary

Fetch a listing of available cover art for a MusicBrainz release.

#### Accepted Methods

- GET
- HEAD

#### Responses

-   200 if there is a release with this MBID. The response body
    should be a listing of available cover art for this release in
    the most specific format that matches the clients Accept header.

    An example of a response body that we might want to support is
    JSON, or XML. See the section on 'cover art data' for a
    definition of what is included in this listing.

-   400 if {mbid} cannot be parsed as a valid UUID.

-   404 if there is no release with this MBID.

-   405 if the request method is not one of GET or HEAD.

-   406 if the server is unable to generate a response suitable to
    the Accept header.

-   503 if the user has exceeded their rate limit.

#### Example

    > GET /release/99b09d02-9cc9-3fed-8431-f162165a9371 HTTP/1.1
    > Host: coverartarchive.org
    > Accept: application/json

    < HTTP/1.0 200 OK
    < Status: 200
    {
      "release": {
        "name": "We Hear You",
        "artist_credit": "Luke Vibert"
      },
      "artwork": [
        {
          "approved": true,
          "id": "af3d070",
          "front": true
        }
      ]
    }


### /release/{mbid}/front

#### Summary

Fetch the image that is most suitable for refering to as the "front" of a
release. This is intentionally vague, and users will help curate this data into
something that is meaningful. A suggested initially style is to use artwork that
users would most likely expect to see in:

* Digital shops when searching for the release
* Their portable media player
* The folder icon in their file browser, if supported
* What they would expect to find if they were looking for this release in a
  shop.

#### Accepted Methods

- GET
- HEAD

#### Responses

- 307 if the community have decided upon a "front" image for this
  release.

- 400 if {mbid} cannot be parsed as a valid UUID.

- 404 if there is either no release with this MBID, or the
  community have not chosen an image to represent the front of a
  release.

- 405 if the request method is not GET or HEAD.

- 503 if the user has exceeded their rate limit.

#### Example

    > GET /release/99b09d02-9cc9-3fed-8431-f162165a9371/front HTTP/1.1
    > Host: coverartarchive.org

    < HTTP/1.1 307 Temporary Redirect
    < Status: 307
    < Location: http://coverartarchive.org/release/99b09d02-9cc9-3fed-8431-f162165a9371/af3d070


### /release/{mbid}/{id}

#### Summary

Fetch a specific piece of artwork. Possible {id} values can be found by parsing
the response of a /release/{mbid} request.

#### Accepted methods

- GET
- HEAD

#### Responses

- 307 redirect to a binary image, matching the Content-Type to a
  value in the requests Accept header field.

- 404 if a release with this MBID cannot be found.

- 405 if the request method is not GET or HEAD.

- 503 if the user has exceeded their rate limit.

#### Example

    > GET /release/foo/af3d070.jpg HTTP/1.1
    > Host: coverartarchive.org

    < HTTP/1.1 307 Temporary Redirect
    < Status: 307
    < Location: http://archive.org/download/mbid-99b09d02-9cc9-3fed-8431-f162165a9371/mbid-99b09d02-9cc9-3fed-8431-f162165a9371-af3d070.jpg


--------

## Cover Art Archive Metadata

The Cover Art Archive provides a collection of metadata with each release, which
allows users to determine what cover art can be used for. The metadata consists
of a set of entries, where each entry contains:

- The id of the entry. This is a URL that can be accessed with a GET request to
  fetch an image.
- The edit ID which uploaded this image.
- Whether the image is pending peer review or has been accepted.
- Whether the image is the 'front' image, or not.

The metadata also contains a description of the release so that the Internet
Archive are able to index this artwork. This will contain:

- The release name as a string
- The release artist credit as a string

### File naming

Files are named using integers derived from the current high resolution system
time, for example via the `Time::HiRes::time` function in Perl. The exact
formula to create a new file name is:

    int((time() - 1327528905) * 100)

This filename is allocated by MusicBrainz at the time of upload, and will never
change.

--------

## The Cover Art Archive and MusicBrainz

### Community Editing

MusicBrainz provides 3 new edit types for manipulating cover art in the Cover
Art Archive.

#### Add Cover Art

This edit attaches a specific piece of cover art to a release. When the edit is
successfully entered, the metadata index is updated to contain this image, with
its status set to 'pending review'.

If the community approves of this edit, the metadata index will be updated to
indicate that it has been approved.

If the community rejects this edit, then the entry will be removed from the index
and the image from the file store.

#### Remove Cover Art

This edit allows a piece of cover art to be removed from the Cover Art
Archive. When the edit is entered, no changes happen at the Cover Art Archive.

If the edit is accepted, the artwork is removed and the entry is removed from
the index.

If the edit is rejected, the artwork is kept.

#### Edit Cover Art

This edit allows users to replace a cover art image and/or edit its metadata. When
the edit is entered, no changes happen to the Cover Art Archive.

When the edit is accepted, the entry in the index is merged with the definition
in the edit. If the edit included a new image, the artwork replaces the
existing cover art — providing it hasn't changed since the edit was entered. If
it has changed, the edit must fail as a conflict.

If the edit is accepted, the Cover Art Archive does not change.
