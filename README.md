@RestResource(urlMapping='/programUpsertResource/*')
global class ProgramUpsertResource {
    
    @HttpPost
    global static void upsertAccount() {
        // Create lists to hold program data for processing
        List<ProgramWrapper> programList = new List<ProgramWrapper>();
        List<Program__c> programListadd = new List<Program__c>();
        
        RestRequest req = RestContext.request;
        RestResponse res = RestContext.response;
        
        if (req != null) {
            String body = req.requestBody.toString().trim();
            
            if (body != null && body != '') 
			{
                try {
                    // Deserialize the JSON request body into a list of ProgramWrapper objects
                    programList = (List<ProgramWrapper>) JSON.deserialize(body, List<ProgramWrapper>.class);
                    System.debug('serviceList>>' + programList);
                    
                    // Fetch the RecordType IDs outside the loop
                    Map<String, Id> recordTypeIds = new Map<String, Id>();
                    List<RecordType> recordTypes = [SELECT Id, Name FROM RecordType WHERE SobjectType = 'Program__c' AND Name IN ('Grant Program', 'Generic Program', 'Housing Program')];
                    
                    // Store the RecordType names and IDs in a map for easy access
                    for (RecordType rt : recordTypes) 
					{
                        recordTypeIds.put(rt.Name, rt.Id);
                    }
                    
                    for (ProgramWrapper we : programList)
                    {
                        // Create a new Program__c object for each ProgramWrapper
                        Program__c s = new Program__c();
                        
                        // Set the fields based on the values in ProgramWrapper
                        
                        // Name field
                        if (we.Name != null && we.Name != '') {
                            s.Name = we.Name;
                        }
                        
                        // Program_ID__c field
                        if (we.ProgramID != null && we.ProgramID != '') {
                            s.Program_ID__c = Decimal.valueOf(we.ProgramID);
                        }
                        
                        // Program_Code__c field
                        if (we.ProgramCode != null && we.ProgramCode != '') {
                            s.Program_Code__c = we.ProgramCode;
                        }
                        
                        // ExternalFiled__c field
                        if (we.ProgramID != null && we.ProgramID != '') {
                            s.ExternalFiled__c = we.ProgramID + ' - ' + we.ProgramCode;
                        }
                        
                        // Program_Start_Date__c field
                        if (we.StartDate != null && we.StartDate != '') {
                            s.Program_Start_Date__c = Date.valueOf(we.StartDate);
                        } else {
                            s.Program_Start_Date__c = null;
                        }
                        
                        // Program_End_Date__c field
                        if (we.EndDate != null && we.EndDate != '') {
                            s.Program_End_Date__c = Date.valueOf(we.EndDate);
                        } else {
                            s.Program_End_Date__c = null;
                        }
                        
                        // Billed_From__c field
                        if (we.BILLEDFROM != null && we.BILLEDFROM != '') {
                            s.Billed_From__c = we.BILLEDFROM;
                        } else {
                            s.Billed_From__c = '';
                        }
                        
                        // Channel_Access_Point_s__c field
                        if (we.ACCESSPOINTS != null && we.ACCESSPOINTS != '') {
                            s.Channel_Access_Point_s__c = we.ACCESSPOINTS;
                        } else {
                            s.Channel_Access_Point_s__c = '';
                        }
                        
                        // Client_Session_Fee__c field
                        if (we.SessionFee != null && we.SessionFee != '') {
                            s.Client_Session_Fee__c = Decimal.valueOf(we.SessionFee);
                        } else {
                            s.Client_Session_Fee__c = null;
                        }
                        
                        // Client_Enrollment_Fee__c field
                        if (we.EnrollmentFee != null && we.EnrollmentFee != '') {
                            s.Client_Enrollment_Fee__c = Decimal.valueOf(we.EnrollmentFee);
                        } else {
                            s.Client_Enrollment_Fee__c = null;
                        }
                        
                        // Client_Cancellation_Fee__c field
                        if (we.CancellationFee != null && we.CancellationFee != '') {
                            s.Client_Cancellation_Fee__c = Decimal.valueOf(we.CancellationFee);
                        } else {
                            s.Client_Cancellation_Fee__c = null;
                        }
                        
                        // Client_Recurring_Fee__c field
                        if (we.RecurringFee != null && we.RecurringFee != '') {
                            s.Client_Recurring_Fee__c = Decimal.valueOf(we.RecurringFee);
                        } else {
                            s.Client_Recurring_Fee__c = null;
                        }
                        
                        // RecordTypeId field
                        if (we.RecordType != null && we.RecordType != '') {
                            if (recordTypeIds.containsKey(we.RecordType)) {
                                s.RecordTypeId = recordTypeIds.get(we.RecordType);
                            }
                        }
                        
                        // Service_Associated__c field
                        if (we.ServiceAssociated != null && we.ServiceAssociated != '') {
                            s.Service_Associated__c = we.ServiceAssociated;
                        } else {
                            s.Service_Associated__c = '';
                        }
                        
                        // Program_Status__c field
                        if (we.Status != null && we.Status != '') {
                            s.Program_Status__c = we.Status;
                        } else {
                            s.Program_Status__c = '';
                        }
                        
                        // Program_Description__c field
                        if (we.Description != null && we.Description != '') {
                            s.Program_Description__c = we.Description;
                        } else {
                            s.Program_Description__c = '';
                        }
                        
                        // Add the Program__c object to the list for upserting
                        programListadd.add(s);
                        System.debug('SizeofList' + programListadd.size());
                        System.debug('programListadd>>>>' + programListadd);
                    }
                    
                    if (!programListadd.isEmpty())
					{
                        // Perform the upsert operation on the Program__c records using ExternalFiled__c field as the external ID
                        Schema.SObjectField externalIdField = Program__c.Fields.ExternalFiled__c;
                        Database.UpsertResult[] upsertResults = Database.upsert(programListadd, externalIdField, false);
                        System.debug('upsertResults>>' + upsertResults);
                        
                        // Set the response status code and headers
                        res.statusCode = 200;
                        res.addHeader('Content-Type', 'application/json');
                        res.responseBody = Blob.valueOf(JSON.serialize(programListadd));
                        System.debug('res.responseBody>>' + res.responseBody);
                    } else
					{
                        // No records to upsert, set appropriate response
                        res.statusCode = 500;
                        res.addHeader('Content-Type', 'application/json');
                        res.responseBody = Blob.valueOf('No records to upsert');
                        System.debug('res.responseBody>>' + res.responseBody);
                    }
                } catch (Exception e)
				{
                    // Handle any exceptions that occur during processing
                    res.statusCode = 500;
                    res.addHeader('Content-Type', 'application/json');
                    res.responseBody = Blob.valueOf(e.getMessage());
                    System.debug('res.responseBody>>' + res.responseBody);
                }
            } else {
                // Invalid request body
                res.statusCode = 500;
                res.addHeader('Content-Type', 'application/json');
                res.responseBody = Blob.valueOf('Invalid Request');
                System.debug('res.responseBody>>' + res.responseBody);
            } 
        } else {
            // RestRequest is null, return error response
            res.statusCode = 400;
            res.addHeader('Content-Type', 'application/json');
            res.responseBody = Blob.valueOf('Invalid request: RestRequest is null');
        }
    }
    
    // Wrapper class for the JSON request body
    public class ProgramWrapper {
        public String Name { get; set; }
        public String Description { get; set; }
        public String Status { get; set; }
        public String ProgramID { get; set; }
        public String ProgramCode { get; set; }
        public String StartDate { get; set; }
        public String EndDate { get; set; }
        public String BILLEDFROM { get; set; }  
        public String ACCESSPOINTS { get; set; }
        public String SessionFee { get; set; }
        public String EnrollmentFee { get; set; }
        public String CancellationFee { get; set; }
        public String RecurringFee { get; set; }
        public String RecordType { get; set; }
        public String ServiceAssociated { get; set; }
    }
}

