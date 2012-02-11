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
          "id": "135741621",
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
    < Location: http://archive.org/download/mbid-99b09d02-9cc9-3fed-8431-f162165a9371/mbid-99b09d02-9cc9-3fed-8431-f162165a9371-135741621.jpg


### /release/{mbid}/back

#### Summary

Fetch the image that is most suitable for refering to as the "back" of a
release. This is intentionally vague, and users will help curate this data into
something that is meaningful. A suggested initial style is to use artwork that
users would most likely expect to see in:

* A tracklisting, barcode, and label
* What they would expect to find on the back cover if they were looking for this 
  release in a shop.

#### Accepted Methods

- GET
- HEAD

#### Responses

- 307 if the community have decided upon a "back" image for this
  release.

- 400 if {mbid} cannot be parsed as a valid UUID.

- 404 if there is either no release with this MBID, or the
  community have not chosen an image to represent the back of a
  release.

- 405 if the request method is not GET or HEAD.

- 503 if the user has exceeded their rate limit.

#### Example

    > GET /release/99b09d02-9cc9-3fed-8431-f162165a9371/back HTTP/1.1
    > Host: coverartarchive.org

    < HTTP/1.1 307 Temporary Redirect
    < Status: 307
    < Location: http://archive.org/download/mbid-99b09d02-9cc9-3fed-8431-f162165a9371/mbid-99b09d02-9cc9-3fed-8431-f162165a9371-135822686.jpg


### /release/{mbid}/{id}

#### Summary

Fetch a specific piece of artwork. Possible {id} values can be found by parsing
the response of a /release/{mbid} request.

#### Accepted methods

- GET
- HEAD

#### Responses

- 307 redirect to a binary image. This redirected request may resolve to a 404
  if the thumbnail does not exist.

- 404 if a release with this MBID cannot be found.

- 405 if the request method is not GET or HEAD.

- 503 if the user has exceeded their rate limit.

#### Example

    > GET /release/foo/135741621.jpg HTTP/1.1
    > Host: coverartarchive.org

    < HTTP/1.1 307 Temporary Redirect
    < Status: 307
    < Location: http://archive.org/download/mbid-99b09d02-9cc9-3fed-8431-f162165a9371/mbid-99b09d02-9cc9-3fed-8431-f162165a9371-135741621.jpg


### /release/{mbid}/{id}-(250|500)

#### Summary

Fetch a thumbnail for a specific piece of artwork. Possible {id} values can be
found by parsing the response of a /release/{mbid} request. The current
supported thumbnail sizes are 250px and 500px.

#### Accepted methods

- GET
- HEAD

#### Responses

- 307 redirect to a binary image. This redirected request may resolve to a 404
  if the thumbnail does not exist.

- 404 if a release with this MBID cannot be found.

- 405 if the request method is not GET or HEAD.

- 503 if the user has exceeded their rate limit.

#### Example

    > GET /release/99b09d02-9cc9-3fed-8431-f162165a9371/135741621-250.jpg HTTP/1.1
    > Host: coverartarchive.org

    < HTTP/1.1 307 Temporary Redirect
    < Status: 307
    < Location: http://archive.org/download/mbid-99b09d02-9cc9-3fed-8431-f162165a9371/mbid-99b09d02-9cc9-3fed-8431-f162165a9371-135741621-250.jpg


### OPTIONS support

All end points at the coverartarchive.org support the HTTP OPTIONS method, to
determine which request methods are valid for each resource. On an OPTIONS
request, the server will do no processing, other than returning an empty
response with the 'Allow:' header field set to the support methods for that
resource. For supported methods, consult the specification of each individual
resource.

--------

## Cover Art Archive Metadata

The Cover Art Archive provides a collection of metadata with each
release, which allows users to determine what cover art can be used
for.  The metadata will be stored and served as application/json.  The
metadata consists of a list of entries, where each entry contains:

- _image_: the full coverartarchive.org url to the original image
- _thumbnails_: { small: “http://coverartarchive.org/...-250.jpg”, large: “http://coverartarchive.org/...-500.jpg” }
- _types_: list of one or more types for the image (see below).
- _front_: boolean
- _back_: boolean
- _comment_: a free text comment
- _order_: a single integer which specifies the order of this image in relation to other images
- _appproved_: whether the image was approved by the musicbrainz edit system
- _edit_: full url to the edit on musicbrainz (e.g. musicbrainz.org/edit/123)

The metadata also contains a description of the release so that the Internet
Archive are able to index this artwork. This will contain:

- _title_: the release title as a string
- _artist_: the artist credit as a string
- _barcode_: the barcode as a string
- _catalog_numbers_: a list of catalog numbers, each item is a string

See the included "example.json" for a full example of how this
file will look.

### Cover art types

The list of image types will be defined as part of this CAA spec, but
can be amended in the future using the normal StyleCouncil process.
So far the types we've agreed on are:

- Other


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
