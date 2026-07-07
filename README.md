Flow Functionality:

Flow 1 – Salesforce Lead Synchronization:

This flow receives lead information through an HTTP endpoint and synchronizes the data between Salesforce CRM and a PostgreSQL database while maintaining transactional consistency.

Processing Steps:

Receives the lead details through the POST /lead endpoint.
Stores the incoming request payload for reuse during processing.
Validates duplicate requests using the Idempotent Message Validator with Object Store to prevent processing the same lead multiple times.
Transforms the incoming JSON payload into the Salesforce Lead object using DataWeave.
Starts a transaction to ensure consistent processing.
Creates a new Lead record in Salesforce.
Stores the generated Salesforce Lead ID for rollback purposes if required.
Inserts the same lead information into the PostgreSQL database.
Commits the transaction when both Salesforce and database operations complete successfully.
Returns a success response confirming that the Lead has been synchronized across both systems.

Flow 2 – Local Lead Storage :

This flow stores lead information directly into the PostgreSQL database without interacting with Salesforce.

Processing Steps:

Receives the lead details through the POST /localLead endpoint.
Stores the incoming request for later use.
Validates duplicate requests using the Idempotent Message Validator.
Starts a database transaction.
Inserts the lead record into PostgreSQL.
Retrieves the inserted record to verify successful insertion.
Commits the transaction and returns the inserted lead details in the response.

Error Handling:

The application implements both transaction-level and flow-level error handling to ensure data consistency and provide meaningful responses to API consumers.

Database Errors:

Handled When:

Database connection failure
SQL execution errors
Constraint violations
Insert operation failure

Behavior:

Logs the database error.
If a Salesforce Lead has already been created, the application deletes the Lead from Salesforce as a compensating rollback operation.
Raises a custom APP:TRANSACTION_ROLLBACK error.
Returns a standardized error response indicating that the transaction has been rolled back.

Salesforce Errors:

Handled When:

Salesforce authentication fails.
Salesforce service is unavailable.
Lead creation fails.
Salesforce connector throws an exception.

Behavior:

Logs the Salesforce error.
Stops further processing.
Database insertion is not executed.
Raises a custom APP:SALESFORCE_ERROR.
Returns a standardized Salesforce error response.

Transaction Rollback:

To maintain synchronization between Salesforce and PostgreSQL, the application performs a compensating rollback whenever a database failure occurs after the Salesforce Lead has been created.

Rollback Process:

Salesforce Lead is successfully created.
Database insertion fails.
Stored Salesforce Lead ID is retrieved.
The created Salesforce Lead is deleted.
A transaction rollback error is raised.
The client receives a rollback response.
