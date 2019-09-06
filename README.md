Free, Open Source software for submitting VAT returns to HMRC under their MTD (Making Tax Digital) scheme. ~~Needs some work to support any kind of VAT return.~~

# About

This software logs you into the HMRC VAT system, and then submits a spreadsheet (which must be in a particular format) as your VAT return. This is modified version that supports submission of VAT returns of any kind as it just reads data from the csv for 7 of the 9 boxes, uses that data to compute the remaining two boxes on submission form and then submits the data as VAT return to HMRC.

You could deploy this to Heroku yourself for free (use the button below), ~~or you can use my deployment at https://standard-vat-return.herokuapp.com - it doesn't store any data, so it's safe to use it from a security point of view. You'll either have to read the code or trust me that the calculations it submits are correct, though.~~

My deployment is still on HMRC sandbox environment so you will not be able to use it to submit your VAT returns and even if you do deploy your own version you will need to get it approved by HMRC before you can use it. I have submitted approval request for mine and will update here as soon as I have the approval. 

Read more about the app at https://standard-vat-return.herokuapp.com

<img src="https://github.com/pubmania/vatreturn/blob/master/img/VAT%20Return%20MTD%20Bridge.gif">


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

I am also providing some guidance on how to connect google sheets to a mysql database but ofcourse this can be a more involved process depending on set-up of your mysql server:

1. In a new googlesheet, open the script editor Tools -> Script Editor, as shown below:
<img src="https://github.com/pubmania/vatreturn/blob/master/img/GoogleSheets%20Step1.png">
2. Now give it a name, a function name and then create connection to your database as shown below:
<img src="https://github.com/pubmania/vatreturn/blob/master/img/Google%20Sheet%20database%20Connection.png">
3. A sample code is as shown below (YOU WILL NEED TO MAKE CHANGES TO CONN VARIABLE, rs2 VARIABLE AND TO doc VARIABLE FOR THIS TO WORK):

```javascript
function your_function_name() { 
  
  var conn = Jdbc.getConnection('jdbc:mysql://your.mysqlserver.com:3306/your_database_name', 'database_read_only_username', 'database_read_only_password'); // Change it as per your database credentials

  var stmt = conn.createStatement();
  var start = new Date(); // Get script starting time

//Create a string containing your query  
//change query as per your database structure
  var rs2 = 
      'select date_format(a.expense_date,\'%b-%Y\') as `Month-Year`, b.name as `Category`,  sum(a.amount) as `Totals` from expenses a' +
      ' join expense_categories b on a.expense_category_id = b.id' +
      ' where a.is_deleted <> \'1\' ' +
      ' AND PERIOD_DIFF(DATE_FORMAT(CURDATE(),\'%Y%m\'),DATE_FORMAT(a.expense_date,\'%Y%m\'))<12' +
      ' group by date_format(a.expense_date,\'%b-%Y\'), b.name'
  
  var rs = stmt.executeQuery(rs2);


   
  var doc1 = SpreadsheetApp.getActiveSpreadsheet(); // Returns the currently active spreadsheet
  var doc = doc1.getSheetByName('SheetName'); //Enter the sheetname you have given on google sheet
  
  //delete current content on spreadsheet
  var start_del, end_del;

  start_del = 1;
  end_del = doc.getLastRow() - 1;//Number of last row with content
  if (end_del = -1) {end_del = 1;}
  //blank rows after last row with content will not be deleted
  
  doc.deleteRows(start_del, end_del);
  //end delete
  
  var cell = doc.getRange('a1');
  var row = 0;
  var getCount = rs.getMetaData().getColumnCount(); // Mysql table column name count.
  
  for (var i = 0; i < getCount; i++){  
     cell.offset(row, i).setValue(rs.getMetaData().getColumnName(i+1)); // Mysql table column name will be fetched and added in spreadsheet.
  }  
  
  var row = 1; 
  while (rs.next()) {
    for (var col = 0; col < rs.getMetaData().getColumnCount(); col++) { 
      cell.offset(row, col).setValue(rs.getString(col + 1)); // Mysql table column data will be fetch and added in spreadsheet.
    }
    row++;
  } 
 
  
  rs.close();
  stmt.close();
  conn.close();
  var end = new Date(); // Get script ending time
  Logger.log('Time elapsed: ' + (end.getTime() - start.getTime())); // To generate script log. To view log click on View -> Logs.
}
```

