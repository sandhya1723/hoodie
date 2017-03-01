hoodie.account
==============

The account object in the client-side Hoodie API covers all user and
authentication-related operations, and enables you to do previously
complex operations, such as signing up a new user, with only a few lines
of frontend code. Since `data in Hoodie is generally bound to a
user </camp/hoodieverse/glossary.html#private-user-store>`__, it makes
sense to familiarise yourself with **account** before you move on to
`store </camp/techdocs/api/client/hoodie.store.html>`__.

``hoodie-account-client`` is a JavaScript client for the 
`Account JSON API <http://docs.accountjsonapi.apiary.io/>`_. 
It persists session information in localStorage (or your own store API) and 
provides front-end friendly APIs for the authentication-related operations as 
mentioned above.

There is also an `admin-specific account client 
<https://github.com/hoodiehq/hoodie-account-client/blob/master/admin>`_.

Example
-------

.. code:: js

    // Account loaded via <script> or require('@hoodie/account-client')
    var account = new Account('https://example.com/account/api')

    if (account.isSignedIn()) {
    renderWelcome(account)
    }

    account.on('signout', redirectToHome)

Constructor
-----------

.. code:: js

    new Account(options)

table

Returns ``account`` API.

Example

.. code:: js

    new Account({
        url: '/api',
        id: 'user123',
        cacheKey: 'myapp.session',
        validate: function (options) {
            if (options.username.length < 3) {
            throw new Error('Username must have at least 3 characters')
            }
        }
    })

account.ready
-------------

`Read-only`. Promise that resolves once the account instance loaded its current state from the store.

account.id
----------

`Read-only`. Returns the account id. Cannot be accessed until the `account.ready <https://github.com/hoodiehq/hoodie-account-client#accountready>`_ promise resolved.

account.username
----------------

`Read-only`. Returns the username if signed in, otherwise ``undefined``. Cannot be accessed until the `account.ready <https://github.com/hoodiehq/hoodie-account-client#accountready>`_ promise resolved.

account.validate
----------------

Calls the function passed into the Constructor. Returns a Promise that resolves to ``true`` by default

.. code:: js

    account.validate(options)

table

+----------------------+-------------+-------------+
| Argument             | Type        | Required    |
+======================+=============+=============+
| ``options.username`` | String      | No          |
+----------------------+-------------+-------------+
| ``options.password`` | String      | No          |
+----------------------+-------------+-------------+
| ``options.profile``  | Object      | No          |
+----------------------+-------------+-------------+

Resolves with an argument.

Rejects with any errors thrown by the function originally passed into the Constructor.

Example

.. code:: js

    var account = new Account({
    url: '/api',
    cacheKey: 'app.session',
    validate: function (options) {
        if (options.password.length < 8) {
        throw new Error('password should contain at least 8 characters')
        }
    }
    })

    account.validate({
    username: 'DocsChicken',
    password: 'secret'
    })

    .then(function () {
    console.log('Successfully validated!')
    })

    .catch(function (error) {
    console.log(error) // should be an error about the password being too short
    })

account.isSignedIn
------------------

Returns ``true`` if user is currently signed in, otherwise ``false``. 
Cannot be accessed until the `account.ready <https://github.com/hoodiehq/hoodie-account-client#accountready>`_ promise resolved.

.. code:: js

    account.isSignedIn()

account.hasInvalidSession
-------------------------

Checks ``account.session.invalid property``. Returns ``true`` 
if user has invalid session, otherwise ``undefined``. 
Cannot be accessed until the account.ready promise resolved.

.. code:: js

    account.hasInvalidSession()

account.signUp
--------------

Creates a new user account on the Hoodie server. 
Does `not` sign in the user automatically, `account.signIn <https://github.com/hoodiehq/hoodie-account-client#accountsignin>`_ must be called separately.

.. code:: js

    account.signUp(accountProperties)

+--------------------------------+---------+----------+
| Argument                       | Type    | Required |
+================================+=========+==========+
| ``accountProperties.username`` | String  | Yes      |
+--------------------------------+---------+----------+
| ``accountProperties.password`` | String  | Yes      |
+--------------------------------+---------+----------+

Resolves with ``accountProperties``:

.. code:: json

    {
        "id": "account123",
        "username": "pat",
        "createdAt": "2016-01-01T00:00.000Z",
        "updatedAt": "2016-01-01T00:00.000Z"
    }

Rejects with:

+----------------------+-----------------------------------------+
| InvalidError	       | Username must be set                    |
+======================+=========================================+
| ``SessionError``     | Must sign out first                     |
+----------------------+-----------------------------------------+
| ``ConflictError``    | Username **<username>** already exists  |
+----------------------+-----------------------------------------+
| ``ConnectionError``  | Could not connect to server             |
+----------------------+-----------------------------------------+

Example

.. code:: js

    account.signUp({
        username: 'pat',
        password: 'secret'
    }).then(function (accountProperties) {
        alert('Account created for ' + accountProperties.username)
    }).catch(function (error) {
        alert(error)
    })

account.signOut
---------------

Deletes the user’s session

.. code:: js

    account.signOut()

Resolves with ``sessionProperties`` like `account.signin <https://github.com/hoodiehq/hoodie-account-client#accountsignin>`_, but without the session id:

.. code:: json

    {
        "account": {
            "id": "account123",
            "username": "pat",
            "createdAt": "2016-01-01T00:00.000Z",
            "updatedAt": "2016-01-02T00:00.000Z",
            "profile": {
                "fullname": "Dr. Pat Hook"
            }
        }
    }

Rejects with:

+-----------+------------------------------------------------------+
| ``Error`` | A custom error thrown in a ``before:signout`` hook   |
+-----------+------------------------------------------------------+

Example

.. code:: js

    account.signOut().then(function (sessionProperties) {
        alert('Bye, ' + sessionProperties.account.username)
    }).catch(function (error) {
        alert(error)
    })

account.destroy
---------------

Destroys the account of the currently signed in user.

.. code:: js

    account.destroy()

Resolves with ``sessionProperties`` like `account.signin <https://github.com/hoodiehq/hoodie-account-client#accountsignin>`_, but without the session id:

.. code:: json

    {
        "account": {
            "id": "account123",
            "username": "pat",
            "createdAt": "2016-01-01T00:00.000Z",
            "updatedAt": "2016-01-02T00:00.000Z",
            "profile": {
                "fullname": "Dr. Pat Hook"
            }
        }
    }

Rejects with:

+---------------------+----------------------------------------------------+
| ``Error``           | A custom error thrown in a ``before:destroy`` hook |
+---------------------+----------------------------------------------------+
| ``ConnectionError`` | Could not connect to server                        |
+---------------------+----------------------------------------------------+

Example

.. code::

    account.destroy().then(function (sessionProperties) {
        alert('Bye, ' + sessionProperties.account.username)
    }).catch(function (error) {
        alert(error)
    })

account.get
-----------

Returns account properties from local cache. Cannot be accessed until the `account.ready <https://github.com/hoodiehq/hoodie-account-client#accountready>`_ promise resolved.

.. code:: js

    account.get(properties)

+-----------------+------------------------------------+---------------------------------------------------------------------------------------------------------+------------+
| Argument        | Type                               | Description                                                                                             | Required   |
+=================+====================================+=========================================================================================================+============+
| ``properties``  | String or Array of strings         | When String, only this property gets returned. If array of strings, only passed properties get returned | No         |
+-----------------+------------------------------------+---------------------------------------------------------------------------------------------------------+------------+

Returns object with account properties, or ``undefined`` if not signed in.

Examples

.. code:: js

    var properties = account.get()
    alert('You signed up at ' + properties.createdAt)
    var createdAt = account.get('createdAt')
    alert('You signed up at ' + createdAt)
    var properties = account.get(['createdAt', 'updatedAt'])
    alert('You signed up at ' + properties.createdAt)

account.fetch
-------------

Fetches account properties from server.

.. code:: js

    account.fetch(properties)

+----------------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+
| Argument       | Type                       | Description                                                                                                                                                                  | Required    |
+================+============================+==============================================================================================================================================================================+=============+
| ``properties`` | String or Array of strings | When String, only this property gets returned. If array of strings, only passed properties get returned. Property names can have '.' separators to return nested properties. | No          |
+----------------+----------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+

Resolves with ``accountProperties``:

.. code:: json

    {
        "id": "account123",
        "username": "pat",
        "createdAt": "2016-01-01T00:00.000Z",
        "updatedAt": "2016-01-02T00:00.000Z"
    }

Rejects with:

+---------------------------+------------------------------+
| ``UnauthenticatedError``  | Session is invalid           |
+---------------------------+------------------------------+
| ``ConnectionError``       | Could not connect to server  |
+---------------------------+------------------------------+

Examples

.. code:: js

    account.fetch().then(function (properties) {
        alert('You signed up at ' + properties.createdAt)
    })
    account.fetch('createdAt').then(function (createdAt) {
        alert('You signed up at ' + createdAt)
    })
    account.fetch(['createdAt', 'updatedAt']).then(function (properties) {
        alert('You signed up at ' + properties.createdAt)
    })

account.update
--------------

Update account properties on server and local cache

.. code:: js

    account.update(changedProperties)

+-----------------------+-----------+--------------------------------------------------------------------------------+----------+
| Argument              | Type      | Description                                                                    | Required |
+=======================+===========+================================================================================+==========+
| ``changedProperties`` | Object    | Object of properties & values that changed. Other properties remain unchanged. | No       |
+-----------------------+-----------+--------------------------------------------------------------------------------+----------+

Resolves with accountProperties:

.. code:: json

    {
        "id": "account123",
        "username": "pat",
        "createdAt": "2016-01-01T00:00.000Z",
        "updatedAt": "2016-01-01T00:00.000Z"
    }

Rejects with:

+--------------------------+----------------------------------------+
| ``UnauthenticatedError`` | Session is invalid                     |
+--------------------------+----------------------------------------+
| ``InvalidError``         | Custom validation error                |
+--------------------------+----------------------------------------+
| ``ConflictError``        | Username **<username>** already exists |
+--------------------------+----------------------------------------+
| ``ConnectionError``      | Could not connect to server            |
+--------------------------+----------------------------------------+

Example

.. code:: json

    account.update({username: 'treetrunks'}).then(function (properties) {
        alert('You are now known as ' + properties.username)
    })