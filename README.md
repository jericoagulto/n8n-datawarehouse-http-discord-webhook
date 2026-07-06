This workflow fetches warehouse order data, routes orders by status, stores processing orders in a data table, calculates statistics for each status category, and sends summaries to Discord every Monday at 9am.

Detailed Step-by-Step Breakdown:

Triggers (2 ways to start):

Manual Trigger - Click "Execute workflow" button to test the workflow immediately
Schedule Trigger - Automatically runs every Monday at 9:00 AM
Data Retrieval:
3. GetDataFromWarehouse - Makes an HTTP GET request to the n8n warehouse API endpoint with authentication header x-assessment-id. Returns order records containing orderID, customerID, employeeName, orderPrice, and orderStatus fields.

Conditional Routing:
4. CheckOrderStatus - Evaluates each order's status field. Routes to TRUE branch if status equals "booked", FALSE branch if status equals "processing" or anything else.

TRUE Branch (Booked Orders):
5. CalculateBookedOrders - Aggregates all booked orders using JavaScript code. Counts total number of booked orders and sums their orderPrice values into totalBooked and bookedSum.
6. TriggerMondays9am_2 - Sends the booked orders summary (total count and sum) to Discord via webhook.

FALSE Branch (Processing Orders):
7. SetProcessingData - Extracts only orderID and employeeName fields from processing orders, discarding other fields.
8. Upsert row(s) - Inserts or updates rows in the "processingOrders" data table. Matches existing rows by orderID and updates employeeName if found, otherwise creates new row.
9. CalculateBookedProcessingOrders - Aggregates all processing orders using JavaScript code. Counts total number and sums orderPrice values.
10. TriggerMondays9am_1a - Sends the processing orders summary to Discord via webhook.

Data Flow:

Each order from GetDataFromWarehouse is evaluated individually by CheckOrderStatus
Booked orders flow through steps 5→6
Processing orders flow through steps 7→8→9→10
Both branches execute in parallel and send independent Discord messages
The data table accumulates processing orders over time for tracking purposes
