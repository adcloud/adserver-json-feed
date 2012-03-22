## Using the API

Given you have a list of placements each identified by an unique ID. For
each of those placements you want to display or deliver ads from our
network. For accessing those ads we provide an API which we also use
internally. This API accepts some paramters which will be further
discussed in other chapters. The following chapter will give you a first
glance on the API, how the response will look like and what caveats
there are.

endprologue.

The main argument you have to provide when calling the API, is the ID of
the placement for which you need information abouts its possible ads.
For each placement there is a list of ads from which the client can
choose to display a selection. This list can be accessed via an JSON
feed from the adcloud adserver:

<plain>
http://a.adcloud.net/adcloud/<PLACEMENT_ID>?dimension=<DIMENSION>&sizes=<SIZES>
</plain>

To give you a first impression of what the data look like we provide an
[example feed][1].  The example feed consists of 25 ads, which is the
maximum amount of ads you will receive at this point when consuming the
feed for a placement:

<javascript>
{
  "id": "3691",
  "ads": [
    {
      "tag": 0,
      "id": 28447,
      "flash": 0,
      "booking": 80977,
      "product": 10229,
      "display": 0,
      "image_text": 1,
      "prio": 85,
      "fc": 0,
      "postview": "",
      "postview_type": "1",
      "fc_days": 0,
      "exclude": [
        "3013"
      ],
      "keyword_lifetime": 0,
      "keywords": {},
      "head": "Partnervermittlung ab 30!",
      "text": "Starten Sie jetzt die Suche nach Ihrem Traumpartner, mit eDarling.",
      "exclusion_keywords": {},
      "link": "Jetzt bei eDarling anmelden!",
      "locations": {},
      "url": "http://ad.doubleclick.net/clk;247082507;65276591;p",
      "images": {
        "158x90": {
          "flash": 0,
          "display": 0,
          "src": "http://ads.adcloud.net/p/10229/313979.gif",
          "width": "158",
          "height": "90"
        }
      },
      "type": "ad"
    },
    // up 24 more ads ...
  ],
  "backfill": [],
  "modified": "2012-03-20 11:44:58"
}
</javascript>

### The Ad-Object

You can always find a detailed description of the the entire [feed][4],
the [ad object][2] an the [ad image object][3] for our feed. But for now
let us have a look at the most important attributes of an ad:

* `id` - The unique ID for this ad
* `head` - A short headline to be displayed (max. 255 characters)
* `text` - A long text to be displayed along with the ad
* `prio` - The priority of this ad to be selected
  INFO: Our optimization
  system assigns numbers between 0 to 85, but the value can manually be
  increased up to 255, whereas higher is better.
* `url` - Destination for the user to be redirected after a click
* `link` - Description of the url to be displayed
* `postview` - Optional tracking image after this ad is displayed. This
  can be used by our customers to track the amount of impressions them
  self
* `postview_type` - The postview image could be either a simple image
  (`value: 1`) or an IFrame (`value: 2`)
* `images` - A list of all available images. The structure should be
  quite self-explanatory. For further information have a look in the
  JSON Schema.

NOTE:
When you consume the feed please make sure, that we will only send out
ads that are valid at the time of your request. And still this selection
could be a subset of a bigger collection of ads. If you need to know
which ads are new and which were deleted between two subsequent
requests, you have to implement such logic in your side.

So there could be two tracking pixels that must be displayed for each
ad impression (the postview image and the adcloud view tracking).

### Adcloud View Tracking

To be able to track if a an ad was displayed or not a transparent GIF
needs to be requested. The query parameters of that request need to
include information to associate the request with the displayed ad and
thus count the impression. The following Javascript example shows how to
construct the URL:

<javascript>
var url = 'http://t.adcloud.net/t.gif' +
    '?r=' + Math.random() +
    '&ac_d=1' +
    '&ac_t=' + static_placement.topic +
    '&ac_pm=' + static_placement.id +
    '&ac_pt=' + static_placement.page_type +
    '&ac_k=' +
    '&ac_z=' +
    '&ac_f=0' +
    '&ac_p=' + current_ad.product +
        'x' + '1' +
        'x' + current_ad.id +
        'x' + current_ad.booking;
</javascript>

The code requires the following preconditions:

 * You requested the JSON feed for a given placement ID
 * You create a `static_placement` based on static metadata we will
   provide you before hand
 * You need the `current_ad` from the list of all received ads based on
   its display position

### Postview Image

If you want to track the impressions on your own too, you have two
options: Using an image or an IFrame. The following examples will give
you an idea of how to use those options:

<javascript>
    var img = new Image;
    img.src = current_ad.postview_url.replace(/\[timestamp\]/i, Math.random());
</javascript>

