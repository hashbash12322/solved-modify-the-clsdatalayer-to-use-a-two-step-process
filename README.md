Download Link: https://assignmentchef.com/product/solved-modify-the-clsdatalayer-to-use-a-two-step-process
<br>
STEP 1: Modify the clsDataLayer to Use a Two-Step Process

1. Open Microsoft Visual Studio.NET.

2. Click the ASP.NET project called <strong>PayrollSystem</strong> to open it.

3. Open the <strong>clsDataLayer</strong> class.

4. Modify the <strong>SavePersonnel</strong>() function so that instead of just doing a single<strong> SQL INSERT</strong> operation with all of the personnel data, it does an <strong>INSERT</strong> with only the FirstName and LastName, followed by an <strong>UPDATE</strong> to save the PayRate, StartDate, and EndDate into the new record. (This two-step approach is not really necessary here because we are dealing with only one table, tblPersonnel, but we are doing it to simulate a case with more complex processing requirements, in which we would need to insert or update data in more than one table or maybe even more than one database.)<strong> Find the following existing code in the SavePersonnel() function:</strong>

<pre> // Add your comments here        strSQL = "Insert into tblPersonnel " +        "(FirstName, LastName, PayRate, StartDate, EndDate) values ('" +        FirstName + "', '" + LastName + "', " + PayRate + ", '" + StartDate +        "', '" + EndDate + "')";        // Add your comments here        command.CommandType = CommandType.Text;        command.CommandText = strSQL;        // Add your comments here        command.ExecuteNonQuery();</pre>

Modify it so that it reads as follows:

<pre>// Add your comments here        strSQL = "Insert into tblPersonnel " +        "(FirstName, LastName) values ('" +        FirstName + "', '" + LastName + "')";        // Add your comments here        command.CommandType = CommandType.Text;        command.CommandText = strSQL;        // Add your comments here        command.ExecuteNonQuery();        // Add your comments here        strSQL = "Update tblPersonnel " +        "Set PayRate=" + PayRate + ", " +        "StartDate='" + StartDate + "', " +        "EndDate='" + EndDate + "' " +        "Where ID=(Select Max(ID) From tblPersonnel)";        // Add your comments here        command.CommandType = CommandType.Text;        command.CommandText = strSQL;        // Add your comments here        command.ExecuteNonQuery();</pre>

5. Set <strong>frmMain</strong> as the startup form and run the PayrollSystem Web application to test the changes. When valid data values are entered for a new employee, things should work exactly as they did previously. To test it, enter valid data for a new employee in frmPersonnel and click Submit. The frmPersonnelVerified form should be displayed with the entered data values and a message that the record was saved successfully. Click the View Personnel button and check that the new personnel record was indeed saved to the database and that all entered data values, including the PayRate, StartDate, and EndDate, were stored correctly. Close the browser window.

Now run the PayrollSystem Web application again, but this time, enter some invalid data (a nonnumeric value) in the PayRate field to cause an error, like this:

6. Now, when you click Submit, the frmPersonnelVerified form should display a message indicating that the record was <strong>not</strong> saved:

However, when you click on the View Personnel button to display the personnel records, you should see that an incomplete personnel record was in fact created, with missing values for the PayRate, StartDate, and EndDate fields.

This occurred because the Insert statement succeeded but the following Update statement did not. We do not want to allow this to happen because we end up with incomplete or incorrect data in the database. If the Update statement fails, we want the Insert statement to be rolled back, or undone, so that we end up with no record at all. We will fix this by adding transaction code in the next step.

Listen

STEP 2: Add Transaction Code

7. In the <strong>clsDataLayer.cls</strong> class file, add code to the SavePersonnel() function to create a transaction object. Begin the transaction, commit the transaction if all database operations are successful, and roll back the transaction if any database operation fails. The following listing shows the complete SavePersonnel() function; the lines you will need to add are marked with ** NEW ** in the preceding comment and are shown in <strong>bold</strong> and underlined.

<pre>// This function saves the personnel data    public static bool SavePersonnel(string Database, string FirstName, string LastName,                                     string PayRate, string StartDate, string EndDate)    {        bool recordSaved;</pre>

<pre><strong>// ** NEW ** Add your comments here</strong>         <strong>OleDbTransaction myTransaction = null;</strong>        try        {            // Add your comments here            OleDbConnection conn = new OleDbConnection("PROVIDER=Microsoft.ACE.OLEDB.12.0;" +                                                       "Data Source=" + Database);            conn.Open();            OleDbCommand command = conn.CreateCommand();            string strSQL;           <strong> // ** NEW ** Add your comments here</strong><strong> myTransaction = conn.BeginTransaction();</strong><strong>  command.Transaction = myTransaction;</strong></pre>

<pre>            // Add your comments here            strSQL = "Insert into tblPersonnel " +                     "(FirstName, LastName) values ('" +                     FirstName + "', '" + LastName + "')";            // Add your comments here            command.CommandType = CommandType.Text;            command.CommandText = strSQL;            // Add your comments here            command.ExecuteNonQuery();            // Add your comments here            strSQL = "Update tblPersonnel " +                     "Set PayRate=" + PayRate + ", " +                     "StartDate='" + StartDate + "', " +                     "EndDate='" + EndDate + "' " +                     "Where ID=(Select Max(ID) From tblPersonnel)";            // Add your comments here            command.CommandType = CommandType.Text;            command.CommandText = strSQL;            // Add your comments here            command.ExecuteNonQuery();           <strong> // ** NEW ** Add your comments here</strong><strong> myTransaction.Commit();</strong>            // Add your comments here            conn.Close();        recordSaved = true;        }        catch (Exception ex)        {           <strong> // ** NEW ** Add your comments here            myTransaction.Rollback();</strong>                       recordSaved = false;        }        return recordSaved;    }</pre>

