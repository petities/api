Api
===

This README documents the application interface (api) for a website
built on the petitions.eu code. 

Other systems can connect with the database of petitions through this
interface.

Users
-----

The api is designed for the needs of:

-   governments as petition recipients
-   lead petitioners as campaigners
-   application developers
-   news media
-   researchers and journalists
-   a European federation of petitioning sites.

Structure of calls
------------------

Three important tables in the database are accessible:

-   petitions
-   signatures
-   organisations.

The common syntax for the path in a call is:

`{{hostname}}/api/v1/{table}?api_key={api_key}&language={language}`

The parameters of the call follow after another & sign.

Path parameters:

-   `v1` stands for version 1, the 2018 version
    -   options: \[v1\]
-   `id` the identifier of a petition or organisation\
-   `table`
    -   options: \[petitions, signatures, organisations\]

GET parameters:

-   `api_key` the key to the api supplied to you, a test key (with some
    limitations) is freely available
-   required, test-key 1234 gives a fixed count: 3 (also for
    signature\_timestamps for a petition)
-   `language` is the language of the petition. Without this it defaults
    to Dutch.
    -   default: nl
    -   options: \[en, de, nl\_fr, nl\_gr\]
-   `count` the number of results returned
    -   petitions default: 10
    -   signatures default: 100
    -   signatures max: 1000 (max unlimited for certain api-keys)
    -   ignore for organisations (show all)
-   `skip` not yet implemented, required for pagination etc. Maybe to
    allow some apikeys full scraping.
-   `petitions` the ids of petitions for which to get signatures\
-   `fields` the element of table petitions or signatures (options for
    certain api-keys)
    -   `petitions`:
        -   options: \[description, initiators, statement, request,
            signatures\_count\]
            -   default: \[id, name\]
            -   options: \[date\_projected, created\_at,
                answer\_due\_date, status\]
    -   `petitions/{id}`:
        -   options: \[description, signatures\_count,
            signature\_timestamps\]
            -   default: \[id, name, description\]
            -   options: \[date\_projected, created\_at,
                answer\_due\_date, status\]
    -   `signatures` always just timestamp
        -   default: \[timestamp\]
        -   options: \[timestamp\] count unlimited for certain api-keys
        -   `from` select signatures from this date (for certain
            api-keys)
        -   `to` select signatures until this date (for certain
            api-keys)
    -   `organisations` the ids of organisations for which to get
        petitions/signatures
        -   default: \[all\]
        -   options: \[id\]

All chronological data is ordered descending based on the created\_at
(for petitions) or confirmed\_at (signatures) date value.

Examples
--------

-   List of petitions\
    `{{hostname}}/api/v1/petitions?api_key={api_key}&language={language}&count={10}&organisations={org1,org2,...}&fields={description,signatures\_count,initiators}`

-   One petition\
    `{{hostname}}/api/v1/petitions/{id}?api_key={api_key}&language={language}&fields={description,signatures\_count,...,signature\_timestamps}`

-   List of signatures\
    `{{hostname}}/api/v1/signatures?api_key={api_key}&language={language}&petitions={pet1,pet2,...}&organisations={org1,org2,...}&count={100}&from={date}&to={date}`
    -   petition: id, name
    -   organisation: id, name
    -   signature: timestamp

When no petition or organisation id is specified: latest count
signatures

-   List of all organisations
    -   id
    -   name
    -   kind

`{{hostname}}/api/v1/organisations?api_key={api_key}&language={language}`

### POST a signature

Post a signature for a specified petition

`{{hostname}}/api/v1/petitions/{id}/signature?api_key={api_key}&language={language}`

All fields required.

in:

    {
        "name": "Firstname Lastname",
        "email": "user@provider.tld",
        "city": "Place",
        "visible": 1
    }

out: success 200

    {
        "message": "{language:Thank you, confirmation e-mail will be sent}"
    }

out: failed 400

    {
        "error": "Required fields missing",
        "fields": ["name,email,city,visible"]
    }