<javascript>
    document.write('<iframe '
        + 'vspace="0" '
        + 'hspace="0" '
        + 'allowTransparency="true" '
        + 'scrolling="no" '
        + 'marginWidth="0" '
        + 'marginHeight="0" '
        + 'frameBorder="0" '
        + 'width="1" height="1" '
        + 'border="0" '
        + 'src="'+ current_ad.postview.replace(/\[timestamp\]/i, Math.random()) +'" '
        + '></iframe>'
    );
</javascript>

You need again the `current_ad` based on its position. In both cases you need
to replace the `[timestamp]` placeholder within the `postview_url` with a
random number.

A complete view tracking example can be found as a [Gist on GitHub][6].

### Dimension and Sizes

As you could see in the example feed URL there are two more parameters:
`dimension` and `sizes`. There meaning is described as the following:

 * `dimension` - Dimension of the complete placement. The dimension have
   to be defined in the format `WIDTHxHEIGHT` (i.e.: `125x100`).
 * `sizes` - The sizes of the single ad "slots". This parameter can be a
   comma separated list of dimension strings (i.e.: `120x85,240x150`).

The feed will always contain **all** ads, that _either_ fit in a single
slot _or_ the entire placement. We will only filter the ads, but **not**
the images within an ad. Thus an ad will always contain all images, even
if only one will fit the given placement.

### Redirect URL

So far we only looked at counting ad impressions. Another important part
to take care of when displaying ads, is the URL to which the user will
be redirected after actually clicking the ad. That URL is stored in the
`url` field. We not only count the ad impressions but also count the
clicks from the user. Due to this every click have to be routed through
our system. For that reasons _every_ URL have to be properly encoded on
the client site. We can not provide you the encoded URL because you need
to add some query parameters first. Again, here is another Javascript
example of how to construct the URL for a given ad. The variables
`static_placement` and `current_ad` have the same meaning as in the
examples before:

<javascript>
var click_url = current_ad.url.replace(/\[TIMESTAMP\]/gm, new Date().getTime());
var final_url = "http://a.adcloud.net/redirect/" +
    Encoding.base64(click_url) + // @SEE: https://gist.github.com/09fb127bd28a57091117
    '?ac_d=1' +
    '&ac_t=' + static_placement.topic +
    '&ac_pm=' + static_placement.id +
    '&ac_pt=' + static_placement.page_type +
    '&ac_k=' +
    '&ac_z=' +
    '&ac_f=0' +
    '&ac_p=' + current_ad.product +
        'x' + '1' +
        'x' + current_ad.id +
        'x' + current_ad.booking +
    '&ac_ad=' + current_ad.id +
    '&ac_pr=' + current_ad.product +
    '&ac_pos2=1' +
    '&ac_part=0' +
    '&ac_b=' + current_ad.booking +
    '&ac_ci=1047';
</javascript>

The resulting URL could look something like the following:

<javascript>
var result_url = 'http://a.adcloud.net/redirect/'
    + 'QWRjbG91ZA@@aHR0cDovL2FkNC5hZGZhcm0xLmFkaXRpb24uY29tL3JlZGk*c2lkPTQ5MzIxMiZraWQ9Mjc0NjU4JmJpZD05NDczNDI@'
    + '?ac_d=1'
    + '&ac_t=2'
    + '&ac_pm=3691'
    + '&ac_pt=0'
    + '&ac_k='
    + '&ac_z='
    + '&ac_f=0'
    + '&ac_pc=1'
    + '&ac_p=12066x2x32985x85173'
    + '&ac_ad=32985'
    + '&ac_pr=12066'
    + '&ac_pos2=1'
    + '&ac_part=0'
    + '&ac_b=85173'
    + '&ac_ci=1047';
</javascript>

NOTE:
We use a slightly modified Base64 encoding where we replace `/` with `*`
and `=` with `@`. You can find the Javascript [code for this encoding
here][5].

### Feedback

This guide should enable you to use and integrate the Adserver JSON Feed
with your systems. We try to anticipate problems and describe them here,
so that you don't have to run into them by yourself. A straight forward
description of the API is our goal.

If you ever happen to discover a mistake or something we haven't thought
of upfront, please don't hesitate to contact us.

[1]: http://a.adcloud.net/adcloud/3691?dimension=728x90&sizes=158x90,158x90
[2]: https://raw.github.com/adcloud/adserver-json-feed/master/source/ad-schema.json
[3]: https://raw.github.com/adcloud/adserver-json-feed/master/source/ad-image-schema.json
[4]: https://raw.github.com/adcloud/adserver-json-feed/master/source/feed-schema.json
[5]: https://gist.github.com/09fb127bd28a57091117
[6]: https://gist.github.com/a5c5ac329da156c2c68a