8. Run your Web application. First, enter <strong>valid data</strong> in all fields of <strong>frmPersonnel</strong>. When you press the Submit button in frmPersonnel, a record should be saved in the tblPersonnel table containing the FirstName, LastName, PayRate, StartDate, and EndDate. With valid data entered in all items, the <strong>successfully saved</strong> message should appear, indicating that the transaction was committed.

Click the View Personnel button and verify that the new record was in fact added to the database table correctly.

9. Now, close the browser, run the Web application again, and this time, test that the transaction will roll back after entering incorrect information. On the frmPersonnel form, enter <strong>invalid</strong> data for PayRate and click Submit. The <strong>not saved</strong> message should appear, which indicates that the transaction was rolled back.

Click the View Personnel button and verify that this time, as desired, an incomplete record was <strong>not</strong> added to the database table.

10.  You have seen how we used the<strong> try/catch</strong> block to catch an unexpected error. You may have noticed that if you enter bad data for the dates, an exception is thrown. Go back to the validation code that you added in the frmPersonnel code and add a try/catch with logic to prevent an invalid date from causing a server error.

11.  In the Week 3 Lab, you learned how to <strong>validate</strong> code once the page was posted back to the server. There is some validation that must be done on the server because it requires server resources such as the database. Some validation can also be done on the client. If you can do validation on the client, it saves a round trip to the server, which will improve performance. In this approach, we will check values before the page is submitted to the server for processing. Normally, there is a combination of server and client validation used in a Web application. ASP.Net includes validation controls which will use JavaScript on the client to perform validation. <strong>You will find these controls in the Validation group in the toolbox.</strong>

12.  Add validation controls to the <strong>frmPersonnel</strong> form as follows: For the first,  last name, and pay rate, make sure each field has data in it. Use the <strong>RequiredFieldValidator</strong> for this task. Add the control to the right of the text box that you are validating. The location of the validator control is where the error message (if there is one) will appear for the control to which you link the validator. You will be adding one validator control for each text box that you want to validate. Remember to set the <strong>ControlToValidate</strong> and <strong>ErrorMessage</strong> properties on the validator control. Making this change eliminates the need for the server-side check you were doing previously. Use a<strong> regular expression validator</strong> to check that the start and end date are in the correct format.

In order to keep the validation controls from causing wrapping, you may want to increase the Panel width.

A regular expression for mm/dd/yyyy is this:

<strong>^(0[1-9]|1[012])[- /.](0[1-9]|[12][0-9]|3[01])[- /.](19|20)dd$</strong>

13.  Remove the <strong>View Personnel</strong> and <strong>Cancel</strong> buttons from the frmPersonnel form, because they will cause a Postback and invoke the client-side editing that you just added. The user is able to get to the View Personnel from the main form and from the personnel verification screen, so there is no need for these buttons now.

14.  Because you have entered data in this lab that is invalid and those partial records are in the database, you will need to add the ability to<strong> remove or update data</strong>. Open up <strong>frmMain</strong> and add a new main form option called <strong>Edit Employees</strong>. Add the link and image for it. This option will take the user to a new form called frmEditPersonnel.

15.  Add the new form <strong>frmEditPersonnel</strong>. On frmEditPersonnel, add the ACIT logo at the top of the form. Add a label that says <strong>Edit Employees.</strong> Add a <strong>GridView</strong> control with an ID of <strong>grdEditPersonnel</strong>.

16.  You will now add a <strong>SQLDataSource</strong> to the page. You will be using a databound grid for this form unlike the previous grids, in which you added as unbound (in the designer).

17.  Add a new <strong>SQLDataSource</strong> control to the frmEditPersonnel in the Design View. This is not a visible control; that is, it will only appear in Design View, but the user will never see it. Note: If you change the folder name or location of your database, you will need to reconfigure the data source (right-click on the data source control and select the Configure Data Source option).

18.  There is a small <strong>&gt;</strong> indicator in the Design View of the SQL Data Source control that you added. If the configuration menu is collapsed (press it to open the menu), or there is a &lt; with the menu displayed, from the data source menu, select <strong>Configure Data Source.</strong>

19.  Press the <strong>New Connection</strong> button and browse for the database.

20.  Press the <strong>Next</strong> button.

21.  When asked if you want to save the connection in the application configuration file, check the <strong>Yes</strong> check box and press Next.

22.  Select the <strong>tblPersonnel</strong> table.

23.  Select all columns (you can use the * for this).

24.  Press the <strong>Advanced</strong> button and check the Generate Insert, Update, and Delete option and press the OK button.

25.  Press the <strong>Next</strong> button.

26.  Press the <strong>Test Query</strong> button and make sure that you see all records in the database like the image below. If it does not, repeat the above steps to make sure that you did everything properly (and selected the correct database – if you are not sure, open the database in Windows Explorer to be sure that it is the one with data in tblPersonnel). Press the Finish button.

27.  Click on the grid that you added in the Design View and expand the <strong>Properties</strong> menu (the little<strong> &gt;</strong> in the upper right of the control). Choose the data source you just added. On the GridView tasks menu, select <strong>Edit</strong> columns. Add an <strong>Edit, Update, and Cancel Command field</strong>.<strong> Add a Delete Command field.</strong> Press OK. You can now test the grid, which is a fully functioning Update and Delete grid. Try it out!

Listen

STEP 3: Test and Submit

28. Once you have verified that everything works as it is supposed to work, save your project, zip up all files, and submit it to the Dropbox.

<strong>NOTE</strong>: Make sure you include comments in the code provided where specified (where the ” // Your comments here” is mentioned) and for any code you write, or else a 5-point deduction per item (form, class, function) will be made.