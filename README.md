# Nem-id integration

A PHP Laravel library for using the Danish NemID for authenticating a user.

I'm sure it can be used easily without laravel also. Feel free to contribute to improvements

![image](https://cloud.githubusercontent.com/assets/1279756/20196240/18e4a2b8-a79a-11e6-832b-36933da588e3.png)
 
### The library supports: 
  - Preparing the parameters for the applet
  - Validate the returned signature and the certificate chain
  - Extract Name and PID
  - Matching PID to CPR SOAP webservice
    
This is a rewrite of an original library for an older version of the applet in java

Original library can be found: https://code.google.com/p/nemid-php/ 
    
## 🔧 Laravel Setup

Setup service provider in `config/app.php`

```php
Nodes\Nemid\ServiceProvider::class
```

Publish config files

```bash
php artisan vendor:publish --provider="Nodes\NemId\ServiceProvider"
```

If you want to overwrite any existing config files use the `--force` parameter

```bash
php artisan vendor:publish --provider="Nodes\NemId\ServiceProvider" --force
```

## Certificates

####Make sure you have bcmath installed
```
sudo apt-get install php7.0-bcmath
```

You got your p12 certificate now generate pem files, use following commands: 

#####publicCertificate:
`openssl pkcs12 -in path.p12 -out certificate.pem -clcerts -nokeys`

#####privateKey & privateKeyPassword
`openssl pkcs12 -in path.p12 -clcerts -out privateKey.pem`

#####certifateAndPrivateKey & password (For PID/CPR match)
`openssl pkcs12 -in path.p12 -out certicateAndPrivateKey.pem -nocerts -nodes`     

Now you have all the certificates needed 

##### Copy the config file to htdocs and fill settings
Look in the config file for more help

#Login integration
In the inspiration folder an example of how you can setup the login flow can be found.

First prepare parameters to inject into the iframe. By creating a Login object.

`$login = new Login(config('nodes.nemid'));`

Setup a html document with the iframe url, js with param data and a form for callbacks

`$login->getIFrameUrl();`

`$login->getParams();`

The iframe will now submit the response to the form 

The submitted data is base64 encoded, besides that all errors comes as string while successfully logins are xml documents

`$response = base64_decode(\Input::get('response'));`

`CertificationCheck::isXml($response)`

Now validate the certificates and extract name and PID from it by initialize a CertificationCheck object

`$userCertificate = new CertificationCheck(config('nodes.nemid'));`

`$certificate = $userCertificate->checkAndReturnCertificate($response);`

`$certificate->getSubject()->getName();`

`$certificate->getSubject()->getPid();`

#PID/CPR match integration
Initialize a PidCprMatch object and call the function with pid and cpr params.

`$pidCprMatch = new PidCprMatch(config('nodes.nemid'));`

`$response = $pidCprMatch->pidCprRequest($pid, $cpr);`

A response object will be returned. The object has functions to to check match and possible errors

`$response->didMatch();`

### Misc

 - The name `Pseudonym` or `Pseudonym Pseudonym` will be used for version 1 of nemid users, which have not set their name afterwards

Enjoy
 


