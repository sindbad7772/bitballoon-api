BitBalloon REST API
===================

BitBalloon is a hosting service for the programmable web. It understands your documents and provides an API to deploy sites, manage form submissions, inject javascript snippets and do intelligent updates of HTML documents. This is a REST-style API that uses JSON for serialization and OAuth 2 for authentication.

Making a request
----------------

All URLs start with `https://www.bitballoon.com/api/v1/`. **SSL only**. The path is prefixed with the API version. If we change the API in backward-incompatible ways, we'll bump the version marker and maintain stable support for the old URLs.

To make a request for all the sites you have access to, you'd append the sites index path to the base url to form something like https://www.bitballoon.com/api/v1/sites. In curl, that looks like:

```shell
curl -H 'User-Agent: MyApp (yourname@example.com)' https://www.bitballoon.com/api/v1/sites?access_token=oauth2_access_token
```

Authenticating
--------------

BitBalloon uses OAuth2 for authentication. All requests must use HTTPS. You'll need an application client key and a client secret before you can access the BitBalloon API. Please contact us at team@bitballoon.com for your credentials.

If you're making a public integration with BitBalloon for others to enjoy, you must use OAuth 2. This allows users to authorize your application to use BitBalloon on their behalf without having to copy/paste API tokens or touch sensitive login info.

The Oauth2 end user authorization endpoint is `https://www.bitballoon.com/oauth/authorize`.

Sites
=====

The `/sites` endpoint allows you to access the sites deployed on BitBalloon.

Get Sites
---------

* `GET /sites` will return all sites you have access to.

```json
[
  {
    "id":"3970e0fe-8564-4903-9a55-c5f8de49fb8b",
    "premium":false,
    "claimed":true,
    "name":"synergy",
    "custom_domain":"www.example.com",
    "url":"http://www.example.com",
    "admin_url":"https://www.bitballoon.com/sites/synergy",
    "screenshot_url":null,
    "created_at":"2013-09-17T05:13:08Z",
    "updated_at":"2013-09-17T05:13:19Z",
    "user_id":{"51f60d2d5803545326000005"},
  }
]
```

Get Site
--------

* `GET /sites/3970e0fe-8564-4903-9a55-c5f8de49fb8b` will return the site from its ID
* `GET /sites/www.example.com` will return the site matching the domain `www.example.com`

```json
{
  "id":"3970e0fe-8564-4903-9a55-c5f8de49fb8b",
  "premium":false,
  "claimed":true,
  "name":"synergy",
  "custom_domain":"www.example.com",
  "url":"http://www.example.com",
  "admin_url":"https://www.bitballoon.com/sites/synergy",
  "screenshot_url":null,
  "created_at":"2013-09-17T05:13:08Z",
  "updated_at":"2013-09-17T05:13:19Z",
  "user_id":{"51f60d2d5803545326000005"},
}
```

Create Site
-----------

Creating a site will initiate a new deploy. You can either create a new site with a list of files you intend to upload, or by posting a ZIP file.

### POST a ZIP file

* `POST /sites` with `zip=zip-file`. You must use `Content-Type: multipart/form-data` to send the zip file in a property called `zip`.

```json
{
  "id": "3970e0fe-8564-4903-9a55-c5f8de49fb8b",
  "subdomain": "some-autogenerated-subdomain",
  "url": "http://some-autogenerated-subdomain.bitballoon.com",
  "state": "processing",
  "required": []
}
```

This will return `201 Created` with the API URL for the new site in the `Location` header, along with a simplified JSON representation of the site. The site will be in the `processing` state and you can poll the URL in the `Location` header until the state has changed to either `ready` or `error`.

Here's an example of creating a new site from a zip file called `landing.zip` via Curl (assuming you've gone through the process to get an oauth access_token) 

```bash
curl -F "zip=@landing.zip;type=application/zip" https://www.bitballoon.com/api/v1/sites?access_token={access_token}
```

### POST a list of files

* `POST /sites` with `{files: {"/index.html": "SHA1_OF_YOUR_INDEX_HTML"}}` will create a new site in the `uploading` state. You must use `Content-Type: application/json`

The files object should contain all the files you wish to upload for this deploy, together with a SHA1 of the content of each file.

```json
{
  "id": "3970e0fe-8564-4903-9a55-c5f8de49fb8b",
  "subdomain": "some-autogenerated-subdomain",
  "url": "http://some-autogenerated-subdomain.bitballoon.com",
  "state": "uploading",
  "required": ["SHA1_OF_YOUR_INDEX_HTML"]
}
```

This will return `201 Created` with the API URL for the new site in the `Location` header, along with a simplified JSON representation of the site. The `required` property will give you a list of files you need to upload. BitBalloon will inspect the SHA1s you sent in the request. You'll only need to upload the files BitBalloon doesn't have on its servers. The `state` can be either `uploading` or `processing` depending on whether or not you need to upload any more files.

To upload any required files, use the `POST /sites/{site_id}/files` endpoint for each file.

Destroy Site
------------

* `DELETE /sites/{site_id}` will permanently delete a site 
* `DELETE /sites/{site_domain}` will permanently delete a site

This will return `200 OK`.


Submissions
===========

The `/submissions` endpoint gives access to the form submissions of your BitBalloon sites. You can scope submissions to a specific site (`/sites/{site_id}/submissions`) or to a specific form (`/forms/{form_id}/submissions`).

Get Submissions
---------------

* `GET /submissions` will return a list of form submissions

```json
[
  {
    "id":"5231110b5803540aeb000019",
    "number":13,
    "title":null,
    "email":"test@example.com",
    "name":"Mathias Biilmann",
    "first_name":"Mathias",
    "last_name":"Biilmann",
    "company":"BitBalloon",
    "summary":"Hello, World",
    "body":"Hello, World",
    "data": {
      "email":"test@example.com",
      "name": "Mathias Biilmann",
      "ip":"127.0.0.1"
    },
    "created_at":"2013-09-12T00:55:39Z",
    "site_url":"http://synergy.bitballoon.com"
  }
]
```
