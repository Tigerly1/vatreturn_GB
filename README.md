Free, Open Source software for submitting VAT returns to HMRC under their MTD (Making Tax Digital) scheme. Needs some work to support any kind of VAT return.

# About

This software logs you into the HMRC VAT system, and then submits a spreadsheet (which must be in a particular format) as your VAT return. This is modified version that supports submission of VAT returns of any kind as it just reads data from the csv for 7 of the 9 boxes, uses that data to compute the remaining two boxes on submission form and then submits the data as VAT return to HMRC.

You could deploy this to Heroku yourself for free (use the button below), or you can use my deployment at https://standard-vat-return.herokuapp.com - it doesn't store any data, so it's safe to use it from a security point of view. You'll either have to read the code or trust me that the calculations it submits are correct, though.

Read more about the app at https://standard-vat-return.herokuapp.com


# Deploy

Nothing is stored in the app - the data is fetched from a CSV at a URL you define (e.g. from a Google Sheet) and then sent to HMRC. Therefore, you can take advantage of Heroku's free deploy tier to do this instantly.  You'll need to register an application with HMRC (see below) and fill out the `client_id`, `client_secret`, and a URL to the CSV:

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)


# Develop


* [Register application](https://developer.service.hmrc.gov.uk/developer/applications/)
<img src="https://github.com/pubmania/vatreturn/blob/master/img/App%20Registration%20Step%201.png" width="70%" height="70%">

* Note the `client_id` and `client_secret` - You will need to provide these while deploying on Heroku.
  <img src="https://github.com/pubmania/vatreturn/blob/master/img/ClientID%20and%20Client%20Secret.png" width="70%" height="70%">

* Set up the heroku URL (or wherever you have hosted the tool) as a Redirect URL within the application registration section of the HMRC interface 
<img src="https://github.com/pubmania/vatreturn/blob/master/img/Add%20Redirect%20URL.png" width="80%" height="80%">

* [Create test user](https://developer.service.hmrc.gov.uk/api-test-user) and note the login number, password, and vat number
<img src="https://github.com/pubmania/vatreturn/blob/master/img/Test%20User%20Creation.png" width="50%" height="50%">


* Set environment variables on Heroku:

    OAUTHLIB_INSECURE_TRANSPORT=1 FLASK_DEBUG=1 FLASK_APP=vatreturn.py flask run redirect_url = http://localhost:5000/
<img src="https://github.com/pubmania/vatreturn/blob/master/img/Heroku%20Configuration.png">


# Google Sheets format

The CSV you've prepared must match this format. The column headers matter and must be named exactly as shown below:

|VAT period	|VAT Due Sales	|VAT Due Acquisitions	|VAT Reclaimed Curr Period	|Total Value Sales Ex VAT	|Total Value Purchases Ex VAT	|Total Value Goods Supplied Ex VAT	|Total Acquisitions Ex VAT |
| :----: |	:----: |	:----: |	:----: |	:----: |	:----: |	:----: | :----: |
| 2019-11-30 |	2000 |	0 |	10 |	10000 |	162 |	0 | 0 |
| 2019-08-31	| 4000 | 0 |	3000 |	20000 |	24000 |	0 |	0 |

For me this is a table generated using a mysql query on the google sheet scripts editor getting data from mariadb database,
and I share it using the Google Sheets "publish as CSV" functionality.
