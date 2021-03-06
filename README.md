# Documented below is the usage for the API LAPasswordManager.

## Start Note:
LAPasswordManager (Look Another Password Manager) is an API and webapp, which functions as a place where you can store and get your passwords as you desire. 
This webapp was created to be an extension of https://github.com/IsaiahSama/Password_Manager (Another Password Manager [APM] of mine), and therefore, APM has been rewritten to provide a direct way to access and use this API. However, the documentation below is for those that wish to access the API using their own way.

You may have up to 100 unique account_name / password pairs.

## About the API:
This API allows you to create, get, update, delete, store and generate passwords. When making a request through the API, you will be sent an email containing a key, that will allow you to interact with the API for 5 hours. This is to ensure your safety and privacy when trusting us to keep your passwords secure.

## Before Using the API:

In order to use this API, you will need to have an account already created. To do this, you may visit https://lapasswordmanager.herokuapp.com. After creating an account, that's all you need before using the API. 

When using the API for the first time in a while (About 5 hours), an email will be sent to you to verify that it is you who is making the request. After, all subsequent requests for 5 hours will no longer request that verification key.

The base URL for all requests is ```https://www.lapasswordmanager.herokuapp.com/api/v1/```. For the following documentation of this API, this base URL will be referred to as the constant `BASE`.

## Note:
For all API requests, valid `JSON` is expected. And in return, valid `JSON` Will be returned.

All responses will ALWAYS contain at least the following:
A successful response will be in the form of:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true,
            "MESSAGE": "A success message"
        },
        "RESPONSE": {}
    }
}
```
Where `MESSAGE` is a success message containing information
Where `RESPONSE` is a field holding requested or returned information. The data type of this value varies, and will be explicitly stated for each of the below routes

Whereas, if an error has occurred, the following `JSON` will be returned
```json
{
    "LAPM":{
        "SUCCESS_OR_FAILURE":{
            "SUCCESS": false,
            "MESSAGE": "Any extra messages",
            "ERROR": "An error message", 
            "EXTRA": []
        }
    }
}
```
Where `ERROR` is the error that occurred.
Where `MESSAGE` is any additional messages provided from the server.
Where `EXTRA` is any extra data. Data type of `EXTRA` will be defined specifically for the routes, but will be typically a list.

## Errors
The `ERROR` field will be one of the following.
* `NoSuchEmailError`: This is raised when a given email address does not exist in the database. For this, you would need to go to https://lapasswordmanager.herokuapp.com to create one
* `LoginFailedError`: This occurs when a login has failed due to an incorrect password
* `NotVerifiedError`: This occurs when a user attempts to use the API without first verifying their session
* `InvalidVerificationError`: This occurs when an incorrect Verification token is passed
* `InvalidDataException`: This exception occurs when the data given does not match the data expected
* `DuplicateDataException`: This occurs when making using the `STORE` route, duplicate data is found, and the `OVERWRITE` key is set to `false`
* `MissingDataExcetpion`: This occurs on either of the 2 conditions
    1. Using the `GET` route, no existing entry was found with the given email name, and `IGNOREMISSING` is false.
    1. Using the `DELETE` route, but no existing entry was found
* `MaxEntriesReached`: This occurs when you have reached your limit of 100 entries. Removing previous, unneeded entries can free up space for your operations.


In case of a success, the other data that was requested, will be also be provided in a `RESPONSE` entry.


## Activating your API
As mentioned twice before now, you have to activate your session before any of your requests can be made. An activated session lasts for 5 hours.
The `URL` for this is:
```py
BASE + "activate"
```

To activate a session, the following are required. These are: The registered email and the password for that email. 
Example:
```py
import requests
import json

BASE = "https://www.lapasswordmanager.herokuapp.com/api/v1/"

# LAPM (Look Another Password Manager), Should be the outermost of the JSON, and everything else comes as a value to this key
to_post = {
    "LAPM": {
        "AUTH": {
            "EMAIL": "someone@example.com",
            "PASSWORD": "securepassword
        }
    }
}


