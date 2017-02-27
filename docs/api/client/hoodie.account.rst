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

..code::

    account.signUp(accountProperties)
    
<3<3<3

