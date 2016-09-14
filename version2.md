To meet the expanding needs of clients, we are developing a RESTful, RAML documented
version of our API.

The fundamental structures are similar to the [v1 API](https://github.com/Loudr/api-specs/blob/master/requests.md).

**Submitting Requests is still only available via v1.** [See here.](https://github.com/Loudr/api-specs/blob/master/requests.md#making-a-request)

### Request Authentication via Secret

The argument `secret` should contain your client token. 
It may be provided in the query string, or in the JSON request data.

**Migrating from v1?** - `_secret` has become `secret`.


## Query Sound Recordings

Endpoint:
`https://loudr.fm/api/client/sound_recordings`


### Pagination

Every query returns a `rel` object containing a link to the `next_page`.
Use this to paginate through the requests.

```json
"rel": {
    "next_page": "/api/client/sound_recordings?count=1&cursor=cXVlcnlUaGVuRmV0Y2g7OTsxMjUzMjY4ODo2OXNzZWJROVQ1dXR4T09hcTA2Y0lROzExOTc2MTg5OjZEVldldzEwUXF1Y2ZZVlRCSTIwdGc7MTIzOTc4NjA6ZU1jcWMtaFNTR2F2eXpCLUF0cDRKdzsxMTU5OTM0Mjo5WXRxUUhDSFN2eWY1bWpLdlVJbmhBOzEyMzA0NzI3OlJGT0xrQmpaUU5tWTdZRHltWjJlY2c7MTIxOTI5NTU6ckZ3OFUtcWFUOHV3TnpHcEpoRWltZzsxMTk5MjM2ODpFSTlHYUtVWFJIdU14TEdOZ2RmWUN3OzExNTk5MzQxOjlZdHFRSENIU3Z5ZjVtakt2VUluaEE7MTE5MDQ2NDg6Y241QVQwQnhUeFdMOU4wRVVyQzd3ZzswOw%3D%3D&secret=xxx"
  }
}
```

### Results

The query will return an array, `results`, containing each "sound recording resource."

```json
{
  "results": [
    { ... }
  ]
}
```

## Individual Resources

Endpoint:
`https://loudr.fm/api/client/sound_recording/<client-id>/<vendor-id>`

The API path for a recording is comprised for the client ID, and vendor ID.
https://loudr.fm/api/client/sound_recording/api/client/sound_recording/xxx/abc123 is the home of request "abc123" provided by client "xxx".

This path is provided from the queries in `rel["canonical"] == "/api/client/sound_recording/xxx/abc123"`




# Resources

## Sound Recording `resource:sound-recording`

uri:
    `string-uri`. URI for the recording.

licensor:
    `string`. Licensor of the request. 'licensor' of the content - in the context of
    a streaming platform, this would be the distributor that initially
    licensed and provided the sound recording underlying this request.

license_config:
    `resource:license-config` License configuration.

license_fulfilled_at:
    `integer-timestamp`. Time when this request was fulfilled.
    A license request is considered fulfilled when its been
    successfully licensed and is ready to be distributed. Importantly,
    see the related `earliest_clearance_at` property, which indicates
    WHEN a request may be distributed at the earliest.

earliest_clearance_at:
    `integer-timestamp`. Earliest timestamp when this request may be distributed.
    A license request may be cleared for distribution at a
    date that trails the `license_fulfilled_at` timestamp. If this is the
    case, `earliest_clearance_at` returns a timestamp indicating when the
    release may go live. A null value indicates that there is no
    additional release restriction being implied.

requests:
    `list of resource:requests`. List of provided request metadata.

recording:
    `resource:recording`. Sound recording metadata.

fulfillment_status:
    `resource:fulfillment-status`. Fulfillment status metadata.

fulfillment:
    `resource:fulfillment`. Details from the fulfillment record.

is_public_domain:
    `bool`. Has this request been identified as a public domain usage?

matched_compositions:
    `list of resource:composition`. Compositions matched to this request.

client:
    `string-uri`. Client which owns this request.

license_kind:
    `string`. Kind of license request. Should always be `client request`.

license_status:
    `int`. One of the [license_status enum values](https://github.com/Loudr/api-specs/blob/master/enums.md#license-statuses).

license_status_str:
    `string`. Status of request in english.

is_rejection:
    `bool`. Has this request been rejected?

is_hard_rejection:
    `bool`. Is this a HARD rejection? Requests which have been 'hard rejected' may not be resubmitted.

rejections:
    `list of resource:rejection`. List of applicable rejections.

rel:
    `object`. Contains links relative to this resource.


## Recording `resource:recording`

Provided metadata about a specific recording.

title:
    `string`. Recording title.
artist:
    `list of string`. Recording artist(s).

album:
    `string`. Recording album the song is released on, if any.

isrc:
    `string`. Recording ISRC.

upc:
    `string`. UPC for recording's release.

label:
    `string`. Label for this release.

licensor:
    `string`. Licensor for this release.

vendor_id:
    `string`. Vendor's ID.

vendor_album_id:
    `string`. Vendor's album ID.


## Request `resource:request`

An individual song request provided for with a sound recording.
More than one may exist per request, if the request is understood to be a medley.
If Loudr determines that a recording is a medley, additional requests will be added
to accomodate this discovery.

provided_research:
    `resource:provided-research`. Object containing provided research for this request.


## Provided Research `provided-research`

Provided research, used to help identify works. 

composition_index: 
    `int`. Index of this request. Helps match requests with matched compositions.
    For example, if the a `matched_compositions` resource has `composition_index: 2`,
    it corresponds to the `requests` resource with `composition_index: 2` as well.

title: 
    `list of string`. Provided composition titles.

artist: 
    `list of string`. Original performing artist(s).

composer: 
    `list of string`. Original composer(s).

author: 
    `list of string`. Original author(s) of the song.
    Authors and composers may overlap, if so, they
    only need to be mentioned once.

album: 
    `list of string`. Original album the song was released on, if any.

source: 
    `list of string`. Original source material, such as a movie or 
    video game title.

link: 
    `list of string`. URL to the source material.

iswc: 
    `string`. International Standard Musical Work Code.
    
notes: 
    `string`. Any additional notes to aid composition search.

suggested_composition: 
    `string-uri`. Composition suggested by user as it already exists in Loudr's DB.


## License Config `resource:license-config`

Describes the quantity and configuration of the license request.
Serialized as an object containing `{license_type: units}`. 
A units value of -1 represents continuous administration, not fixed quantity.

```json
"license_config": {
    "stream": -1,
    "dpd": -1
}
```

Requests infinite streaming & digital downloads.


## Fulfillment Status `resource:fulfillment-status`

Represents the fulfillment status for a request.

license_fulfilled:
    `bool`. Has this request been fulfilled?

license_fulfilled_at:
    `timestamp-int`. When was this request fulfilled?

maximum_expense:
    `int`. Maximum expense imposed by this request.


## Fulfillment Detail `resource:fulfillment`

Describes the status of fulfillment for a recording.

completed:
    `bool`. Has this fulfillment been completed.
    property_name="archived")

compositions:
    `list of resource:composition-usage`. List of composition usages contained and fulfilled by this request.

fulfillment_license_ids:
    `list of int`. License fulfillment IDs.

fulfillment_methods:
    `list of string`. Methods used to fulfill this request.

territories:
    `list of string`. List of territories this license is being fulfilled in.



## Composition Usage `resource:composition-usage`

Represents the specific fulfillment strategy for a given composition, on a given request.
Contained within `resource:fulfillment`.

composition:
    `resource:composition`. Composition linked to this usage.

publishers:
    `list of string`. Publisher names who were used to fulfill this request.

license_receipt_ids:
    `list of string`. Receipt IDs for the licenses.



## Composition `resource:composition`

Represents a composition in Loudr's database which matched to this request.

title:
    `string`. Title of the composition.

alias:
    `list of string`. Alternate titles for the composition.

composition_index:
    `int`. Used when the composition relates to a request index in a given way.
    For example, if the a `matched_compositions` resource has `composition_index: 2`,
    it corresponds to the `requests` resource with `composition_index: 2` as well.



# Example Sound Recording

```json
{
  "fulfillment": null,
  "matched_compositions": [
    {
      "publishers": [
        "Sony/ATV Music Publishing LLC"
      ],
      "kind": "composition",
      "display_name": "LE PARDON DU CHEVREUIL",
      "title": "LE PARDON DU CHEVREUIL",
      "uri": "composition/dabt2KGm",
      "composition_index": 0,
      "author_names": [
        "Laurent Lescarret"
      ],
      "alias": [],
      "rel": {
        "canonical": "/api/composition/dabt2KGm"
      },
      "authors": [
        {
          "ipi": null,
          "role": {
            "code": "CA",
            "name": "Composer & Author",
            "id": 1
          },
          "name": "Laurent Lescarret"
        }
      ]
    }
  ],
  "is_public_domain": false,
  "fulfillment_status": {
    "maximum_expense": 0,
    "license_fulfilled_at": null,
    "license_fulfilled": false
  },
  "is_rejection": false,
  "rel": {
    "canonical": "/api/client/sound_recording/xxx/abc123"
  },
  "license_fulfilled_at": null,
  "license_kind": "client request",
  "payment_complete": false,
  "license_status": 2,
  "is_hard_rejection": false,
  "renewal_of": null,
  "requests": [
    {
      "provided_research": {
        "album": [],
        "artist": [],
        "iswc": "T7027742505",
        "author": [
          "Laurent Lescarret"
        ],
        "composition_index": 0,
        "source": [],
        "link": [],
        "composer": [
          "Laurent Lescarret"
        ],
        "title": [
          "LE PARDON DU CHEVREUIL"
        ],
        "suggested_composition": "",
        "notes": null
      }
    }
  ],
  "payment_initiated": null,
  "license_config": {
    "stream": -1
  },
  "uri": "sound_recording/creator/xxx--abc123",
  "renewal_complete": null,
  "recording": {
    "album": "Lieu-dit (Bonus Track Version)",
    "artist": [
      "Doriand"
    ],
    "title": "Le pardon du chevreuil",
    "vendor_id": "abc123",
    "upc": "3700551739820",
    "label": "Johnny Flyer",
    "vendor_album_id": null,
    "isrc": "FR13Z1100005",
    "licensor": null
  },
  "earliest_clearance_at": null,
  "client": "creator/xxx",
  "renewal_failed": null,
  "licensor": null,
  "license_status_str": "In Progress",
  "rejections": []
}
```
