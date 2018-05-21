# Admin API doc

## index
- [accounts](#accounts)
    - [`GET /api/accounts/:identifier`](#get-apiaccountsidentifier)
    - [`POST /api/accounts/`](#post-apiaccounts)
    - [`PATCH /api/accounts/:identifier/set_tier`](#patch-apiaccountsidentifierset_tier)
    - [`PATCH /api/accounts/:identifier/pwd`](#patch-apiaccountsidentifierpwd)
    - [`PATCH /api/accounts/:identifier/email`](#patch-apiaccountsidentifieremail)
- [auth](#auth)
    - [info](#authorizing-requests)
    - [`POST /api/auth`](#post-apiauth)
    - [`POST /api/auth/logout`](#post-apiauthlogout)
- [email templates](#email-templates)
    - [`GET /api/email_templates`](#get-apiemail_templates)
    - [`GET /api/email_templates/:name`](#get-apiemail_templatesname)
    - [`POST /api/email_templates`](#post-apiemail_templates)
    - [`DELETE /api/email_templates/:name`](#delete-apiemail_templatesname)
    - [`PATCH /api/email_templates/:name`](#patch-apiemail_templatesname)
    - [`POST /api/email_templates/:name/activate`](#post-apiemail_templatesnameactivate)
    - [`POST /api/email_templates/:name/deactivate`](#post-apiemail_templatesnamedeactivate)
    - [`POST /api/email_templates/:name/send`](#post-apiemail_templatesnamesend)
    - [`POST /api/email_templates/send`](#post-apiemail_templatessend)
- [libraries](#libraries)
    - [`GET /api/library_dump`](#get-apilibrary_dump)
    - [`PUT /api/library_dump`](#put-apilibrary_dump)
- [ui config](#ui-config)
    - [`GET /api/ui_config`](#get-apiui_config)
    - [`GET /api/ui_config/:host`](#get-apiui_confighost)
    - [`PUT /api/ui_config/:host`](#put-apiui_confighost)

## accounts

### `GET /api/accounts/:identifier`
returns information about an account using an identifier such as id, uuid, username or e-mail address. 

**example response**:
```$xslt
{
    "id": number,
    "uuid": "string",
    "email": "string",
    "username": "string",
    "given_name": null,
    "family_name": null,
    "organization": null,
    "country": null,
    "phone": null,
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "flags": [
    ],
    "tier": "string",
    "change_log": [
    ],
    "totp_secret": null,
    "technical_contact_email": null,
    "billing_contact_email": null,
    "requested_email": null,
    "requested_email_expires_at": null,
    "active": true,
    "verified": false,
    "tc_client_ip": null,
    "tc_accepted_at": null
}
```

### `POST /api/accounts/`
creates an account with username, email and password. `201` if successful, `400` if you passed bad data. a new account will be set as developer tier and active by default. 

**example request**:
```$xslt
{   
    "username": "string", 
    "email": "string", 
    "password": "string"
}
```

### `PATCH /api/accounts/:identifier/set_tier`
update an account's tier using UUID. `200` if successful, `404` if not found and `400` if you passed bad data ("developer" or "enterprise" for input).

**example request**:
```$xslt
{   
    "tier": "string"
}
```

### `PATCH /api/accounts/:identifier/pwd`
update an account's password using UUID. `204` if successful, `400` if you passed bad data. 

**example request**:
```$xslt
{   
    "password": "string"
}
```

### `PATCH /api/accounts/:identifier/email`
update an account's email using UUID. `204` if successful, `400` if you passed bad data. 

**example request**:
```$xslt
{   
    "email": "string"
}
```

## auth

### authorizing requests
- an authorized request includes an `x-ei-admin-token` header, obtained 
using [`POST /api/auth`]($post-apiauth). the token is valid for 24 hours,
but will expire if not used for 1 hour.
- the only endpoints that allow unauthenticated requests are: 
    - `/login` (for UI login)
    - `/api/auth` (so you can post auth without being redirected)
    - `/static/*` (so we can load the webapp)
- all unauthenticated requests will be redirected to `/login`

### `POST /api/auth`
checks username and password, returning `200` with a auth token,`401` 
(credentials don't match), `403` (no admin access), or `500` (something else failed).

**example body**: 
```
{
    id: "string"
    password: "string"   
}
```

**example response**:
```$xslt
{
    token: "64-character string to pass as x-ei-admin-token",
    username: "username of user you're logged in as",
    email: "email of user"
}
```

### `POST /api/auth/logout`
invalidates token passed in `x-ei-admin-token` header. no body is required. `200` response expected.

## email templates

### `GET /api/email_templates`
returns array of email template names.

### `GET /api/email_templates/:name`
returns data for the requested template (or `404` if not found).

### `POST /api/email_templates`
creates a new template. `201` if successful, `400` if you passed bad data.

**example body**: 
```
{
    name: "string",
    subject: "string",
    text: "plain text email body. {{values_like_this}} will be replaced with values when sent",
    html: "<html><body>same as text, but for html emails</body></html>"    
}
```

### `DELETE /api/email_templates/:name`
deletes an email template from the database. `204` if successful, `404` if not found.

### `PATCH /api/email_templates/:name`
update a specified email template. `200` if successful, `404` if not found.

**example body**: 
```
{
    subject: "string",
    text: "plain text email body. {{values_like_this}} will be replaced with values when sent",
    html: "<html><body>same as text, but for html emails</body></html>",
    active: boolean  
}
```

### `POST /api/email_templates/:name/activate`
no body. shortcut for PATCH with only `active` key. `200` if successful, `404` if not found.

### `POST /api/email_templates/:name/deactivate`
no body. shortcut for PATCH with only `active` key. `200` if successful, `404` if not found.

### `POST /api/email_templates/:name/send`
send an existing template to the email address specified in the body. `context` will be used 
to populate the `{{handlebars}}` variables in the email subject, text, and html. email will
be sent regardless of `active` property.

**example body** 
```
{
    email: "example@domain.com",
    context: {
        key_referenced_in_email_html_and_or_text: "value to replace that key"
        // etc...
    }  
}
```

### `POST /api/email_templates/send`
like [`POST /api/email_templates/:name/send`](#post-apiemail_templatesnamesend`), but 
for an arbitrary `subject`, `text`, and `html` specified in the request body.

**example body** 
```
{
    email: "example@domain.com",
    subject: "not an existing template",
    text: "hello, dear friend",
    html: "html can still be plain text!",
    context: {
        key_referenced_in_html_and_or_text: "value to replace that key"
        // etc...
    }  
}
```





## libraries
this duplicates the `bootstrap/libctl` from the API. 

### `GET /api/library_dump`
dumps a blob.

### `PUT /api/library_dump`
load a blob that looks like the dump, picking up any changes or additions.







## ui config

### `GET /api/ui_config`
returns map of `hostname: config` pairs.

**example payload**
```$xslt
{
    default: {
        features: {
            self_signup: true,
            show_feature1: false,
        },
        customCopy: {
            welcome_text: "We're so glad you're here!"
        }
    },
    azure: {
        features: {
            self_signup: false
            extra_property: true
        }
    },
    arbitrary_name: {
        features: {
            show_feature1: false,
            show_feature2: true
        }
    }
}
```

### `GET /api/ui_config/:host`
returns config key/values set only for requested host. does not include default values. 
`404` if not found.

**example payload**
```$xslt
// GET /api/ui_config/default
{
    host: "default",
    config: {
        features: {
            self_signup: true,
            show_feature1: false,
        },
        customCopy: {
            welcome_text: "We're so glad you're here!"
        }
    }
}
```

### `PUT /api/ui_config/:host`
create or update config for specified host. keys with non-null values will be set to '
that value, and keys set to "null" will be deleted. other existing host values are left as-is.
if all values on a host are deleted, the host is deleted as well.

**example body**
```
// PUT /api/ui_config/default
{
    features: {
        self_signup: false,
        show_feature1: null
    }
}
```
**example payload**
```$xslt
// status: 200
{
    host: "default",
    config: {
        features: {
            self_signup: false,
        },
        customCopy: {
            welcome_text: "We're so glad you're here!"
        }
    }
}
```