url = BASE + "activate"
# Therefore: The URL is https://www.lapasswordmanager.herokuap.com/api/v1/activate

x = requests.post(url, json=to_post)
# Checks to see that the connection was made successfully
x.raise_for_status()

# To get the response data, you can do
# response = x.json()
# Or if you want it directly as a python dictionary for you to use, you can do
response = loads(x.json())
# You can then see if the operation was a success by doing
# Checks if the operation was a success
if response["LAPM"]["SUCCESS_OR_FAILURE"]["SUCCESS"] == True:
    # Print
    print("All is good")
else:
    print("Something went wrong")

# Prints out the message
print(response["LAPM"]["SUCCESS_OR_FAILURE"]["MESSAGE"])
```

For a successful response, the response will be in the format:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true,
            "MESSAGE": "message"
        },
        "RESPONSE": "the_email_that_the_email_was_sent_to"
    }
}
```
After a successful response, a verification email will be sent to the given email address, containing the verification code to be sent in the following request.

An unsuccessful response will return the default `SUCCESS_OR_FAILURE` response as defined earlier in these docs.

In order to verify now, and have your account available for you to use. The procedure is almost the exact same.
Example:

```py
import requests
import json

BASE = "https://www.lapasswordmanager.herokuapp.com/api/v1/"

# LAPM (Look Another Password Manager), Should be the outermost of the JSON, and everything else comes as a value to this key
to_post = {
    "LAPM": {
        "AUTH": {
            "EMAIL": "someone@example.com",
            "PASSWORD": "securepassword",
            "VERICODE": "insert verification code here"
        }
    }
}

# Therefore: The URL is https://www.lapasswordmanager.herokuapo.com/api/v1/activate
url = BASE + "activate"

x = requests.post(url, json=to_post)
# Checks to see that the connection was made successfully
x.raise_for_status()

# To get the response data, you can do
response = x.json()
# Or if you want it directly as a python dictionary for you to use, you can do
response = loads(x.json())
```
Where `VERICODE` is the code received from the email. If you are already verified, then the response will be the same as the one below.

For a Successful response, will return:

```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true,
            "MESSAGE": "You have successfully been authorized."
        },
        "RESPONSE": "200 OK TRUE"
    }
}
```

An unsuccessful response will be returned if an error occurs, or you do not have an account registered.


Returns only the basic `SUCCESS_OR_FAILURE` for a failed response as shown above.

## Storing and Updating Passwords.
The main purpose of this application is to allow you to store your passwords safely online. Now, we will look at the API method to do such.

To store passwords, the URL used is:
```py
BASE + "passwords/store/"
```

The format of the expected `JSON` is as follows:
```py

# Authentication details
auth = {
    "EMAIL": "youremailhere@example.com",
    "PASSWORD": "yourpassword"
}

to_post = {
    "LAPM": {
        # Put the auth details here
        "AUTH": auth,
        # Put the account names and passwords to store here. Supports from 1 entry, up to a max of 20 entries at once
        "STORE": [{"account_name1": "account_password1"}, {"account_name2": ""}]},
        "OVERWRITE": False
    }
}
```
Where `STORE` is a list of dictionaries which can store up to 20 unique entires, meaning you can upload 20 different pairs to be stored. 
If an empty string is provided for a password, we will generate a password of 12 characters for you.

### NOTE: IF AN EMPTY STRING IS PROVIDED FOR A PASSWORD, WE WILL AUTOMATICALLY GENERATE A SECURE ONE FOR YOU.
Where `OVERWRITE` is a boolean value. True means that any entries with the same `account_name` already stored, will be updated to the new `account_password`. False, means that this entry will be  skipped, all other non-duplicate entries will be inserted. If all given entries are duplicates, then the error response will be returned. If at least one entry is valid, then a Success response will be returned, and a list of `{account_name: account_password}` will be returned in the `FAILED` key as shown below.