### PUT an update to a petition

`{{hostname}}/api/v1/petitions/{id}?api_key={api_key}&language={language}`

    {
       "status": "{concept,staging,live,to_process,withdrawn,in_process,rejected,completed}",
       "date_projected": "{DATE}",
       "reference_field": "{256characters}",
       "answer_due_date": "{DATE}"
    }

All fields optional.

out: success 200

    {
        "message": "language:Update request received, no further notifications will be sent.",
        "fields": {
            "status": "new status"
    }

### POST/PUT explained

1.  Petitions can only be added through the webinterface, but certain
    properties to existing petitions can be changed by certain users
    (depending on their api key) with a PUT request. Recipients can
    update the status, relevant dates of or reference to a petition. The
    reference is what the organisation uses to identify the petition as
    an incoming document. The date is the day of the handover of the
    petition. The status is `concept`, `staging`, `live`, `to_process`,
    `withdrawn`, `in_process`, `rejected` or `completed`. The
    `answer_due_date` is when an answer is expected.
2.  Signatures can be added, but not confirmed or modified through this
    interface. Signatures are confirmed by following a unique link from
    an e-mail sent to the e-mail address entered by the signatory. There
    the data can be modified by the signatory. Use the content types:
    name, email, city, visibility. To specify the petition, the id of
    the petition is mentioned first. The petition is not a type of the
    signature here! The conditions for these content types are:
    -   `name` (required), datatype string, a 255 characters maximum,
        validated with regular expression `/\A.+( |\.).+\z/` which
        results in at least an initial and name
    -   `city` (optional), datatype string, a 255 characters maximum
    -   `email` (required), datatype string, a 255 characters maximum,
        validated with regular expression
        `/\A([\w+\-].?)+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i`
    -   `visibility` (optional), datatype boolean, 1 (true) or 0 (false,
        the default).
3.  Organisations receive and answer petitions. Modifications are done
    by the moderator of the website and are not possible through the
    api.

### GET explained

1.  Information about petitions can be retrieved through the interface
    as long as it does not infringe upon anyone's privacy. You can
    assume the moderator of new petitions filters such information out.
    Use these fields: description, initiators, statement, request,
    created\_at, date, status, signatures.
2.  Information about signatures can be retrieved as long as they can
    not identify a signatory. With the timestamps of signatures for a
    certain petition the development of support for a petition can be
    examined or visualised for example.
3.  An organisation that is petitionable can be queried about the
    petitions addressed to the organisation. Use the queries for
    petitions to get the number of signatures for petitions addressed to
    an organisation, the texts of petitions addressed to an
    organisation, how many signatures these petitions collected, the
    status of petitions addressed to a an organisation, etc.
4.  An organisation that is petitionable can be queried about the
    petitions addressed to the organisation. Use the fields `concept`,
    `staging`, `live`, `to_process`, `withdrawn`, `in_process`,
    `rejected` or `completed` to specify which kinds of petitions. Use
    the queries for petitions to get the number of signatures for
    petitions addressed to an organisation, the texts of petitions
    addressed to an organisation, how many signatures these petitions
    collected, the status of petitions addressed to a an organisation,
    etc.

Use cases
---------

-   Description + chart of running petition
    -   call:
        `{{hostname}}/api/v1/petitions/{id}?api_key={api_key}&language={language}&fields={name,description,signature\_timestamps}`
-   List of initiators adressing an organisation
    -   call
        `{{hostname}}/api/v1/organisations/{id}?api_key={api_key}&language={language}&fields={initiators}`

Authentication
--------------

An API key is required to communicate with the api. Institutional users
ask their service provider for one. Service providers contact
webmaster\@petities.nl for access to the key-management environment.
Researchers and app-deveopers contact webmaster\@petities.nl for an
api-key. Test keys with restrictions are for free.

Contact
-------

Please e-mail webmaster\@petities.nl if you find bugs or have questions
about the api not answered here.