### Note, because of the OVERWRITE entry, this same url is used to update entries that you have saved.


A successful response will be formatted as follows:

 ```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true,
            "MESSAGE": "A success message"
        },
        "RESPONSE": [{"account_name": "account_password"}],
        "FAILED": []
    }
}
```
Where `RESPONSE` is a list of dictionaries (Mapping account_name to account_password) of the names of all added accounts.
Where `FAILED` is a list of dictionaries of all entries that were duplicates.

An error response will follow the same structure of `SUCCESS_OR_FAILURE`. Which is:
```json
{
    "LAPM":{
        "SUCCESS_OR_FAILURE":{
            "SUCCESS": false,
            "MESSAGE": "any extra information",
            "ERROR": "an error message",
            "EXTRA": [{"account_name": "account_password"}]
        }
    }
}
```
Where `MESSAGE` is any extra content, and `ERROR` is an error message.
Where `EXTRA` is a list of dictionaries mapping account_name to account_password, of all entries that were duplicates. Note, this will only be returned if the the error returned was due to `OVERWRITE` being `false`.

## Reading/ Getting Passwords

A very important part of storing passwords, is then being able to retrieve them as you want them. For this, the following URL is used:
```py
BASE + "passwords/get/"
```

The format of the expected JSON is as follows:
```py
auth = {
    "EMAIL": "your@email.here",
    "PASSWORD": "Good password"
}

to_post = {
    "LAPM": {
        # Auth goes here
        "AUTH": auth,
        # Get is a list of account names with a minimum of 0, and a max of 20
        # If GET is an empty list, all accounts will be returned
        "GET": ["account_name_1", "account_name_2"],
        "IGNOREMISSING": True
    }
}
```
Where `GET` is a list of account names that you wish to receive the passwords for. This list must contain at least 1 item, and can have up to 20 entries. Anything after 20 entries will be ignored.
Where `IGNOREMISSING` is a boolean. If `true`, then any account names that cannot be found will be returned in the `FAILED` entry of the success response. If False, and an entry is missing, will return the Error response instead, where `MESSAGE` will be a string of all failed account_names separated by commas (, ).

A successful response will return the following JSON:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true,
            "MESSAGE": "A success message"
        },
        "RESPONSE": [{"account_name": "account_password"}],
        "FAILED": ["account_name1"]
}
```
Where `RESPONSE` is a list of dictionaries, mapping the account names to their respective passwords.
Where `FAILED` is a list of all account_names that could not be found. Note, this will only be returned if `IGNOREMISSING` is true.

In case of an error, the following response will be returned:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": false,
            "MESSAGE": "Any extra content",
            "ERROR": "The error that occurred",
            "EXTRA": ["account_name"]
        }
    }
}

```

Where `EXTRA` will be a list of account_names that could not be found. Note, that `EXTRA` will only be returned if the error caused was due to `IGNOREMISSING` being false.

## Deleting
As with everything, there comes a time when it's time for them to be removed. In order to remove your acc_name/acc_pass pairs, use the following URL.
```py
BASE + "passwords/delete/"
```

The format of the JSON expected is as follows:
```py

to_post = {
    "LAPM": {
        # As usual, auth goes here
        "AUTH": auth,
        "DELETE": "account_name"
    }
}
```
Where `DELETE` is the name of one account.

A successful response will be in the form of:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": true, 
            "MESSAGE": "A success message"
        },
        "RESPONSE": {"account_name": "account_password"}
    }
}
```
Where `RESPONSE` is a single dictionary mapping the name of the deleted account to it's former password.
An error response will be in the form of:
```json
{
    "LAPM": {
        "SUCCESS_OR_FAILURE": {
            "SUCCESS": false,
            "ERROR": "The error that occurred",
            "MESSAGE": "Any extra information",
            "EXTRA": "The account_name given"
        }
    }
}
```

# End
And that's it. If you encounter any problems that aren't here in the docs, feel free to contact me at zeldaplayergl@gmail.com